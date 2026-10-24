---
title: Agents and reviewers
description: The six read-only reviewer agents and one mechanical helper agent, their tools, models and checks, the shared output format and masking rule, and the review protocol with its fix loop and escalation.
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

# Agents and reviewers

## Design of a reviewer

Every reviewer follows the same pattern:

1. **Read-only by tool list**: `tools: Read, Glob, Grep` (plus read-only design-tool tools for the brand reviewer and WebFetch for registry dates for the code-stack reviewer). It cannot edit because it has no edit tool.
2. **Two opening lines**, written word for word in the agent file and in every dispatch. The read-only line:

   > You are read-only. You may use Read, Grep and Glob (and the design tool's read tools where listed). You never edit, write, run shell commands or call write tools. If a fix is needed, describe it; the main session applies it.

   The masking line:

   > Never repeat personal information in your output: replace a name or street address with the participant ID or `[name]` / `[address]`, and mask any other identifier (first two and last two characters, the rest `*`; six characters or fewer become `****`).

3. **Read first** list: the governance files that are its checklist.
4. **Numbered checks**, each citable as `Rule: <n>`.
5. **Fixed output**, the same block in every reviewer. It opens with "Return only this, no narrative:", then:

   ```text
   VERDICT: APPROVE | REQUEST FIXES
   Scope: <what you reviewed>
   Inspected: <every file path you opened, plus any design-tool query or registry URL>
   Findings:
   1. File: <path> | Location: <line, heading, card or node> | Fails: <what> | Would pass: <what> | Rule: <check number>
   ```

   It closes with: "If there are no findings, write `Findings: none`. Report only what you verified in files." A reviewer may add lines after the block (the brand reviewer's proposed avoid-list additions, for example); it never changes the block. [Appendix B](appendices/B-Skill-and-Agent-Sources.md) writes these lines and the block out in full in every agent file.

6. **Model and effort**: Opus, high effort (reviewers are judgement work).

## The six reviewers

| Agent | Tools | Checks |
|---|---|---|
| `process-gate-reviewer` | Read, Glob, Grep | 1 step stated matches the step table; 2 order (no step passed while an earlier one is neither passed nor skipped); 3 exit criteria evidenced by existing files; 4 Define sentence format and status, `agreed` with who and when if any later step started; 5 traceability of options, cards and measures to the Define sentence; 6 no ranking language in Sketch; 7 Decide: one decider, 2 to 3 options with benefit and cost, reasoning recorded; 8 Deliver: every state, measures with baseline, target, source; 9 AI boundaries (no AI personas, decisions, sessions or replaced conversations). For retrofit plans: inferred step shown as a proposal, candidates not marked agreed |
| `prd-card-reviewer` | Read, Glob, Grep | 1 all fourteen PRD sections of [06](06-PRD-and-Project-Template.md) present, Decision record included (central's PRD is exempt from none), no leftover placeholders; 2 frontmatter complete and valid, `decider` present, one H1 matching title; 3 releases: at most one open, 2 to 5 outcomes, ID format, closed release has attributed safety statement; 4 nine card sections in order; 5 one outcome per card, existing in the open release; 6 exactly one FALSIFIER line; 7 doors: Doing count matches marker and ≤ 3, Doing cards pass Start, Done cards have evidence, closed outcomes valid; 8 board consistency and shape markers; 9 brand exception IDs exist; 10 no personal information |
| `brand-design-system-reviewer` | Read, Glob, Grep + the Figma server's read tools as the `figma` plugin supplies them: `mcp__plugin_figma_figma__search_design_system`, `get_metadata`, `get_screenshot`, `get_variable_defs` (same prefix), plus the server's session tool if searches need one (confirm every name in the live tool list) | 1 allowed library only, verified against the tool (sample of at least five names plus any doubtful); 2 no invented tokens, raw values or components unless labelled PROPOSAL; 3 brand-agnostic unless an exception ID exists; 4 avoid-list, plus proposed additions; 5 tone matches the tone file, "tone unverified" while draft, no jargon, no unbacked superlatives; 6 locale spelling. Output adds `Proposed avoid-list additions:` |
| `accessibility-reviewer` | Read, Glob, Grep | WCAG 2.2 AA checklist with SC numbers. Code: semantics, labels, autocomplete, focus, ARIA, live regions, target size, reflow, `lang`, time limits. Specs: every state with accessible name and announcement, error text says how to fix, no colour-only meaning. Contrast only from actual values; otherwise "not verifiable". Documents without UI: only check requirements are present where required |
| `code-stack-reviewer` | Read, Glob, Grep, WebFetch (public registry metadata only) | 1 toolchain pinned; 2 exact pins; 3 release age ≥ 14 days and workspace settings; 4 latest stable; 5 lockfile present and tracked; 6 own libraries first with reasons; 7 no secrets, `.env*` ignored; 8 scripts: guarded destructive operations, no network calls with project content, inputs validated |
| `privacy-reviewer` | Read, Glob, Grep | 1 no personal information in PRD, README, CLAUDE.md, cards, syntheses, reports, governance files, commits; 2 participant IDs; 3 `contains_pii: true` where personal information exists; 4 `.gitignore` excludes research and `.env*`; 5 reports show counts and masked samples only; 6 staff names noted only next to customer data. Uses the grep patterns in `<CENTRAL>/marketplace/plugins/<PLUGIN>/hooks/lib/pii.mjs` (emails, phone formats, region-specific identifiers), then opens each hit to judge context |

**WebFetch rule for the code-stack reviewer:** never a blanket WebFetch allow; allow only `WebFetch(domain:registry.npmjs.org)`. The reviewer reads release dates from the npm registry and sends no project content anywhere.

## The helper agent

| Agent | Model / effort | Job | Output |
|---|---|---|---|
| `retrofit-auditor` | Sonnet / low | Reads a project and its dry-run report; writes one-sentence descriptions, corrects `doc_type` and `process_step` with reasons, finds up to five existing problem statements and returns them as **candidate** Define sentences with sources, notes misplaced files, near-duplicates and personal information the patterns missed (masked) | JSON only: `inspected`, `frontmatter_corrections`, `define_candidates`, `missed` |

## Review protocol

```text
Pick reviewers → dispatch in parallel (paths, read-only line, what changed)
      │
      ▼
Validate each return ── invalid (0 tool uses / no Inspected / no VERDICT)?
      │                     └─ re-dispatch once → still invalid → review it yourself, say so
      ▼
Collect findings (file, location, fails, would pass; else propose discarding with reason)
      │
      ▼
Decider (central: administrator) approves fixes → main session applies them, records approval
      │
      ▼
Re-dispatch only reviewers that requested fixes ── loop count +1
      │
      └─ two loops without APPROVE → STOP, escalate with findings, what was tried, options
```

**Report:** one table (reviewer, verdict, findings, fixed, open), then open findings verbatim. No transcripts of reviewer reasoning.

## Which reviewers run when

| Moment | Reviewers |
|---|---|
| New project | prd-card, process-gate, privacy |
| Retrofit report | privacy, prd-card, process-gate (+ code-stack if code) |
| PRD from research | privacy, prd-card, process-gate |
| Process step advance | process-gate (+ prd-card if board changed) |
| UI build | brand-design-system, accessibility, code-stack |
| Release safety pass | privacy, accessibility, brand-design-system |
| Governance change | those covering the area; all for MAJOR |

## Adding a reviewer

1. Write `agents/<name>-reviewer.md` with the pattern above.
2. Give it the narrowest tool list that lets it verify.
3. Add it to the `review` skill's table.
4. Map the rules it checks in `ENFORCEMENT.md`.
5. Release as a MINOR version.
