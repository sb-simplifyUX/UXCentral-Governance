---
title: Adapting to a new business
description: Discovery questionnaire and replacement checklist for adapting the governance system to a new organisation, industry, region, brand set, design tool or team size.
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

# Adapting to a new business

## What stays the same everywhere

The design process, the AI boundaries, the Define sentence, the kanban method, roles and approvals, the reviewer pattern, the privacy rules, the Markdown standard, WCAG 2.2 AA, the release-age and exact-pin rules, non-destructive retrofit, and versioning. These are the method; do not weaken them per business.

## What changes per business

| Area | Values to set | Where |
|---|---|---|
| Organisation | `<ORG>`, marketplace and plugin descriptions | README, marketplace.json, plugin.json |
| People | `<ADMIN>`; each project's decider at creation; each user at install | central PRD; project PRDs; installer |
| Brands | Number of brands, slugs for the `brand` field, brand-agnostic or single-brand | core rules (Brands), md-standard allowed values, brand-rules, tone files, PRD template |
| Industry | Business funnel; accessibility journeys in scope; industry examples in falsifiers and checks | `standard-processes/business-funnel.md`, `accessibility.md` |
| Region and language | `<LOCALE>`, spelling variant, `lang` tag, date and currency formats, required notices | core rules, md-standard, tone files, accessibility (3.1.1) |
| Region-specific personal information | Phone formats, national ID and tax number formats, any reference number format your organisation issues, postcode formats | `hooks/lib/pii.mjs`, privacy.md, privacy reviewer patterns |
| Allowed email domains for the PII guard | `example.com` plus the organisation's own domain, `<ORG_EMAIL_DOMAIN>`: `const ALLOWED_EMAIL_DOMAINS = ['example.com']; // add your organisation's domain, <ORG_EMAIL_DOMAIN>` | `hooks/lib/pii.mjs` |
| Design tool | Tool, MCP server, file and library key, how the library names its primitive, semantic and component tokens | brand-rules, design-library map, ask permissions, brand reviewer tools |
| Stack | Framework (Astro by default), package manager, toolchain versions | stack.md, code-stack reviewer |
| Folders | `<ROOT>`, folder names | core rules "Where things live"; hooks `CONFIG`; the installer, including the central Edit rule it fills from `{{CENTRAL_POSIX}}`; the plugin folder name (`<PLUGIN>`) in the path the md library (`tools/lib/md.mjs`) imports `pii.mjs` from; the project template's folder names (`phases/`, `research/`, `prototypes/`) in its `.gitignore`, its `.claude/settings.json` and `new-project`'s `PHASE_FOLDERS`; presets |
| Git host | Private remote for central; who can push | README, PRD, enforcement (administrator becomes Hard) |
| Models | Orchestration and implementation models and effort | user settings, agent frontmatter |

## Discovery questionnaire

Ask the administrator in one batch; write `TBC` where unknown.

**Organisation and people**

1. Organisation name (or a neutral label if this must stay anonymous)?
2. Administrator: full name and how to address them?
3. Who else will run the system, and who usually decides on projects?

**Brands and market**

4. How many brands? Names and slugs. Do they share one product (brand-agnostic first) or run separate ones?
5. Links to each brand's public About page and any written tone guide.

**Industry and customers**

6. Industry, customer types (for example self-serve and sales-led), main products or services.
7. The main customer journeys online (for accessibility scope and the funnel).
8. Who in the business can review the funnel draft (sales, operations, support)?

**Region**

9. Language and region for customer copy (`<LOCALE>`). Spelling variant?
10. Formats for dates, currency, phone numbers, addresses. Required notices (privacy, terms), and who supplies their wording?
11. Identifiers that count as personal information here (any your organisation issues, and others your field collects, such as dates of birth or national ID numbers)?

**Design tool**

12. Which tool holds the design system, and which file is its published library?
13. How does the library name its primitive, semantic and component tokens?
14. Is there an MCP server for the tool? If not, which alternative ([09](09-Design-Tool-Connection.md#alternatives-to-figma))?

**Engineering**

15. Front-end framework and package manager (default Astro and pnpm)?
16. Existing component or token packages to treat as "own libraries"?
17. Git host for central and projects; who may push?

**Machines**

18. Operating systems in use; editor; where the work root folder should be?

## Region-specific PII patterns: how to write them

Add patterns to `PII_PATTERNS` in `hooks/lib/pii.mjs`. Rules:

- Use lookarounds so a pattern does not match inside longer keys or hex strings: `(?<![\w+])…(?!\w)`.
- Allow common separators (space, hyphen) in any grouping, and the country prefix.
- Test against real-looking fake values and against long hex keys (no false matches).
- Keep the names short (`phone_international`, `phone_national`, `account_ref`, or a `phone_mobile` you add) because they appear in block messages.
- Mirror the patterns in the privacy reviewer's grep list.

Example shapes (replace every placeholder with your region's values):

```js
// <CC>: country calling code; <MOBILE_PREFIX>: the digits a mobile number starts with;
// <ID_PREFIX>: the fixed prefix of an organisation identifier; <N>: its digit count.
phone_mobile: /(?<![\w+.])(?:\+<CC>|0)[\s-]?<MOBILE_PREFIX>(?:[\s-]?\d){7,9}(?![\w.])/g, // then 7 to 9 digits in any grouping; adjust to your region
account_ref:  /(?<![\w-])<ID_PREFIX>\d{<N>}(?!\w)/g,
```

`<CC>` is the country calling code, `<MOBILE_PREFIX>` the digits a mobile number starts with after the leading 0 or the country code, `<ID_PREFIX>` the fixed prefix of an organisation identifier and `<N>` its digit count (the blueprint's own `account_ref` is `ACC-` followed by 8 digits). A pattern with a placeholder left in it still loads but never matches, so replace them all and run the hook tests before relying on it.

## Smaller teams and single brands

- **One person:** the user, decider and administrator are the same person. Keep the roles; approvals are still recorded.
- **One brand:** set `brand` allowed values to the single slug plus `n/a` if needed; the exception table stays empty; brand-agnostic rules never trigger.
- **No design tool:** use tokens-in-code as the source ([09](09-Design-Tool-Connection.md#alternatives-to-figma)); the brand reviewer checks code and token files.
- **No front-end work:** skip `ui-build`, libraries and the code-stack reviewer; keep everything else.

## Larger organisations

- Put central on a remote that only the administrator can push to; this turns "only the administrator changes central" from guided into hard.
- Turn the plugins on in the organisation's claude.ai account so they sync into every member's Claude Code (they load as `<name>@synced`; `claude plugin list --json` shows them).
- Add CI that runs `check-md`, `claude plugin validate` and the tool tests (`node --test 'tools/test/*.test.mjs'`) on every central pull request.
- Consider a dedicated board app, like the one in [a kanban case study](https://sebastianbourges.com/case-studies/simplifyux-kanban/), when several AI sessions share work at once.

## Final adaptation checklist

- [ ] No invented example from this guide remains (search for Sam Example, Brand A and the `ACC-` identifier).
- [ ] All placeholders replaced or marked "not used".
- [ ] Locale rules written and applied to tone files and accessibility.
- [ ] PII patterns cover the region; tests prove no false matches on keys.
- [ ] The design system's library listed with its keys read from the tool.
- [ ] Funnel drafted and reviewed by the business.
- [ ] Tone files drafted with source URLs, `status: draft`.
- [ ] Enforcement matrix complete; known limits written.
