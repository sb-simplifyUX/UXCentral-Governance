---
title: Overview and architecture
description: What the UX central governance system is, the problem it solves, its five layers, the folder layout and how every project inherits the rules without copying them.
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

# Overview and architecture

## The problem it solves

A UX team that works with an AI coding assistant across many projects tends to meet the same failures:

- Rules are pasted into each project and drift apart.
- The assistant jumps to solutions before the problem is understood.
- The assistant judges ideas, makes decisions and writes personas that a person should own.
- Tasks are declared "done" because they feel finished, not because anything proves it.
- Design-system tokens and components are invented when a real one cannot be found.
- Customer personal information from research can leak into documents, prompts and web requests.
- Nobody can say which version of the rules a project follows.

The system answers each of these with one central source of truth, a fixed design process, a kanban board that refuses bad moves, read-only reviewers, and mechanical guards that block what instructions alone cannot.

## Principles

1. **Central is the single source of truth.** Projects hold only project facts. Rules are written once.
2. **Find the right problem, then the right solution.** The six-step process runs in order, and Define cannot be skipped.
3. **AI is an intern, not an oracle.** It does volume work; people keep judgement, accountability and conversations with other people.
4. **One named decider per project** approves everything in that project.
5. **Every task maps to a card**, and every card states what failure would look like.
6. **Prevent at the door, check after the fact.** Hard guards where possible, reviewers where not, and an honest list of what is only guided.
7. **Never invent.** Not evidence, not tokens, not components, not approvals.
8. **Personal information stays where it arrived.**
9. **Non-destructive by default.** Plan and diff first, back up, never delete without approval.
10. **Versioned.** Every project records the governance version it follows.

## The five layers

| Layer | What it is | Mechanism (Claude Code) | Portable equivalent |
|---|---|---|---|
| 1. Core rules | A short file of durable rules (under 150 lines) | User-level `~/.claude/CLAUDE.md` imports `<CENTRAL>/governance/CLAUDE.md` with an `@` import, so every session loads it | The tool's global or system instructions file |
| 2. Specs and standards | Kanban spec, Markdown standard, privacy, accessibility, stack, brand rules, design process, funnel, tone | Markdown files in `<CENTRAL>` that rules, skills and reviewers read on demand | Same files, referenced by path |
| 3. Procedures | Skills (step-by-step playbooks) and agents (reviewers, helpers) | A local-folder plugin marketplace in `<CENTRAL>` with one plugin, installed at user scope and loaded from that folder | Saved prompts, custom commands, custom modes |
| 4. Enforcement | Permission rules and hooks that block or ask whatever the AI decides; scripts that refuse bad input | `~/.claude/settings.json` permissions; plugin hooks; Node.js tools | Tool-specific approvals, git hooks, CI checks |
| 5. Project facts | Each project's README, PRD (with board), CLAUDE.md, settings | Created from `<CENTRAL>/project-template` | Same files |

## How a session works

```text
Session opens in <PROJECTS>/<Project name>
  │
  ├─ Core rules load (user CLAUDE.md → @import central rules)
  ├─ Project CLAUDE.md loads (project facts only)
  ├─ SessionStart hook adds a "UX governance:" line to Claude's context:
  │   governance version vs central, current process step,
  │   Doing count, the decider, and any overdue design-library check
  │
User asks for work
  │
  ├─ Process gate: which step are we in? Does this work belong to a later step?
  ├─ Kanban: which card does this serve? None → propose one, wait for the decider
  ├─ Skill runs the procedure (UI build, sketch, PRD from research…)
  ├─ PreToolUse hooks block personal information leaving the machine
  │   and ask before shell commands that change central
  ├─ Permissions ask before delete, push, install, central edits, design-tool writes
  ├─ Reviewers (read-only) return APPROVE or REQUEST FIXES with evidence
  └─ The decider approves; the approval is recorded with name and date
```

## Folder layout

