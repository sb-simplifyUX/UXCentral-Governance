---
title: Build order and lessons
description: The recommended order for building the UX central governance system, version by version from 1.0.0 to 1.4.0, with the working method for the AI assistant, the decisions worth copying and the lessons behind the rules.
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

# Build order and lessons

This chapter gives the order in which to build the system, version by version, so each step rests on the one before. One administrator working with Claude Code can build it over several working sessions, governed by its own kanban board from the first hour. The lessons at the end explain why the rules read as they do.

## What you need before you start

- The design process in [04](04-UX-Design-Process.md) and the kanban method in [05](05-Kanban-Method.md), both created by the author.
- A design system in a design tool, if you have one ([09](09-Design-Tool-Connection.md#alternatives-to-figma) covers working without one).
- A list of any UX projects you already have. The system can retrofit them later without changing what they hold.

The tone files and the lifecycle funnel are drafted during the build; nothing needs to exist beforehand.

## Working method with the assistant

1. **Plan first, approved as a whole.** The administrator approves one plan that names the outputs, the board and the first release. Every card log then records `approved by <ADMIN>, <YYYY-MM-DD>, with the plan`.
2. **Central governs itself.** Central has its own `PRD.md` with a board. The administrator is its decider. Every change is a card.
3. **Opus orchestrates and reviews; Sonnet implements.** Mechanical subagents run on a smaller model; reviewers run on the most capable model at high effort.
4. **Reviewers before "done".** Nothing is called done until all six reviewers return APPROVE, the Markdown checker passes and the plugin validator passes.
5. **Dry runs before writes.** A retrofit first produces a plan and a diff, with file hashes proving the dry run changed nothing.
6. **Every lesson becomes a rule or a test.** When a problem surfaces, it becomes a line in the core rules, a check in a reviewer, a refusal in a script or a test case.

## Central's board (first release)

| Outcome | Description |
|---|---|
| R1-O1 | Central rules, specs, plugin and settings exist and pass their reviewers |
| R1-O2 | New projects are created from the template, wired to central |
| R1-O3 | Existing projects can be retrofitted non-destructively |
| R1-O4 | Own libraries have a versioned home fed by the design-system variables |

Each card below is shown by its one line and its FALSIFIER. From these lines the AI drafts the full nine sections of each card ([05](05-Kanban-Method.md#the-nine-card-sections)), and the administrator approves them.

| Card | In one line | FALSIFIER |
|---|---|---|
| C-001 | Build and evolve central governance | A reviewer still requests fixes after two loops on a central file |
| C-002 | Create a sample project from the template and review it | The generated project still contains an unfilled `{{PLACEHOLDER}}` after the brief is drafted |
| C-003 | Dry-run a retrofit of any existing projects | Any file hash in any existing project differs after the dry run |
| C-004 | Generate brand tokens from the design-system variables | A generated token value differs from its design-tool variable value |
| C-005 | Review the draft tone file with the person who approves the brand's voice | Copy reviewed against the new files is still labelled "tone unverified" |
| C-006 | Apply the approved retrofits | Any original file content is missing from both the project and its backup |

## Build order, version by version

Build in this order. Each version adds one layer and leaves the earlier ones working.

### 1.0.0: the foundation

- Core rules file, kanban spec, Markdown standard, privacy, accessibility (WCAG 2.2 AA), stack rules, brand rules, a draft tone of voice, front-end avoid-list.
- Plugin with 9 skills, 6 read-only reviewers, 1 mechanical retrofit helper, a PII guard hook and a session-start version-check hook.
- Project template in central: phase folders `phases/01-understand/` to `phases/06-deliver/`, with a `research/` folder (local only, never committed) in each of the first five and `prototypes/` in `phases/06-deliver/` instead; README, PRD, CLAUDE.md, project settings and `.gitignore`.
- Tools: `check-md`, `new-project`, `retrofit-audit` (dry run), `retrofit-apply`.
- `libraries/` pnpm workspace for tokens and UI components.
- User config: imports the central rules, sets the orchestration model and medium effort, defaults subagents to the smaller model, adds safety permissions.
- Any project created before 1.0.0 needs a retrofit.

### 1.1.0: the design-library map

- Read the design system live through the design tool's MCP server and map it: pages, components, variable collections, and modes and breakpoints where the library has them.
- A `figma-library-check` skill re-verifies the map; a refresh is data, not a rule change, so it needs approval and a commit but no version bump.
- The session-start hook reminds when the map's 14-day check is due.
- The Markdown standard allows extra frontmatter fields when a tool reads them and the file documents them.

### 1.2.0: roles

- Three roles, so that no rule names a person: **user** (whoever runs the assistant, named by the installer), **decider** (one per project), **administrator** (owns central).
- The installer asks the user's name and how to address them. The template and tools require `--decider` and `--decider-address`.
- Prompt presets for new project, retrofit and PRD from research.
- New skill `prd-from-research`: a de-identified research summary and a PRD draft with a source, `[inferred]` or `TBC` mark on every claim, nothing written until the decider approves.
- Every approval names the decider: review fixes, kanban block and unblock, release opening and safety-pass manual checks.
- One masking rule across reviewers: names and street addresses are replaced, other identifiers masked.

### 1.3.0: hardening the retrofit and the edit-inline rule

- New core rule: the assistant changes files only with its Edit and Write tools and reads the change back; it never edits through the shell.
- New `central-guard` hook: it asks before shell commands that would change central, which the `Edit` permission rule cannot see.
- `retrofit-apply` becomes two-phase: plan and check every write first, then write. A refusal leaves the project unchanged and creates no backup.
- A long list of refusals (exempt files, non-UTF-8, placeholders, invalid values, aliased paths, locked files, overlapping renames…).
- Frontmatter merges insert lines without rewriting existing bytes; backups never overwrite.
- A test suite with fixture projects in a temporary folder.

### 1.3.1: privacy and robustness fixes

- Retrofit reports no longer carry a file's existing frontmatter values.
- Research files whose names may hold a personal name get a "fix by hand" item for the decider.
- Files with personal-information hits but no `contains_pii: true` flag are listed and get a fix-by-hand item.
- More validation of command-line input and output folders.

### 1.4.0: refuse instead of ask

- New `shell-guard` hook: it denies shell commands that write, copy, move or bulk-delete files, run inline code, reach the network, download and run packages, publish, or discard git work, and its reason names the tool to use instead. The edit-inline rule of 1.3.0 becomes Hard.
- Permissions are split by what they cost. `deny` holds what must never happen and interrupts nobody; `ask` holds only approval gates (deleting one named file, push, install, central and user-config edits, design-tool writes, Opus subagent calls), because an ask prompts the person even in auto mode.
- `curl`, `wget` and the other outward shell commands move from `ask` to `deny`; web reads go through WebFetch, which the PII guard checks. `npx`, `pnpm dlx` and `npm exec` are refused: a tool is an exact-pinned dependency run with `pnpm exec`.
- `central-guard` lets ordinary work in central run without a prompt: `git diff`, `log`, `show`, `add`, `commit` and `fetch`, the tool tests and the plugin validator. Push, pull and the write tools still ask.
- The setup preset no longer has the assistant fetch the blueprint; the person copies the guide's repository into a new, empty folder and gives the assistant that folder.
- No retrofit needed: projects are unchanged. Re-run the installer to merge the new permission lists. Old `ask` entries that are now `deny` stay in the user's settings but no longer prompt, because deny rules are checked first.

## Decisions worth copying

| Decision | Why |
|---|---|
| Rules short in the core file, procedures in skills | The core file loads every session; long files cost context and get ignored |
| Plugin installed from a local-folder marketplace in central, so it loads from central's folder | A `git pull` in central reaches every project on that machine at the next session or after `/reload-plugins`; nothing is copied |
| Reviewers read-only by tool list, not by instruction | An instruction can be ignored; a missing tool cannot be used |
| A reviewer return with zero tool calls is invalid | A reviewer that read nothing is guessing |
| Two fix loops, then escalate to a person | Stops endless AI-to-AI loops |
| The release-closing job can be started only by a person | Closing is accountability; it belongs to the decider. The AI could still edit `PRD.md` directly, so [14](14-Hooks-Permissions-Enforcement.md) grades this below Hard |
| The design-library map is a guide, a pinned search is the proof | Maps go stale; the live library wins |
| Status shown by shape as well as colour | Accessibility, and status that survives printing |
| Hooks fail closed (PII, shell) or fail towards asking (central) | A broken guard must not silently allow |
| Refuse with `deny`; ask only at approval gates | A refusal reaches the AI and costs nobody a prompt; an ask reaches the person every time, even in auto mode, so a long ask list ends autonomy |
| Files change only through the file tools | An Edit fails when its anchor does not match and can be rewound; a shell edit can match nothing and still succeed, or change every file at once |

## Lessons learned

1. **Instructions are not enforcement.** Anything that must never happen needs a permission rule, a hook or a script refusal. Keep an honest "known limits" list for the rest.
2. **Name roles, not people.** A rule that names a person stops working when that person is away or leaves, and leaves everyone else unsure who approves.
3. **Make refusals teach.** A refusal must quote the failing line and say what would let it through, or people work around it.
4. **Typed escape sequences can become raw characters.** In code written by the assistant, build invisible characters (line separators, the byte-order mark) from character codes instead of typing `\u` escapes.
5. **Retrofits must be two-phase.** Check every write before the first write, so a refusal never leaves a half-changed project.
6. **Reports leak too.** A dry-run report that copies a file's frontmatter can copy personal information; report counts and masked samples only.
7. **Heuristics need their limits written down.** A check for personal names in file names will miss single names and flag harmless pairs; say so.
8. **Treat an AI's study as a checklist, not evidence.** (From [a kanban case study](https://sebastianbourges.com/case-studies/simplifyux-kanban/): none of the study's published scores could be reproduced from its own rubric, and the one idea that held was a hard work-in-progress limit.)
