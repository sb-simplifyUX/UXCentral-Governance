---
title: Skills catalogue
description: Every skill in the governance plugin with its purpose, who can invoke it, arguments, effort, steps, human gates and the checks it runs.
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

# Skills catalogue

Skills are invoked as `/<PLUGIN>:<name>`. "User only" means `disable-model-invocation: true`: the AI cannot start it; a person must type the command. Full generalised source for every skill is in [Appendix B](appendices/B-Skill-and-Agent-Sources.md).

## Summary

| Skill | Purpose | Invocation | Effort | Card needed |
|---|---|---|---|---|
| `new-project` | Create a project from the template and draft README and PRD | User only | medium | No (setup) |
| `retrofit-project` | Bring an existing project to the current version, non-destructively | User only | medium | No (setup) |
| `prd-from-research` | Draft PRD sections and a research summary from research | User only | medium | No (setup) |
| `process-gate` | State the step, check exit criteria, name skipped steps, advance | AI or user | high | n/a |
| `kanban` | Create, start, finish, block, unblock cards; close outcomes | AI or user | medium | n/a |
| `release` | Open a release, run the safety pass, prepare close | User only | medium | n/a |
| `review` | Run reviewers, fix loop, escalate | AI or user | high | n/a |
| `figma-sketch` | Ideas, storyboards, rough screens, flows in the design tool | AI or user | medium | Yes |
| `ui-build` | Build front-end UI with the design system and rules | AI or user | medium | Yes |
| `figma-library-check` | Re-verify the design-library map | AI or user | medium | Central data refresh |
| `governance-change` | Change central with version bump, changelog, review, commit | User only | medium | Central card |

## new-project

