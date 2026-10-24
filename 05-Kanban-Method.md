---
title: Kanban method
description: The author's kanban method as used by the governance system, covering releases and outcomes, columns, the nine card sections, the FALSIFIER, the five doors, refusals and the board format in PRD.md.
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
source: https://sebastianbourges.com/case-studies/simplifyux-kanban/
---

# Kanban method

**Source:** the kanban method is Sebastian Bourges' own, documented in [a kanban case study](https://sebastianbourges.com/case-studies/simplifyux-kanban/) of a task board that a person and several AI sessions share. It was built because AI agents doing real work drift: they make changes nobody asked for, declare tasks done when they only feel finished, and two sessions start the same job. An ordinary board trusts whoever moves a card; this board **refuses bad moves**.

In the governance system the board lives in Markdown inside each project's `PRD.md` under `## Kanban`, so it needs no separate app. A dedicated board app is an option (see the case study); the rules are the same.

## Core rule

**Every task maps to a card.** Before any project work, the AI finds the card it serves. If none exists, it proposes one (all nine sections drafted) and waits for the decider's approval. Project setup (new project, retrofit, PRD from research) is the only exception, because no release can exist yet.

## Releases and outcomes

- Work is cut into **releases** of **2 to 5 outcomes**, fixed when the release opens. Outcomes cannot be added, removed or reworded while it is open; new ideas go under "Held for next release".
- Outcome IDs: `R<release>-O<n>`, for example `R1-O2`.
- An outcome is a change for customers or the business, not a task. It traces to the PRD objectives or, once agreed, the Define sentence.
- Every card serves **exactly one** outcome of the open release.
- One release open per project at a time.
- A release tracks: status, a **safety pass** (must be complete before close), and ideas held for next release.

## Columns

Each column has a **shape as well as a colour**, so status never depends on colour alone.

| Column | Marker | Limit |
|---|---|---|
| To do | ▲ yellow triangle | none |
| Doing | ■ red square | **3** |
| Blocked | ▬ black bar | none |
| Done | ● blue circle | none |

The hard Doing limit of three stops drift before it happens; every other mechanism only notices it afterwards. Two common additions are left out on purpose: self-scoring, because a score you give yourself can never go red, and a model to predict time and cost, because its constants cannot be calibrated.

## The nine card sections

Card IDs are `C-<nnn>`, unique within the project. Sections in this order:

| # | Section | Rule |
|---|---|---|
| 1 | In one line | What the card delivers, one sentence |
| 2 | Outcome served | One outcome ID of the open release |
| 3 | Process step | Understand, Define, Sketch, Decide, Validate or Deliver; central's own cards may use `n/a`, matching central's `process_step: n/a` |
| 4 | Traces to | The Define sentence, or the PRD section the card serves |
| 5 | Brand | All, or the brands it applies to (for example `<BRAND_1>` only); plus exception IDs if any |
| 6 | Done when | Observable acceptance criteria |
| 7 | **FALSIFIER** | One line naming what failure would look like. Work needing two falsifiers is two cards |
| 8 | Evidence | One line, filled at Finish; `not yet` before |
| 9 | Hard stops | What must not happen while doing this card |

> "If you cannot write the FALSIFIER line, you do not yet understand the task well enough to start it."

Good falsifiers are observable and binary: "Fewer than 3 of 5 participants find the settings page without help." "Any file hash in any existing project differs after the dry run." "A generated token value differs from its design-tool variable value."

## The five doors

| Door | Passes when | Who |
|---|---|---|
| **Create** | The card names an outcome of the open release | AI proposes, decider approves |
| **Start** | Sections 1 to 7 and 9 filled, section 7 exactly one line, Doing has a free slot | AI, after the decider's approval |
| **Finish** | Card is in Doing; sections 1 to 7 and 9 still filled; section 7 still one line; section 8 has one line of evidence (no slot check) | AI, after the decider's approval |
| **Close outcome** | Every card serving it is Done, plus one line of outcome evidence | AI proposes, decider approves |
| **Close release** | Every outcome closed, safety pass complete, the decider writes the safety statement | **Decider only** |

Block and unblock: a Doing card can be blocked when the blocker is named (frees a slot); unblocking needs a free slot. Both need the decider's approval and are logged.

## Refusals that teach

When a door refuses, quote the failing line and what would pass. When Doing is full, name the three cards holding the slots.

> Start refused for C-007. Doing is full: C-002, C-004, C-005 hold the slots. Section 7 (FALSIFIER) is empty. To pass: finish or block one Doing card, and add one line such as "Fewer than 3 of 5 participants find the settings page without help."

A refusal that does not name the failing line is often wrong or unclear; naming the exact line and the fix is what makes a refusal useful.

## The safety pass (before a release closes)

- No personal information in the PRD, cards or deliverables (privacy reviewer).
- Accessibility checked for customer-facing output (accessibility reviewer). Anything "not verifiable" is resolved by a manual check by a person (contrast tool on rendered tokens, keyboard-only pass, one screen-reader pass), recorded with their name and date. The AI never records a manual check as done.
- Brand and design-system checks passed for any UI.
- No open REQUEST FIXES from any reviewer.
- The decider approves the recorded results.

## Board format in PRD.md

```markdown
## Kanban

### Release R1: <name>
- **Status:** open (opened 2030-01-07, approved by <decider>, 2030-01-07)
- **Safety pass:** not started
- **Safety statement:** not yet (on close: "<the decider's words>" (<decider full name>, YYYY-MM-DD))
- **Held for next release:** none

| Outcome | Description | Status | Evidence |
|---|---|---|---|
| R1-O1 | … | open | not yet |

### Board
- ▲ **To do:** C-003
- ■ **Doing (2/3):** C-001, C-002
- ▬ **Blocked:** none
- ● **Done:** none

### Cards

#### C-001: <in one line>
- **Column:** Doing
- **In one line:** …
- **Outcome served:** R1-O1
- **Process step:** Understand
- **Traces to:** PRD > Objectives
- **Brand:** All
- **Done when:** …
- **FALSIFIER:** …
- **Evidence:** not yet
- **Hard stops:** …
- **Log:** created 2030-01-07 (approved by <decider>, 2030-01-07); started 2030-01-08 (approved by <decider>, 2030-01-08)
```

The session-start hook reads the `■ **Doing (n/3):**` line, so keep its exact format. An empty Doing column reads `■ **Doing (0/3):** none`, and every other empty list reads `none`; an unfilled field reads `not yet`.

## Enforcement

| Rule | Enforced by | Strength |
|---|---|---|
| Every task maps to a card | Core rules; `kanban` skill | Guided |
| Nine sections; one FALSIFIER | Spec; `kanban` skill; `prd-card-reviewer` checks 4, 6 | Checked |
| 2 to 5 outcomes fixed at open; one outcome per card | `release` skill; reviewer checks 3, 5 | Checked |
| Doors; Doing limit 3 | `kanban` skill; reviewer check 7 | Guided + Checked |
| Close release is the decider's act | `release` skill cannot be invoked by the AI, but the AI could still edit PRD.md directly; `prd-card-reviewer` check 3 | Checked |
| Board in PRD.md | Template; session-start hook reads and prints the Doing line; `prd-card-reviewer` check 8 | Checked |
