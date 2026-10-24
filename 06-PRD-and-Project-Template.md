---
title: PRD and project template
description: The project folder template, the required PRD sections and frontmatter, the phase index files with their exit criteria, and how projects are created, drafted from research and retrofitted.
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

# PRD and project template

## Project folder structure

Every project is created from `<CENTRAL>/project-template/`:

```text
<PROJECTS>/<Project name>/
├── README.md              at a glance, background, folder map, commands
├── PRD.md                 objectives → kanban; the project's single control document
├── CLAUDE.md              project facts only (no rules)
├── .gitignore             ignores phases/*/research/, secrets, build output
├── .claude/settings.json  ask before editing research; deny reading .env
└── phases/
    ├── 01-understand/
    │   ├── _index.md
    │   └── research/          raw research, local only, never committed
    ├── 02-define/             (_index.md, research/)
    ├── 03-sketch/             (_index.md, research/)
    ├── 04-decide/             (_index.md, research/)
    ├── 05-validate/           (_index.md, research/)
    └── 06-deliver/
        ├── _index.md
        └── prototypes/        front-end prototypes
```

There are five research folders, one in each of `phases/01-understand/` to `phases/05-validate/`; `phases/06-deliver/` holds `prototypes/` instead. The research folders are local only and never committed. Empty folders hold a `.gitkeep` in central; the `new-project` tool re-creates them, because git ignores the research folders.

File naming: `<nn>-<step>-<topic>.md`, lowercase with hyphens, for example `02-define-problem-statement.md`.

## PRD.md

### Frontmatter

```yaml
---
title: <Project name> PRD
description: Objectives, scope, decisions and the kanban board for <Project name>.
doc_type: prd
project: <Project name>
process_step: understand
brand: all
status: draft
owner: <decider>
created: YYYY-MM-DD
updated: YYYY-MM-DD
governance_version: 1.0.0
contains_pii: false
decider: <full name>
decider_address: <how to address them>
---
```

`governance_version` is the version of your own central install that the project was created or retrofitted under; a new install starts at `1.0.0`.

### Required sections (any order)

These fourteen sections are required in every PRD, central's included, and `prd-card-reviewer` check 1 checks for exactly this list.

| Section | Content | Who fills it |
|---|---|---|
| Objectives | What we want to achieve | Brief, or research draft |
| Goals | Measurable or observable aims | Brief, or research draft |
| Scope | What is included | Brief |
| Not in scope | What is excluded | Brief |
| Hard stops | What must not happen whatever the solution | Brief |
| Outputs | What the project produces | Brief |
| Definition of done | How we know it is finished | Brief |
| Success measures | Table: Measure, Baseline, Target, Source. Drafted early, confirmed in Deliver | Brief, research, Deliver |
| Named decider | Name, address form, handover rule | Brief |
| Current process step | Current step and the step table (Step, Status, Gate passed, Approved by) | Process gate |
| Define sentence | Status, Sentence, Evidence, Agreed by / date | Decider agrees |
| Decision record | Option table (Option, Likely benefit, Likely cost, Traces to Define), Winner, Why it won, Why others did not, Decided by / date | Decider |
| Brand applicability | Default (all brands), exception table (ID, What differs, Brand, Why, Source, Approved by) | Decider approves exceptions |
| Kanban | Release block, Board, Cards | Kanban and release skills |

Empty sections hold `TBC` or the template's explicit "not yet" wording, never a leftover `{{PLACEHOLDER}}`.

### Claim marks when drafted from research

| Mark | Meaning | After approval |
|---|---|---|
| **[research]** | Stated in a source (cite the summary theme) | Replaced by an `Evidence:` line pointing to the summary |
| **[inferred]** | The AI's reading of several sources | Kept, so assumptions stay visible |
| **TBC** | The research has no answer | Kept |

Never invent numbers, owners or dates.

## README.md and CLAUDE.md

- **README.md**: at-a-glance table (brands, decider, current step and board pointers, governance), background, folder map, commands table for the skills.
- **CLAUDE.md**: five or six bullet lines of project facts (summary, brands, decider, where the step and board live, governance version location, where raw research lives). It never repeats a rule.

## Phase index files

Each `_index.md` has: Goal, Exit criteria (checkboxes), Expected outputs, AI may, Human gate. Key exit criteria:

| Step | Exit criteria |
|---|---|
| Understand (`01-understand`) | Who experiences the problem, what they are trying to do, what evidence supports it, why it matters; research summary written |
| Define (`02-define`) | The customer, the need and what we learned in one sentence; cites Understand evidence; the team can say it; decider agreed it; customer types, a map of the customer's path and sharper questions for Sketch written |
| Sketch (`03-sketch`) | Many genuinely different concepts; no early convergence; no ranking language |
| Decide (`04-decide`) | Decider picked 2 or 3 options; benefit and cost per option; winner and reasoning recorded; assumptions known |
| Validate (`05-validate`) | Evidence customers understand the solution, tasks achievable, risks addressed, or design changes identified |
| Deliver (`06-deliver`) | Every state for every screen; accessibility requirements per screen with accessible names and announcements; business rules; success measures with baseline, target, source; design QA plan with keyboard and screen-reader checks; launched and tracked; learnings documented; new questions logged |

## Project settings and git

`.claude/settings.json`:

```json
{
  "permissions": {
    "ask": ["Edit(**/phases/*/research/**)"],
    "deny": ["Read(**/.env)", "Read(**/.env.*)"]
  }
}
```

`.gitignore` ignores `phases/*/research/`, `.env`, `.env.*`, key and certificate files, `.npmrc`, `.claude/settings.local.json`, `CLAUDE.local.md`, `node_modules/`, `dist/`, `.astro/` and OS files.

## Project lifecycle

| Situation | Skill | Result |
|---|---|---|
| New project | `/<PLUGIN>:new-project <name>` + brief | Folder created by script; README and PRD drafted from the brief; starts at Understand with no Define sentence and no release |
| Project has research | `/<PLUGIN>:prd-from-research` | De-identified research summary plus PRD sections, each claim marked, written only after the decider approves |
| Project exists but predates governance | `/<PLUGIN>:retrofit-project` | Dry-run plan and diff; backup; applies only approved items |
| Start delivering work | `/<PLUGIN>:release open` | 2 to 5 outcomes fixed |
| Move a step | `/<PLUGIN>:process-gate advance` | Exit criteria checked, reviewer run, decider approves |

The copy-and-paste briefs for these are in [Appendix D](appendices/D-Prompt-Presets.md). Full template files are in [Appendix A](appendices/A-File-Templates.md).