1. **Brief.** Read the pasted brief (`Label: value` lines; `<…>` or `TBC` means not answered). Ask in one batch for anything missing: decider (full name, address form), name and summary, background, brand applicability and exceptions, objectives, goals, scope, not in scope, hard stops, outputs, definition of done, draft success measures. Never invent; write `TBC`. If the user is not the decider, get the decider's approval of the brief first.
2. **Create.** `node "<CENTRAL>/tools/new-project.mjs" --name "<Project name>" --brand all --summary … --decider … --decider-address …` (for a project that serves one brand only, `--brand` takes that brand's slug, for example `--brand <brand-1>`). Refuses if the folder exists (suggest retrofit).
3. **Draft.** Replace remaining placeholders with the brief's words, tightened not embellished. Keep `process_step: understand`, Define not drafted, decision record empty, no release open.
4. **Check.** `check-md` on the project; review with `prd-card`, `process-gate`, `privacy` reviewers.
5. **Report and stop.** Folder, decider, version, TBC fields, verdicts, next steps. No design work.

## retrofit-project

**Dry run (always first):** resolve the folder; read PRD (`governance_version`, `decider`); ask for the decider if missing; run `retrofit-audit.mjs` (read-only) into `<WORK>/reports/<YYYY-MM-DD>-<Project name>/`; delegate refinement to the `retrofit-auditor` agent (paths only); list customer-detail files with counts and masked samples; review the report with privacy, PRD and process reviewers (plus code-stack if code exists); present the summary; **stop** unless the decider approves.

**Apply (after approval):** draft placeholders with the decider into `inventory.json` > `new_file_contents`; record exact approvals (`md_changes`, `new_files`, `body_changes`, `renames`) with "approved by"; run `retrofit-apply.mjs --inventory … --backup <WORK>/backups/<YYYY-MM-DD>-<Project name>[-n]`; apply approved body changes with Edit one at a time; "fix by hand" items go to a person; for a step later than Understand, check each earlier step's exit criteria and record passed or skipped with approval (Define is never skipped); run `check-md` and reviewers; report.

Notes from the preset: files under "Do not touch" get nothing but their privacy findings stay reported; a supplied Define sentence is recorded `agreed` only if in exact format and confirmed by the decider with name and date, otherwise `candidate`.

## prd-from-research

Understand-step drafting. It decides nothing.

1. **Ready?** PRD with `governance_version` and a decider; note current step and sections already written (never overwritten).
2. **Find research** in the project's research folders (`phases/*/research/`) plus paths in the notes; confirm the list if more than 15 files or unusual types; de-identify file names in every listing and citation.
3. **Read and summarise** under privacy rules 1 to 8. Use an existing participant-ID mapping or batch prefixes. More than five files: Sonnet subagents per batch, each prompt starting with the read-only line from [13](13-Agents-and-Reviewers.md#design-of-a-reviewer), paths only, privacy rules in full; zero-tool-call returns re-dispatched once. Cluster into themes (customers by situation, goals, failures, magnitudes as found, business context, open questions). Up to three **emerging** problem statements in the Define format, with evidence. Check the summary for personal information before preview.
4. **Draft PRD** sections with `[research]`, `[inferred]` or `TBC` on every statement. Never draft decider, step, Define sentence, decision record or kanban.
5. **Propose, then write**: preview; on approval back up the PRD, write `phases/01-understand/01-understand-research-summary.md` (new dated name if one exists) and only the approved sections; `[research]` becomes `Evidence:` lines.
6. **Check and report**; offer `process-gate` to see whether Understand can close.

## process-gate

Arguments: `status | check | advance | request <step>`.

- **status**: the current step in one line, what it is for, what the decider owns.
- **request `<step>`**: for later-step work, name every skipped step and its normal output; ask; record `skipped (<decider>, <date>)`. Gates that never open: Define needs `agreed` for anything after it; Decide needs a named decider who picks the shortlist; Deliver specs need every state and success measures.
- **check**: each exit criterion as met (with the file), not met, or unclear. Facts separate from assumptions; never invent evidence.
- **advance**: check, then `process-gate-reviewer` (and `prd-card-reviewer` if the board changed), then the decider's approval, then update `process_step`, the step table and `updated:`.
- **Human gates**: prepare, name the gate and who acts, stop.
- **Step specifics**: Define drafts 3 to 5 candidates with evidence; Sketch no ranking; Decide drafts benefit and cost per decider-picked option and stress-tests; Deliver every state and the learning loop.

## kanban

Arguments: `create | start | finish | block | unblock | close-outcome` and an ID. Reads the spec before applying a door. Before any project work, finds the card or proposes one and waits. Each door checks, then changes the board and logs `<action> <date> (approved by <decider>, <date>)`, or refuses quoting the line and the fix. After changes: update `updated:`, keep ▲ ■ ▬ ● markers, run `prd-card-reviewer` when asked or when more than three cards changed.

## release

User only. `open`: refuse if one is open; draft 2 to 5 outcomes with the decider (changes for customers or business, tracing to objectives or Define); write after approval; outcomes are then fixed (changes refused, ideas held). `safety-pass`: privacy, accessibility (manual checks by a named person), brand and design system, no open fixes; complete only with the decider's approval. `prepare-close`: list outcomes and evidence, confirm safety pass, ask for the safety statement in the decider's words; close only after the decider writes it and says so; held ideas become next-release candidates.

## review

Pick reviewers that fit the target (`all` for everything); run independent reviewers in parallel. Dispatch with the literal read-only line and masking line ([13](13-Agents-and-Reviewers.md#design-of-a-reviewer)), target paths (not content), and what changed on re-checks. Validate returns (tool uses, `Inspected:`, `VERDICT:`); re-dispatch invalid once, then review in the main session using the agent file as checklist. Fix loop: findings need file, location, fails, would pass; present fixes and proposed discards to the decider (central: administrator); apply approved fixes in the main session; re-dispatch only reviewers that requested fixes; **after two loops without APPROVE, escalate**. Report one table: reviewer, verdict, findings, fixed, open.

## figma-sketch

Gate: refuse unless the Define sentence is `agreed`; route through `process-gate request sketch` if the step differs; find the card. Never write to library files; the decider names or approves a Sketch file. Load the design-tool skills before their tools.

- **ideas**: at least 16 across genuinely different mental models (self-serve, assisted, automated, a change to the product, a change to the process, doing nothing differently), each one line plus the part of the Define sentence it addresses; no ranking; saved to `phases/03-sketch/03-sketch-ideas.md`.
- **storyboards**: four-panel comic strips (situation, trigger, interaction, outcome) for ideas the decider picks; situations from the business funnel; customers by situation, never invented personas.
- **screens**: low fidelity, library components and tokens only via pinned search; brand-agnostic; missing items as labelled proposals; placeholder copy in tone and locale, "tone unverified" while draft; checked against the avoid-list. `frontend-design` is not used at this fidelity.
- **flows**: simple boxes and arrows, happy path plus the main failure branch.
- Finish: list links and node ids in the sketch file, stop. Judging happens in Decide.

## ui-build

Before: card in Doing; Define agreed and Decide winner recorded (otherwise refuse or route through the gate); load `frontend-design:frontend-design` with the brand design system as the direction; read avoid-list, accessibility and stack rules. Sources in order: own libraries, then the design tool via MCP (pinned; `figma-design-to-code` before `get_design_context`; confirm every returned token), then nothing. Build in Astro and pnpm, brand-agnostic with per-brand bindings, per-brand string sets in tone and locale, every state. New dependency only with justification, exact version at least 14 days old and approval. Delegate mechanical implementation to a Sonnet subagent with a self-contained brief (files, tokens, components, states, acceptance criteria, applicable WCAG criteria, each state's accessible name and announcement). After: brand, accessibility and code-stack reviewers; propose new avoid-list entries; report proposals and tone status.

## figma-library-check

1. Have components been published since the map's dates? One session, one pinned search across sample components; compare the newest `updatedAt`.
2. What changed? Variables always (getters-only script: collections, modes, counts, viewport values); pages; components when dates moved or `full` (getters-only inventory per component page). Confirm every new or renamed name with a pinned search. Show each script to the administrator before it runs; no setters.
3. Report a short diff to the administrator.
4. Update the map body and freshness fields; `check-md`; on approval, commit with the message `design-library map: verified YYYY-MM-DD`. No version bump unless a rule changes.

## governance-change

User only; administrator approves.

1. **Propose**: the change, files, and bump. MAJOR: projects must be retrofitted (schema or structure changes). MINOR: new rule, skill, agent, avoid-list entry, package or template file. PATCH: wording, or a fix making a tool do what rules already say.
2. **Change**: edits; `VERSION` and `plugin.json` version match; changelog entry with "Retrofit needed: yes/no"; frontmatter `updated` and `governance_version` on changed files; `ENFORCEMENT.md` if a rule moved or was added.
3. **Check**: `node tools/check-md.mjs governance standard-processes prompts README.md PRD.md CHANGELOG.md ENFORCEMENT.md` and `node tools/check-md.mjs project-template --template`; `claude plugin validate`; when tools changed, `node --test 'tools/test/*.test.mjs'`; reviewers covering the area (`all` for MAJOR).
4. **Commit** on approval, staging only the files in the approved change: `governance <version>: <summary>` and tag `v<version>`. Pushing needs separate approval each time.
5. **Tell**: new version, retrofit needed or not, which projects are behind.
