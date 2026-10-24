---
title: Roles and authorisation
description: The three roles (user, decider, administrator), what each approves, how approvals are recorded, how roles are handed over and where staff details are stored.
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

# Roles and authorisation

## The three roles

| Role | Who | Where recorded | Approves |
|---|---|---|---|
| **User** | Whoever is running the AI session | `~/.claude/CLAUDE.md`, written by the installer (`Name`, `Address as`) | Nothing by role alone; they relay approvals from the decider or administrator |
| **Decider** | One person per project | Project `PRD.md` frontmatter: `decider` (full name), `decider_address` (how to address them, pronouns only if given) | Everything in their project: file changes, deletions, commits, cards, the Define sentence, the Decide shortlist and winner, closing releases |
| **Administrator** | One person for the whole organisation | Central `PRD.md` frontmatter: `administrator`, plus `decider` and `decider_address` (the administrator is central's decider) | Every change to central governance |

One person can hold all three roles. The system never assumes it.

## Rules the assistant follows

Quoted from the core rules:

> The decider approves everything on their project: file changes, deletions, commits, cards, the Define sentence, the Decide shortlist and winner, and closing releases. Only the administrator approves changes to central. When the user is not the person who must approve, prepare the work, stop, and ask the user to get that approval; record it as `approved by <name>, YYYY-MM-DD`. I cannot verify who is at the keyboard, so I never claim to have.

- If `decider` is missing, ask the user who it is **before any project work**.
- If only `decider_address` is missing, use the decider's name and they/them, and ask once how they would like to be addressed.
- Record either value in the PRD only once the named decider has approved it.
- Never assume the decider is the user.

## How approvals are recorded

Every approval carries a name (full name or the `decider_address` form, used consistently on a board) and an ISO date:

```text
approved by Sam Example, 2030-01-07
skipped (Sam Example, 2030-01-08)
passed 2030-01-09 (approved by Sam Example, 2030-01-09)
- **Safety statement:** "<the decider's words>" (Sam Example, 2030-01-28)
```

Where they appear:

| Event | Where the record goes |
|---|---|
| Card created, started, finished, blocked, unblocked | The card's `Log:` line |
| Outcome closed | Outcomes table, Evidence column |
| Release opened or closed | Release block `Status:` and `Safety statement:` |
| Process step passed or skipped | PRD step table `Gate passed` and `Approved by` |
| Define sentence agreed | PRD `Agreed by / date` |
| Decision made | PRD `Decided by / date` |
| Brand exception | Brand applicability table `Approved by` |
| Retrofit items | `inventory.json` > `approved` |
| Review fixes | The review report |
| Central change | Changelog entry and commit, after the administrator's approval |

## Gates that only a person can open

| Gate | Who | Mechanism |
|---|---|---|
| Starting project setup, retrofit or PRD-from-research | A person types the slash command | Skills have `disable-model-invocation: true` |
| Changing central | Administrator | `ask` permission on central edits, central-guard hook on shell commands that could change central, `governance-change` is user-invoked only |
| Closing a release | Decider | `release` skill is user-invoked only and only prepares; decider writes the safety statement |
| Deleting, pushing, installing | Decider (project) or administrator (central) | `ask` permissions for one named file, a push or an install. Bulk deletes, copies and moves are refused (`deny` rules and the shell guard), so a person does them |
| Writing to the design tool | Decider | `ask` permissions on every design-tool write tool |
| Manual accessibility checks | A person the decider names | The safety pass never records a manual check as done by the AI |

## Handover

**Decider:** change `decider` and `decider_address` in the project PRD, with a dated note under `## Named decider`, approved by the current decider.

**Administrator:** the current administrator changes `administrator`, `decider` and `decider_address` in central `PRD.md` with a dated note, and updates the core rules (Roles), `README.md` and `ENFORCEMENT.md` through the governance-change procedure. Once central is on a remote repository only the administrator can push to, the administrator role becomes hard-enforced rather than guided.

## Staff details are not customer data

The names of the user, decider and administrator, and how to address them, are staff details. Never ask for more than a name and how to address someone; record pronouns only if the person offered them; never store staff details next to customer data.

## Validating person values

Names and address forms are written into files and permission patterns, so tools validate them at the boundary. A value is refused when it is blank, starts with `--` (a missing flag value), spans more than one line, or contains control characters, `{{`, `$`, `|`, `@` or backticks.
