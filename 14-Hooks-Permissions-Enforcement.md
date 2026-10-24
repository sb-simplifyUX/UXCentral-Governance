---
title: Hooks, permissions and enforcement
description: The four plugin hooks, when to refuse and when to ask, the full user-level and project-level permission lists, the enforcement strength scale, the rule-to-enforcement matrix and the honest list of known limits.
doc_type: governance
project: central
process_step: n/a
brand: all
status: approved
owner: Central administrator
created: 2026-10-23
updated: 2026-10-24
governance_version: 1.4.0
contains_pii: false
---

# Hooks, permissions and enforcement

## Strength scale

| Strength | Meaning |
|---|---|
| **Hard** | A permission rule, hook or script refusal stops or asks, whatever the AI decides |
| **Checked** | A script or reviewer agent verifies it after the fact |
| **Guided** | An instruction the AI follows (core rules, skill), or a mechanism that only prints or reminds; not mechanically enforced |

Central keeps `ENFORCEMENT.md`: every rule, what enforces it, and its strength. A rule with no enforcement entry is a defect.

## Hooks

`marketplace/plugins/<PLUGIN>/hooks/hooks.json`:

```json
{
  "description": "UX governance: block personal information in outbound tool calls; refuse shell commands that write or bulk-delete files, run inline code, reach the network or discard git work; ask before shell commands that change central governance; report governance version and project status at session start",
  "hooks": {
    "PreToolUse": [
      { "matcher": "^(WebFetch|WebSearch|Artifact|Agent|Task|mcp__.*)$",
        "hooks": [{ "type": "command", "command": "node \"${CLAUDE_PLUGIN_ROOT}/hooks/pii-guard.mjs\" || exit 2", "timeout": 15 }] },
      { "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "node \"${CLAUDE_PLUGIN_ROOT}/hooks/shell-guard.mjs\" || exit 2", "timeout": 15 },
                  { "type": "command", "command": "node \"${CLAUDE_PLUGIN_ROOT}/hooks/central-guard.mjs\" || exit 2", "timeout": 15 }] }
    ],
    "SessionStart": [
      { "matcher": "startup|resume|clear|compact",
        "hooks": [{ "type": "command", "command": "node \"${CLAUDE_PLUGIN_ROOT}/hooks/version-check.mjs\" || true", "timeout": 15 }] }
    ]
  }
}
```