```text
<ROOT>/
├── 000-central/                     ← <CENTRAL>: the governance repository (git)
│   ├── README.md                    what central is, install, change
│   ├── PRD.md                       central's own board (administrator is decider)
│   ├── VERSION                      1.0.0 in a new install
│   ├── CHANGELOG.md                 one entry per version, "Retrofit needed: yes/no"
│   ├── ENFORCEMENT.md               every rule → what enforces it, and how strongly
│   ├── governance/
│   │   ├── CLAUDE.md                core rules (imported into every session)
│   │   ├── kanban-spec.md
│   │   ├── md-standard.md
│   │   ├── privacy.md
│   │   ├── accessibility.md
│   │   ├── stack.md
│   │   └── brand/
│   │       ├── brand-rules.md       allowed library, keys, token tiers, exceptions
│   │       ├── design-library-structure.md   map of the design-system file
│   │       ├── avoid-list.md        generic front-end patterns to avoid
│   │       └── tone-<brand-1>.md    one tone file per brand
│   ├── standard-processes/
│   │   ├── UX-Design-Process.md
│   │   └── business-funnel.md       industry customer lifecycle
│   ├── project-template/            copied into each new project
│   ├── prompts/project-presets.md   copy-and-paste prompts
│   ├── marketplace/                 local plugin marketplace
│   │   ├── .claude-plugin/marketplace.json
│   │   └── plugins/<PLUGIN>/
│   │       ├── .claude-plugin/plugin.json
│   │       ├── skills/<name>/SKILL.md   (11 skills)
│   │       ├── agents/<name>.md         (7 agents)
│   │       └── hooks/                   hooks.json + 4 scripts + lib/pii.mjs
│   ├── libraries/                   pnpm workspace: <LIB_SCOPE>/tokens, <LIB_SCOPE>/ui
│   ├── tools/                       check-md, new-project; retrofit-audit, retrofit-apply, tests when built
│   └── user-config/                 CLAUDE.md, settings.json, install.mjs
├── 001-projects/                    ← <PROJECTS>
│   └── <Project name>/              one folder per project, from the template
└── _governance-work/                ← <WORK>: local only, never in git
    ├── reports/<YYYY-MM-DD>-<Project name>/   retrofit plans, inventories, diffs
    └── backups/<YYYY-MM-DD>-<Project name>/   copies made before any change
```

The leading `000-`, `001-` and `_` keep the folders sorted and make central visibly separate.

## How projects inherit without copying

| What | How it reaches a project | Why |
|---|---|---|
| Rules | Imported at session start from central | Change once, every project follows on next session |
| Skills, agents, hooks | Plugin installed from the local-folder marketplace in `<CENTRAL>/marketplace` and loaded from that folder | A `git pull` in central reaches the next session or `/reload-plugins`; keep central on its main branch between releases |
| Permissions | User-level settings written by the installer, plus a small project settings file | Machine-wide safety, project-specific research protection |
| Template | Copied once at project creation, then filled | The only thing copied; later changes arrive by retrofit |
| Version | `governance_version` in each project's PRD frontmatter | The session-start hook warns when a project is behind |

If a change from central does not appear, run `/reload-plugins`; if it still does not, reinstall or update the plugin (see `claude plugin --help` for the update command).

## What lives where: central versus project

| Central only | Project only |
|---|---|
| Rules, specs, process, funnel, tone, brand rules, design-library map | Objectives, scope, hard stops, success measures |
| Skills, agents, hooks, tools, libraries | Named decider, current step, Define sentence, decision record |
| Template, prompt presets, changelog, enforcement matrix | Kanban board and cards |
| | Research (local only), syntheses, sketches, specs, prototypes |

A rule repeated in a project file is a defect: the PRD reviewer flags it.

## Inventory of this blueprint (version 1.4.0)

| Kind | Count | Names |
|---|---|---|
| Skills | 11 | new-project, retrofit-project, prd-from-research, process-gate, kanban, release, review, figma-sketch, ui-build, figma-library-check, governance-change |
| Reviewer agents | 6 | process-gate, prd-card, brand-design-system, accessibility, code-stack, privacy |
| Helper agents | 1 | retrofit-auditor |
| Hooks | 4 | pii-guard, shell-guard, central-guard, version-check |
| Tools | 4 + tests | check-md, new-project, retrofit-audit, retrofit-apply; retrofit test suite |
| Own libraries | 2 | tokens, ui |
| Prompt presets | 5 | stored in central: new project, retrofit, PRD from research; in [Appendix D](appendices/D-Prompt-Presets.md) only: set up the system, adopt on another machine |
