---
title: Porting to other AI tools
description: How to reproduce each layer of the governance system in AI assistants other than Claude Code, which pieces are portable as-is, which need an equivalent mechanism, and what to do when no equivalent exists.
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

# Porting to other AI tools

The method is tool-independent. About 70 per cent of the system is plain Markdown and Node scripts that any assistant can read and run. The rest relies on Claude Code mechanisms (imports, plugins, subagents with tool lists, hooks, permissions) that other tools offer in different forms, or not at all. Feature sets change quickly: **check the target tool's current documentation** before mapping.

## Portable as-is

| Piece | Why it ports |
|---|---|
| Design process, Define sentence, AI boundaries | Plain text rules |
| Kanban spec and the board in PRD.md | Markdown; any assistant can apply the doors |
| PRD and project template | Files and folders |
| Privacy, accessibility, stack and Markdown standards | Plain text |
| Business funnel, tone files, brand rules, avoid-list | Plain text |
| Tools (`check-md`, `new-project`, retrofit) | Node scripts run from any terminal or CI |
| `.gitignore`, pnpm workspace rules | Enforced by git and pnpm themselves |
| Prompt presets | Paste into any chat |

## Layer-by-layer mapping

| Claude Code layer | Generic equivalent | What to do |
|---|---|---|
| `~/.claude/CLAUDE.md` with `@` import | The tool's global or user instruction file, or a repository instruction file such as `AGENTS.md` | If the tool cannot import, generate the file from central with a script (concatenate core rules + user identity) and re-run it after each governance change. Never hand-edit copies |
| Project `CLAUDE.md` | The tool's project instruction file | Same five or six fact lines |
| Skills | Custom commands, saved prompts, prompt files, custom modes or "rules" files that load on request | Keep each skill's text intact; put the "user only" skills behind commands that only a person triggers |
| Subagents (reviewers) | Separate chat or agent configured with read-only access; a second model call with only file-reading tools; or a script that runs the model with the reviewer prompt | Keep the output format and the "zero files inspected = invalid" rule |
| Agent `tools:` lists | Tool or mode permissions, read-only modes | If no tool scoping exists, review is **Guided**, not Hard: say so in ENFORCEMENT.md |
| Model routing (Opus vs Sonnet) | Model selection per mode or command | Most capable model for orchestration and review; smaller for mechanical |
| `PreToolUse` hooks | Pre-execution hooks, approval policies, a proxy or wrapper around outbound calls | Reuse the Node scripts if the tool can call a command with the tool input as JSON on stdin; otherwise move the PII check to a git pre-commit hook and CI (after-the-fact) |
| `SessionStart` hook | Startup script or a "status" command | Run `version-check.mjs` manually at the start of work, or from the editor's task runner |
| Permissions `ask`/`deny` | The tool's command approval list, sandbox or allow-lists | Reproduce the lists; for anything unsupported, add OS-level protection (read-only file permissions on central for non-administrators, protected branches) |
| MCP design-tool connection | The tool's MCP client support | Most current assistants support MCP; keep write tools on approval |
| Plugin marketplace | Shared repository of prompt and config files | Distribute central via git; an install script copies or links config into the tool's folders |

## Hard guarantees without the AI tool's help

When the target tool has no hooks or permissions, move enforcement out of the assistant:

| Rule | Tool-independent enforcement |
|---|---|
| Central changes only by the administrator | Protected main branch; only the administrator can merge; file system read-only for others |
| No raw research in git | `.gitignore` plus a pre-commit hook refusing paths under `phases/*/research/` |
| No personal information committed | Pre-commit hook running `pii.mjs` patterns over staged text; privacy reviewer in CI |
| Markdown standard | `check-md` in pre-commit and CI |
| Exact pins and release age | pnpm workspace settings (enforced by pnpm) |
| Secrets | Secret scanning in CI; OS permissions on credential files |
| Kanban doors | A small script that parses PRD.md and fails CI on a card missing sections, more than one FALSIFIER line, or Doing > 3 |

## Porting checklist

1. Decide where the core rules load from and how they stay in sync with central.
2. Map each skill to a command or saved prompt; mark the five user-only skills.
3. Implement the reviewers with read-only access and the fixed output format.
4. Decide, rule by rule, where enforcement now lives; update ENFORCEMENT.md strengths honestly.
5. Connect the design tool via MCP with write tools on approval.
6. Add pre-commit and CI checks for anything that lost hard enforcement.
7. Run Phase 10 of the [implementation guide](16-Implementation-Guide.md#phase-10-verify-the-whole-system) in the new tool.