| Hook | Event | Behaviour | Failure mode |
|---|---|---|---|
| **pii-guard** | Before web fetch/search, published pages, subagent prompts (`Agent`, and `Task`, the older name of the subagent tool) and every MCP call | Scans every text value of the tool input, and its URL-decoded form, for emails (allow-listing example domains and the organisation's own), phone numbers and region-specific identifiers; blocks with exit code 2 and a masked summary telling the AI to de-identify | **Fails closed**: if patterns cannot load, the input cannot be read, `node` is missing or the hook crashes, the call is blocked (the command falls back to `exit 2`) |
| **shell-guard** | Before every shell command | Denies, with a reason that names what to use instead: shell file writes (`>` or `>>` into a file, `tee`, a heredoc into a file, `sed -i`, `perl -i`, `awk -i inplace`, `dd of=`, `truncate`, `cp`, `mv`); bulk deletes (recursive, wildcard or piped `rm`, `find` with `-delete` or `-exec`); inline code (`python3 -c`, `node -e`, an interpreter or shell reading code from a heredoc, `sh -c`, `eval`); outward network commands (`curl`, `wget`, `ssh`, `scp`, `rsync`, `nc` and similar); download-and-run (`npx`, `pnpm dlx`, `npm exec`); publishing; and git that discards work or rewrites what was pushed (`reset --hard`, `clean`, `checkout --`, `restore` of working files, force push, `remote add` or `set-url`). It looks through quotes, full paths such as `/usr/bin/curl`, `sudo`, `env`, `xargs`, shell keywords and command substitutions, and skips heredoc bodies, which are data. A command it has no rule for gets no output, so the permission rules decide: `rm` of one named file, a plain push and installs still ask | **Fails closed**: unreadable input is denied; a missing `node` or a crash blocks (the command falls back to `exit 2`) |
| **central-guard** | Before every shell command | If the command names central or runs inside it, it returns an `ask` decision unless the command is read-only or ordinary work: `ls`, `pwd`, `cat`, `head`, `tail`, `wc`, `grep` and `git status` with plain arguments; `git diff`, `log`, `show`, `add`, `commit` and `fetch` (a quoted commit message is fine, `--output` still asks); the tool tests (`node --test 'tools/test/*.test.mjs'`); `claude plugin validate`; and central's own read-only tools (`node <path>/tools/check-md.mjs ...` and `node <path>/tools/retrofit-audit.mjs ...`, by relative or absolute path, quoted or not). Every other command that touches central asks, including `git push`, `git pull`, `new-project.mjs` and `retrofit-apply.mjs`: publishing central, taking in changes, creating a project and applying a retrofit ask for confirmation by design | **Fails towards asking**: unreadable input or an error asks; a missing `node` or a crash blocks (the command falls back to `exit 2`) |
| **version-check** | Session start, resume, clear, compact | Inside central: the version and the governance-change reminder. Inside a project: governance version against central, current step, Doing count, decider (or asks for one), retrofit hint if the versions differ. Anywhere governed: design-library check reminder when due. Nothing outside governed folders. Values read from `PRD.md` are printed only when valid. The output goes into Claude's context, not necessarily onto the screen: to check it, ask Claude to quote the `UX governance:` line from its session context | Silent on error (the command falls back to `true`); prints facts only |

The `|| exit 2` at the end of the three guard commands turns a missing `node` or a crash into a block instead of letting the call through. The `|| true` at the end of the version check means it never reports an error at session start.

Why a central guard as well as an `Edit` permission: Edit permission rules are checked for Claude's file tools, for some recognised shell file commands (such as `sed`, `tee`, `cat`) and for `>` and `>>` redirection targets. They are not checked for files a program such as `node`, `pnpm`, `git` or a script writes itself; the central guard covers those, and `cp`, `mv` and `rm`.

Why a shell guard as well as `deny` rules: a Bash rule matches the command as written, so `Bash(curl *)` does not stop `/usr/bin/curl` or `sh -c 'curl …'`, and no rule can say "a redirect into any file". The shell guard reads the whole command for those forms. The `deny` rules stay as the first layer because they hold even when the plugin is not loaded.

Hook sources (generalised and configurable) are in [Appendix C](appendices/C-Hook-and-Tool-Sources.md).

## Refuse, or ask

Every guard either refuses an action or asks a person about it, and the two cost very differently:

| Mechanism | Who sees it | In auto mode |
|---|---|---|
| A `deny` rule, or a hook that returns `deny` | The AI, as the reason the call failed | Blocks without a prompt; the AI switches tool or asks the user in words |
| An `ask` rule, or a hook that returns `ask` | The person, as a permission prompt | Still prompts: an explicit ask is never approved by the auto-mode classifier |

So refuse what must never happen, and ask only at a real approval gate: deleting one named file, a push, an install, a change to central or to user settings, a design-tool write, an Opus subagent call. A long `ask` list turns auto mode back into constant prompting, while a `deny` list interrupts nobody. Both behaviours are stated in the current Claude Code permissions and auto-mode documentation; check them there after an upgrade.

Files change only through the Edit and Write tools. An Edit fails when its anchor does not match exactly once, each call changes one file and shows a diff, and `/rewind` restores what the file tools changed; a file changed only by a shell command is not tracked. In auto mode, Edit and Write inside the working folder run without a prompt, so the safe path is also the uninterrupted one. The refusals matter in `acceptEdits` mode too, which otherwise approves `rm`, `mv`, `cp` and `sed` inside the working folder without asking.

## User-level permissions

### deny (secrets)

```text
Read(//**/.env)   Read(//**/.env.*)   Read(//**/*.pem)   Read(//**/*.pfx)   Read(//**/*.p12)   Read(//**/*.key)
Read(~/.ssh/**)   Read(~/.claude/.credentials.json)   Read(~/.npmrc)   Read(~/.git-credentials)
Read(~/.config/gh/**)   Read(~/.aws/**)   Read(~/.azure/**)   Read(~/.gnupg/**)   Read(~/.netrc)
Read(~/.docker/config.json)   Read(~/.kube/**)   Read(~/.config/gcloud/**)
Bash(cat *.env*)   Bash(cat *.pem*)   Bash(cat *.key*)   Bash(cat *credentials*)
```

### deny (what must never happen)

Each pattern is a `Bash(...)` rule. The shell guard refuses the same actions when they are written another way, so these rules are the first layer, not the only one.

| Group | Patterns |
|---|---|
| Shell file writes | `sed -i*`, `sed * -i*`, `perl -i*`, `perl -pi*`, `tee *`, `cp *`, `mv *` |
| Bulk deletes | `rm -r*`, `rm -R*`, `rm -fr*`, `rm -fR*`, `rm * -r*`, `rm * -R*`, `find * -delete*`, `find * -exec*`, `find * -ok*` |
| Inline code | `python -c *`, `python3 -c *`, `node -e *`, `node --eval *`, `node -p *`, `perl -e *`, `ruby -e *`, `sh -c *`, `bash -c *`, `zsh -c *`, `eval *` |
| Git that discards work or rewrites what was pushed | `git reset --hard`, `git reset --hard *`, `git clean *`, `git checkout -- *`, `git checkout .`, `git push --force*`, `git push -f*`, `git push * --force*`, `git push * -f*`, `git remote add *`, `git remote set-url *` |
| Outward | `curl *`, `wget *`, `scp *`, `sftp *`, `rsync *`, `ssh *`, `nc *`, `ncat *`, `socat *`, `telnet *` |
| Publish and download-and-run | `npm publish*`, `pnpm publish*`, `npx *`, `pnpx *`, `pnpm dlx *`, `npm exec *` |

Web pages are read with WebFetch, which the PII guard checks. A tool that `npx` would have fetched becomes an exact-pinned dependency (an install, which asks) run with `pnpm exec`. A copy, a move or a bulk delete is the user's to do.

### ask (approval gates only)

| Group | Patterns (shell commands are `Bash(...)` rules, bare and with arguments) |
|---|---|
| Delete one named file | `Bash(rm *)`, `Bash(rmdir *)`. Recursive and wildcard forms are refused above |
| Push | `git push`, `git push *`, `git -C * push *`, `git * push *` |
| Install | `pnpm add`, `pnpm install`, `pnpm i`, `pnpm update`, `pnpm up`, `npm install`, `npm i`, `npm ci`, `npm add`, `npm update`, `pip install`, `claude plugin install`, `brew install`, `brew upgrade`, `apt install`, `apt-get install`, `dnf install` |
| Central and user config | `Edit(//{{CENTRAL_POSIX}}/**)` (the installer fills `{{CENTRAL_POSIX}}` with your central path without its leading slash), `Edit(~/.claude/settings.json)`, `Edit(~/.claude/CLAUDE.md)` |
| Design-tool writes | Every write tool of the design MCP server. For Figma, the server comes from the `figma` plugin, so its tools are named `mcp__plugin_figma_figma__<tool>`: `use_figma`, `generate_figma_design`, `send_code_to_figma`, `create_new_file`, `upload_assets`, `add_code_connect_map`, `send_code_connect_mappings`, `create_shader`, `update_shader`, `create_generative_plugin`, `update_generative_plugin`, `generate_diagram`, `generate_image` (the list in [Appendix A §3](appendices/A-File-Templates.md#3-user-settings-user-configsettingsjson)), and any other write tool your live session lists. Confirm every name against the tools listed in the session before writing the rules |
| Model | `Agent(model:opus)` and `Agent(model:<OPUS_MODEL_ID>)`. `Agent(model:opus)` asks only when a spawn passes model opus on the call; it does not match the full model ID or a model set in an agent's frontmatter, hence the second rule. Reviewers that set `model: opus` in their frontmatter run without a prompt |

## Project-level permissions

```json
{ "permissions": { "ask": ["Edit(**/phases/*/research/**)"], "deny": ["Read(**/.env)", "Read(**/.env.*)"] } }
```

## Script refusals

| Script | Refuses |
|---|---|
| `install.mjs` | Missing `node` or `claude` on PATH; current settings that switch the guards off (`disableAllHooks: true` or `defaultMode: "bypassPermissions"`) or are not valid JSON; placeholders left in central's settings file; invalid person values; unsafe characters in the home or central path for permission patterns; damaged governance markers in the user's `CLAUDE.md`; no terminal and no flags. It never replaces the user's `CLAUDE.md`, only the block between its markers |
| `new-project.mjs` | Existing folder; invalid name for the file system; missing or invalid decider values; multi-line summary; an unreadable `VERSION` (checked before anything is written) |
| `check-md.mjs` | A file or folder that does not exist (exit 66); outside `--template`, any unfilled `{{NAME}}` in a field value is reported as a problem |
| `retrofit-audit.mjs` | Output folder inside the project or inside central (case-insensitive, link-aware); non-empty output folder that is not this project's report; invalid brand; auditor corrections to decider values |
| `retrofit-apply.mjs` | Anything that cannot be checked before writing: exempt or non-Markdown files, non-UTF-8, unreadable frontmatter, placeholders, values outside allowed lists, invalid dates, aliased or duplicate paths, read-only or locked files, non-empty backup folder, unapproved or altered body changes and renames, overlapping renames, and more |
| `pnpm` | Versions younger than 14 days (`minimumReleaseAge`). Range prefixes are not refused: `savePrefix: ''` only sets the prefix `pnpm add` writes, so the code-stack reviewer checks for ranges |

## Enforcement matrix (condensed)

| Area | Rule | Enforced by | Strength |
|---|---|---|---|
| Architecture | Central is the single source; projects reference | User CLAUDE.md import; live plugin; PRD reviewer | Checked |
| | Each project records its version | Template; tools; version-check hook (warns, never refuses); `check-md` requires the field | Checked |
| | Central changes need approval | `ask: Edit(central)`; central-guard; governance-change user-only | Hard |
| Roles | Every project has a decider | Tools refuse without one; hook asks; PRD reviewer | Hard + Checked |
| | The decider approves everything | Core rules; skills stop and record | Guided |
| | Setup jobs start only from a person | `disable-model-invocation` | Hard |
| Reviewers | Read-only | Tool lists | Hard |
| | Zero-tool-call review invalid; two loops then escalate | `review` skill | Guided |
| Process | Steps in order; step shown each session | Session-start hook (prints the step only); process-gate skill; PRD reviewer | Guided + Checked |
| | Define cannot be skipped | process-gate; figma-sketch; ui-build; reviewer | Guided + Checked |
| Kanban | Nine sections; one FALSIFIER; doors; Doing ≤ 3 | kanban skill; PRD reviewer | Guided + Checked |
| | Close release is the decider's | `release` is user-only, but the AI could still edit `PRD.md` directly; PRD reviewer checks the safety statement | Guided + Checked |
| Markdown | Frontmatter and one H1 | check-md; PRD reviewer | Checked |
| Design system | Allowed library only; never invent | brand rules; pinned search; brand reviewer | Checked |
| | Never write to the library | `ask` on design-tool writes | Hard |
| | Map re-checked every 14 days | Hook reminder; skill | Guided |
| Stack | Release age ≥ 14 days | pnpm `minimumReleaseAge`; code-stack reviewer | Hard + Checked |
| | Exact pins | pnpm `savePrefix: ''` sets the prefix `pnpm add` writes but refuses nothing; code-stack reviewer | Checked |
| | Ask before install | `ask` permissions | Hard |
| | No download-and-run (`npx`, `pnpm dlx`, `npm exec`) | `deny` rules; shell-guard | Hard (command text; see Known limits) |
| Models | Opus orchestrates; Sonnet implements; reviewers run on Opus | Settings; agent frontmatter; `ask: Agent(model:opus)` and `Agent(model:<OPUS_MODEL_ID>)`, which ask only when a call passes the model | Guided |
| Safety | No PII outbound | pii-guard | Hard (patterns only) |
| | No PII in documents | privacy reviewer; `contains_pii` | Checked |
| | Raw research never committed | `.gitignore` | Hard |
| | Research not overwritten silently | Project `ask: Edit(**/phases/*/research/**)` | Hard (file tools and recognised shell writes; see Known limits) |
| | Retrofit non-destructive | Two-phase apply; backups; tests | Hard |
| | Ask before deleting one file and before push | `ask` permissions | Hard |
| | No bulk deletes; no git that discards work or rewrites what was pushed | `deny` rules; shell-guard | Hard (command text; see Known limits) |
| | No outward shell commands; web reads through WebFetch | `deny` rules; shell-guard; pii-guard on WebFetch | Hard (command text; see Known limits) |
| | Secrets out of repositories | `deny` read; `.gitignore`; reviewer | Hard + Checked |
| | Files change only through Edit and Write | `deny` rules; shell-guard; central-guard in central | Hard (command text; see Known limits) |
| | Read the change back after every edit | Core rules | Guided |

## Known limits (keep this list honest)

- Shell permission rules match the command text the AI writes; they are not a security boundary against a program invoked another way. The shell guard and the central guard read only the command text too. The shell guard catches the forms an AI writes (full paths, `sudo`, `env`, `xargs`, `sh -c`, quotes, substitutions, heredocs), not a script file that writes files itself, a write inside a quoted `sed` or `awk` program, or a shell alias. They stop habits, not a determined program; for an operating-system boundary, use Claude Code's sandbox.
- The shell guard refuses some read-only commands that look like writes, such as `find . -exec grep …` or a `>` written as a comparison. Its reason names the tool to use instead (Grep, Glob, Read), so this costs a retry, not a prompt.
- The central guard asks only when the command names central or runs inside it. A program that writes into central without the command naming it (for example a Python or Perl script run from a project folder that builds the path itself) is not caught. Inline Python or Perl that names central asks like any other command.
- Edit rules on research folders (`phases/*/research/`) cover Claude's file tools and recognised shell writes, not files that a program such as `node` or a script writes itself. `cp` and `mv` into them are refused everywhere by the `deny` rules and the shell guard.
- The shell secret denies cover `cat` only: `head`, `grep`, `less` and other readers are not denied. The `Read(...)` denies cover Claude's file tools.
- The PII guard catches pattern-shaped identifiers, not names or street addresses, and does not inspect shell commands. Outward shell commands (`curl`, `wget`, `scp`, `rsync`, `ssh`, `nc` and similar) are refused instead, so web reads go through WebFetch, which it does inspect. The privacy reviewer covers the rest after the fact.
- The current hooks documentation says a `PreToolUse` command hook that times out does not block the call. The guards finish in well under a second, so the 15-second timeout is a margin, not a control; re-check this after a Claude Code upgrade.
- The documentation does not state a PATH requirement for hooks. Check with `command -v node` in the shell Claude Code uses; if a hook cannot find node, restart Visual Studio Code after installing Node.js.
- `/plugin install` inside a session is not a shell command, so no permission rule gates it; the panel's own confirmation is the only check. `Bash(claude plugin install *)` gates installs from a shell only.
- "Every task maps to a card" and the door refusals depend on the AI following the skills; reviewers catch violations afterwards.
- The AI cannot verify who is at the keyboard; approvals are recorded, not authenticated. The administrator role becomes hard only when central's remote allows only the administrator to push.
- Tone checks are only as good as the tone files: until a written tone guide replaces each draft, they check against drafts.
