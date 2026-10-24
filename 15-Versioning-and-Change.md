---
title: Versioning and change management
description: How central governance is versioned, how changes are proposed, checked, committed and announced, how projects record and catch up with versions through retrofit, and how data refreshes differ from rule changes.
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

# Versioning and change management

## Version numbers

`MAJOR.MINOR.PATCH`, stored in `<CENTRAL>/VERSION` and mirrored in `plugin.json`.

| Bump | When | Retrofit needed |
|---|---|---|
| MAJOR | Projects must be retrofitted: PRD or card schema change, new required frontmatter field, template structure change | Yes |
| MINOR | A new rule, skill, agent, avoid-list entry, library package or template file that projects can adopt later | No |
| PATCH | Wording, typo, clarification, or a fix that makes a tool do what the rules already say; no new rule | No |
| (none) | A data refresh such as the design-library map, approved and committed (message `design-library map: verified YYYY-MM-DD`) | No |

## Where versions are recorded

| Place | Field |
|---|---|
| Central | `VERSION`; `plugin.json` `version`; each governance file's `governance_version` (the version it was last changed under) |
| Project | `PRD.md` frontmatter `governance_version` (the version it was created or retrofitted to); every Markdown file's `governance_version` |
| Git | Tag `v<version>` on each release commit |

The session-start hook compares the project's version with central's and suggests a retrofit when they differ.

## Changelog format

```markdown
## 1.0.0 (YYYY-MM-DD)

- <one line per change, naming the file or tool>
- Tests: <count> pass
- Retrofit needed: no. <any instruction for projects>
```

Newest entry at the top. Every entry ends with "Retrofit needed: yes" or "Retrofit needed: no". A new install's first entry is `1.0.0`.

## Change procedure (governance-change)

1. **Propose** the change, files and bump; the administrator approves.
2. **Change**: edits; `VERSION` and `plugin.json` match; changelog entry; frontmatter `updated` and `governance_version`; `ENFORCEMENT.md` when a rule moves or is added.
3. **Check**, from `<CENTRAL>`:
   - Markdown checker on central: `node tools/check-md.mjs governance standard-processes prompts README.md PRD.md CHANGELOG.md ENFORCEMENT.md`
   - Markdown checker on the template: `node tools/check-md.mjs project-template --template`
   - `claude plugin validate <CENTRAL>/marketplace`
   - Tool tests when tools change: `node --test 'tools/test/*.test.mjs'`
   - Reviewers for the area (all for MAJOR).
4. **Commit** on approval with message `governance <version>: <summary>`; tag `v<version>`. Pushing needs separate approval each time.
5. **Tell**: new version, retrofit needed or not, projects behind.

## Central's own board

Central has its own `PRD.md` with release, outcomes and cards; the administrator is its decider. A governance change serves a card, and a card's scope extension is logged with approval (`scope extended to governance <version>: <what changed>, approved by <ADMIN>, <YYYY-MM-DD>`).

## Retrofit: bringing a project up to date

Non-destructive by design. Detailed in [12](12-Skills-Catalogue.md#retrofit-project); the tool contract is in [Appendix C](appendices/C-Hook-and-Tool-Sources.md).

```text
Dry run (read-only)  →  report: plan.md, inventory.json, changes.diff  in <WORK>/reports/<YYYY-MM-DD>-<Project name>/
   ↓ helper agent refines; reviewers check the report
Decider approves all, some or none (recorded in inventory.json > approved)
   ↓
Apply: plan and check every write → back up to <WORK>/backups/<YYYY-MM-DD>-<Project name>[-n]/ → write
   ↓
Body changes by Edit, one at a time; "fix by hand" items by a person
   ↓
Markdown check and reviewers again; report what changed and where the backup is
```

Guarantees:

- Frontmatter is inserted on top; existing bytes, byte-order marks and line endings stay as they were.
- Nothing is deleted. Nothing is overwritten. Backups are never overwritten (each run needs an empty folder).
- A refusal leaves the project unchanged and creates no backup.
- A project cannot be set past Define without an agreed Define sentence; earlier steps are marked passed only with evidence, otherwise skipped with approval.
- Reports contain counts and masked samples only.

Re-run the audit before applying a retrofit planned under an earlier version.

## Releasing central to other machines

Central is a git repository on your private git host. Each machine clones it to `<CENTRAL>`, runs the installer and installs the plugins (see [16 Phase 11](16-Implementation-Guide.md#phase-11-onboard-other-people)).

A release updates the plugin too: `VERSION` and the `version` in `plugin.json` move together, and the commit carries both. On each machine:

1. `git pull` on the main branch. A plugin installed from a local-folder marketplace is loaded from that folder, so the pull reaches rules, skills, agents and hooks at the next session or after `/reload-plugins`.
2. If a change does not appear, run `/reload-plugins`; if it still does not, reinstall or update the plugin (see `claude plugin --help` for the update command).
3. If the release changed `user-config/settings.json` or `user-config/CLAUDE.md`, re-run the installer (it merges and backs up).
