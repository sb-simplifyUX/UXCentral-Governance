---
title: "Appendix E: glossary"
description: Definitions of every term used in the UX central governance system, from administrator to WCAG.
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

# Appendix E: glossary

| Term | Meaning |
|---|---|
| Administrator | The one person who owns and approves changes to central governance |
| Agent (subagent) | A separately configured AI worker with its own instructions, tool list and model; here, reviewers and the retrofit helper |
| AI boundaries | The list of what the AI may and must not do; "an intern, not an oracle" |
| Anthropic's official marketplace | `claude-plugins-official`, the plugin marketplace Claude Code adds by default; source of the `frontend-design` plugin and of the `figma` plugin, which supplies the Figma MCP server |
| Ask permission | A Claude Code permission rule that makes a tool call wait for the user's approval |
| Avoid-list | Named generic front-end patterns not allowed unless the design system itself uses them |
| Board | The `## Kanban` section of a PRD: release, outcomes, columns, cards |
| Brand-agnostic | One design and component API for all brands, with per-brand token bindings and copy |
| Brand exception (BX-nn) | A recorded, approved requirement that differs by brand |
| Business funnel | Central's reference map of the customer lifecycle for the organisation's field (`standard-processes/business-funnel.md`): stages, customer pathways, KPIs and relative value. Reference material, not a rule |
| Card (C-nnn) | A unit of work with nine sections, serving one outcome |
| Central | The governance repository: the single source of truth |
| Claim marks (`[research]`, `[inferred]`, `TBC`) | Marks on a PRD drafted from research: `[research]` is stated in a source, `[inferred]` is the AI's reading of several sources, `TBC` means the research has no answer |
| CLAUDE.md | Claude Code's instruction memory file; user, central core rules and project facts |
| Data refresh | An approved, committed update to data in central, such as the design-library map, that changes no rule; no version bump |
| Decider | The one person per project who approves everything in it |
| Define sentence | "[customer] needs a way to [need], because [what we learned]." The agreed problem every later decision traces to |
| Deny permission | A permission rule that blocks a tool call outright |
| Design-library map | A central file describing the design-system files; a guide, verified by pinned search |
| Doing limit | At most three cards in Doing at once; a fourth is refused and the three cards holding the slots are named |
| Doors | The five checks a card or release passes: Create, Start, Finish, Close outcome, Close release |
| Double Diamond | The shape of the process: open up and narrow down on the problem, then on the solution |
| Dry run | A read-only audit that produces a plan and diff without changing anything |
| Effort | Claude's reasoning depth setting: low, medium, high, xhigh, max. Availability depends on the model; the `effortLevel` setting accepts low to xhigh |
| Emerging problem statement | An Understand-stage draft in the Define format; not a Define candidate |
| ENFORCEMENT.md | The matrix of every rule, what enforces it and how strongly |
| Exit criteria | Conditions a process step must meet, evidenced by files, before moving on |
| FALSIFIER | One line on a card naming what failure would look like; two falsifiers means two cards |
| Fix by hand | A retrofit item the tools will not apply (an unsafe file, personal information in a file without `contains_pii: true`, a research file name that may hold a personal name); a person fixes it, then the audit is re-run |
| Fix loop | Reviewer findings fixed and re-checked; escalated after two loops |
| Frontmatter | The YAML block at the top of a Markdown file holding its metadata |
| Governance version | `MAJOR.MINOR.PATCH` of central; recorded in every project's PRD |
| Guided / Checked / Hard | Enforcement strength: instruction only / verified after / blocked or asked mechanically |
| Hard stops | What must not happen whatever the solution: a required PRD section, and the ninth section of every card (what must not happen while doing it) |
| Held for next release | The list in a release block where new ideas wait while the release is open, because its outcomes are fixed |
| HMW (How might we) | A question that turns a problem into an opportunity to design for ("How might we ...?"); drafted in Define as a starting point for Sketch |
| Hook | A script Claude Code runs on an event (session start, before a tool call) |
| Human gate | A point where the AI prepares materials and stops for a person |
| JTBD | Jobs to be done: what a customer is trying to achieve |
| Kanban | The board method with releases, outcomes, nine-section cards and doors |
| Locale | Language and region of customer copy (spelling, formats, `lang` tag) |
| Marketplace | A catalogue of Claude Code plugins; here a local folder in central |
| MCP | Model Context Protocol: how the AI connects to external tools such as the design tool |
| Outcome (Rn-On) | A change for customers or the business that a release commits to; 2 to 5 per release |
| Own libraries | The organisation's shared token and component packages (`<LIB_SCOPE>/tokens`, `<LIB_SCOPE>/ui`) in central `libraries/`; used first, and a missing component is built there before any project uses it |
| Participant ID | P01, P02…: how research participants are referred to |
| Personal information (PII) | Anything that identifies a customer; stays in its research file |
| Phase index (`_index.md`) | The file in each step folder of a project listing the step's goal, exit criteria as checkboxes, expected outputs, what the AI may do and the human gate |
| Pinned search | A design-system search restricted to the allowed library key |
| Plugin | A package of skills, agents and hooks for Claude Code |
| POV (point of view) | A sentence naming a customer, their need and the insight behind it; the Define sentence is this system's POV |
| PRD | The project's control document: objectives to kanban |
| Process gate | The `process-gate` skill: it states the current step, checks its exit criteria against files, names skipped steps, and advances a project only after the process-gate reviewer has run and the decider approves |
| Proposal | A labelled substitute for a missing token or component |
| Release (Rn) | A set of 2 to 5 fixed outcomes; closed only by the decider with a safety statement |
| Research folder | A `research/` folder in each of a project's first five step folders (`phases/*/research/`) holding raw research; git-ignored, local only, never committed |
| Retrofit | Bringing an existing project up to the current governance version, non-destructively |
| Reviewer | A read-only agent returning APPROVE or REQUEST FIXES with evidence |
| Safety pass | Privacy, accessibility, brand and open-findings checks before a release closes |
| Safety statement | The decider's own words recorded when closing a release |
| Sketch file | A separate design file (screens) or whiteboard file (storyboards and flows) where Sketch work goes, named or approved by the decider; never the library file |
| Skill | A packaged procedure (SKILL.md) invoked as a slash command or by the AI |
| Tone unverified | Label on copy written against a draft tone file |
| Token tiers | The three layers of design tokens: primitive (raw values, never bound directly), semantic (roles such as colour, spacing and typography) and component (values that belong to one component) |
| User | Whoever runs the AI session, named by the installer |
| WCAG 2.2 AA | The accessibility standard every customer-facing screen meets |
