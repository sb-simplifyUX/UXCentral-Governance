---
title: Brand, tone of voice and regional communication
description: Brand-agnostic-first rules for multi-brand organisations, brand exceptions, the tone-of-voice file format, regional and locale communication rules, per-brand copy and the front-end avoid-list.
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

# Brand, tone of voice and regional communication

## Brand-agnostic first

This section matters only when several brands share one product; with one brand, keep the same structure and the rule never triggers. When they do, most solutions should work the same way in each, so the rule applies:

> Design brand-agnostic first. A brand-specific requirement is an explicit exception, recorded in the project's PRD under Brand applicability.

**Brand-agnostic means one design and one component API, with per-brand bindings**, not one set of token names. Brands may bind different tokens to the same component. So:

- Use the design system's own component (or its component tokens), resolved per brand at build.
- Never rebuild a component from token names; use the component itself.

## Recording a brand exception

Only when a requirement differs by brand. In the project PRD, under Brand applicability:

| ID | What differs | Brand | Why | Source | Approved by |
|---|---|---|---|---|---|
| BX-01 | … | `<BRAND_1>` only | … | Design node or guideline link | `<decider>, YYYY-MM-DD` |

Cards that touch the exception list its ID in their Brand section. An empty table means the project is fully brand-agnostic.

**Different wording does not need an exception; a different flow, field, rule or content structure does.**

## Tone of voice files

One file per brand in `governance/brand/`, named by the brand's slug: `tone-<brand-1>.md`, `tone-<brand-2>.md` and so on. If a brand has no written tone guide, start with a **draft inferred from the brand's public copy** (for example its website), with the source URL and read date in frontmatter. While a tone file has `status: draft`, all copy written with it is labelled **"tone unverified"**.

### Tone file format

```markdown
# <BRAND_1> tone of voice (draft)

> Draft inferred from public copy. Replace it once the person who approves the brand's voice signs off a written guide.
> Copy written with this draft is labelled "tone unverified".

## Personality
<Two or three adjectives and a one-line brand promise in the brand's own words.>

## What the copy does
| Trait | Evidence from the source | Do | Don't |
|---|---|---|---|

## Error and problem copy
<How the brand behaves when things go wrong.>

## Formatting habits seen
<Capitalisation, punctuation and emoji conventions.>

## Open questions to resolve
- …
```

### Worked example: one invented brand

Brand A is invented for this guide. A second brand gets its own tone file in the same shape.

| Trait | Brand A |
|---|---|
| Language | Everyday words; a technical term is explained the first time it appears |
| Voice | Short sentences that talk to "you" |
| Headings | Say exactly what the page is for |
| Reassurance | Say what happened, then give the next step |
| Claims | Only claims we can back up |
| Problems | Plain and calm when something has gone wrong |

A rule for every brand: **error copy names what happened, what the customer can do now, and when to try again.** Apologising without helping is a defect.

## Regional communication (locale)

Every customer-facing string is written in one declared locale (`<LOCALE>`). Record the locale rules in the core rules and in `md-standard.md`:

| Area | Rule to decide and write down | Example decision |
|---|---|---|
| Spelling variant | Which dictionary | British-family spelling: colour, organise, centre |
| Dates and times | Display format; ISO `YYYY-MM-DD` in files | Customer: per locale, for example 7 January 2030 or January 7, 2030; files: 2030-01-07 |
| Currency | Symbol, position, decimals, tax wording | `<amount> per <period>`, with tax wording as the locale requires |
| Phone and address | Formats and input fields | Split address fields with standard `autocomplete` tokens |
| Required notices | Privacy, terms and similar notices | Record who supplies the wording; never draft it |
| Formality | Address forms, contractions, humour | Contractions allowed; humour as the tone file says |
| Inclusive language | Pronouns, they/them default | Never infer pronouns from a name |
| Accessibility | Page language tag | `lang="<LOCALE>"` on every page |
| Region-specific PII | Identifier formats the privacy guard must catch | National phone formats, national ID and tax numbers, any reference number format your organisation issues |

Internal documents follow the same spelling variant so copy and specs match.

## Per-brand copy in a brand-agnostic build

Customer-facing copy is a **per-brand string set resolved at build**, like tokens: same meaning and structure in every brand, each brand's wording in its own tone, all in `<LOCALE>`.

```text
strings/
├── <brand-1>.<LOCALE>.json   { "signup.workspace.error": "We couldn't find that workspace. Check the name and try again." }
└── <brand-2>.<LOCALE>.json   { "signup.workspace.error": "<the same message, in brand 2's tone>" }
```

The first string is in the invented Brand A's tone above. One key, one meaning; each brand's file holds the wording in its own tone.

## Front-end avoid-list

Generated front ends fall back on a few default styles; a general instruction like "avoid a generic look" just swaps one default for another. So central keeps a **named list of patterns to avoid** (`governance/brand/avoid-list.md`). A pattern on the list is allowed **only if a design-system component or token itself uses it**. After each UI review, any new default pattern the result used is added with date and project, through governance-change.

| # | Pattern | Source |
|---|---|---|
| 1 | Cream or off-white page backgrounds | Anthropic front-end design guidance |
| 2 | Italic accent words inside headlines | Anthropic front-end design guidance |
| 3 | Numbered "01 / 02 / 03" section labels | Anthropic front-end design guidance |
| 4 | Monospace labels or eyebrow text | Anthropic front-end design guidance |
| 5 | Pill-shaped buttons not taken from the library's Button | Anthropic front-end design guidance |
| 6 | Purple or blue-violet gradients, especially on white | Anthropic prompting best practices |
| 7 | Default font stacks (Inter, Roboto, Arial, system-ui) set as a stand-alone `font-family`; a font is allowed only through the design system's typography styles | Anthropic prompting best practices |
| 8 | Three equal feature cards with icon, title and two lines | Common generated default |
| 9 | Glassmorphism, blurred blobs, mesh-gradient heroes | Common generated default |
| 10 | Emoji used as icons instead of the library icon set | Common generated default |
| 11 | Placeholder testimonials, ratings or statistics that are not real | Common default; trust risk |
| 12 | Raw hex, px or rem values where a semantic token exists | Design-system rule |

## Enforcement

| Rule | Enforced by | Strength |
|---|---|---|
| Brand-agnostic first; exceptions recorded | brand-rules; PRD table; brand reviewer check 3; PRD reviewer check 9 | Checked |
| Tone in customer copy; "tone unverified" while draft | tone files; brand reviewer check 5 | Checked |
| Locale spelling | brand reviewer check 6 | Checked |
| Avoid-list | `ui-build`, `figma-sketch`; brand reviewer check 4 proposes additions | Checked |
