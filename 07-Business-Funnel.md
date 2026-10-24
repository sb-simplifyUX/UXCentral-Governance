---
title: Business funnel
description: How to build and use an industry-specific customer lifecycle funnel as a central reference, with the stage template, discovery questions, KPI framework and a fictional worked example for a subscription software provider.
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

# Business funnel

## Why central holds a funnel

Design work goes wrong when it treats one screen as the whole story. A **customer lifecycle funnel** written for the business field gives every project the same map of where customers come from, what they go through and where value grows. Central stores it at `standard-processes/business-funnel.md`. It is reference material, not a rule.

It is used to:

| Use | Where |
|---|---|
| Locate a project's problem in the lifecycle | Understand: scoping, research plan |
| Write realistic situations for storyboards | Sketch: `figma-sketch` uses funnel moments for the "situation" panel; customers are described by situation, never by invented persona |
| Generate discovery and interview questions | Understand: interview guides |
| Pick success measures that connect to business value | PRD Success measures; Deliver |
| Find validation checks and common failure points | Validate test scenarios; Deliver states (error, partial) |

## Recommended shape: one lifecycle, several pathways

Use a **single lifecycle funnel** with pathways per customer type (for example self-serve and sales-led), rather than one funnel per pathway. The aim is to see the whole path a customer takes, from first hearing of the product to recommending it.

```text
Awareness
    ↓
Sign-up or enquiry
    ↓
Qualification and discovery
    ↓
Proposal and purchase
    ↓
Account setup
    ↓
Onboarding
    ↓
Adoption
    ↓
Support
    ↓
Expansion
    ↓
Renewal and retention
    ↓
Advocacy
```

Rename, merge or drop stages to fit the business (for example, drop Qualification and discovery where customers buy without talking to anyone).

## Stage template

For each stage and each customer pathway, write:

| Field | Question it answers |
|---|---|
| Triggers | What makes a customer enter this stage? |
| Products or services in play | What is offered or used here? |
| Activities | What the business and the customer do |
| Flow | The steps as a short arrow diagram |
| Validation checks | How we know the stage succeeded for this customer |
| Common issues | Where it typically fails (feeds error and partial states) |
| Information captured | What data is collected (flag anything that is personal information) |
| KPIs | How the stage is measured |
| Value potential | Relative value: low, medium or high |

## Discovery: understand first

The qualification stage's objective is to understand how the customer works today, find pain points and uncover opportunities. Write discovery questions per topic (for example onboarding, integrations, reporting, security). These become the seed of interview guides.

## KPI framework

Chain the stage measures so drop-off is visible:

```text
Sign-ups and enquiries → Qualified opportunities → Purchases → Accounts set up
→ Successfully onboarded → Teams in weekly use → Support cases resolved → Seats or add-ons added
→ Renewals → Referrals
```

## Growth model

```text
Acquire customer → Smooth setup → Regular use → Expand use
→ Retain → Generate referrals
```

Measure progress by what customers **get done**, not by which features they touch. The goal is to grow use of the product over time.

## Worked example (fictional): subscription software provider

A fictional provider of a subscription software product for teams, with a self-serve pathway for individuals and small teams and a sales-led pathway for larger organisations. Plan and product names are generic.

### Awareness

| | Self-serve | Sales-led |
|---|---|---|
| Triggers | Team outgrows spreadsheets; frustration with a current tool; recommendation from a peer; free-plan limit reached; content found through search | New initiative or budget; tool consolidation; current vendor contract expiring; team growth; compliance or security requirement |
| Products | Free plan; Starter plan; Team plan; extra seats; integrations | Enterprise plan; single sign-on; advanced reporting; data migration; premium support; training |
| Relative value | Starter low; Team medium; Team with add-ons high | Enterprise high; Enterprise with services high |

### Sign-up or enquiry

Self-serve sources: website sign-ups, free trials, in-product referrals, app marketplace listings, content and webinars, review sites. Sales-led sources: demo requests, partner referrals, outbound prospecting, events, expansion from self-serve accounts. Information captured: name, work email, company, role, team size, current tools, budget timing. **In research exports this is personal information: it stays in the project's research folders (`phases/*/research/`).**

### Qualification and discovery

| Topic | Self-serve questions | Sales-led questions |
|---|---|---|
| Onboarding | Who will set it up? How many people need access in the first week? | Rollout plan? Pilot team? Admin roles? |
| Integrations | Which tools must it connect to: calendar, email, file storage? | Identity provider? Data warehouse? Custom APIs? |
| Reporting | What do you need to see each week? | Which dashboards do leaders need? Export requirements? |
| Data and migration | Anything to import from a spreadsheet? | Data to import? Volume? Retention rules? |
| Security | Two-step sign-in needed? | Single sign-on? Audit logs? Security questionnaire? |

### Proposal and purchase

Self-serve example: Team plan for 8 users + calendar integration (relative value: medium). Sales-led example: Enterprise plan for 200 users + single sign-on + data migration + premium support (relative value: high). Purchase activities: plan selection, seat count, payment details, terms acceptance (self-serve); security review, proposal, procurement, contract signature, rollout planning (sales-led). Outputs: subscription created → payment confirmed → account setup starts.

### Account setup

Often the stage where new customers first get stuck.

| Pathway | Flow | Validation checks | Common issues |
|---|---|---|---|
| Self-serve account | Sign-up → email verified → workspace created → teammates invited → first project created | Sign-in works, invitations delivered, first task completed | Verification email not received, invitations sent to the wrong address, empty workspace with no guidance |
| Integration | Integration chosen → permissions granted → first sync → data visible | Sync completes, data matches the source, errors reported clearly | Permission refused, partial sync, duplicate records |
| Enterprise rollout | Kick-off → single sign-on configured → pilot team → feedback → company-wide rollout | Users sign in through the identity provider, admins trained, pilot feedback addressed | Identity provider misconfigured, rollout dates slip, pilot feedback not acted on |
| Data migration | Export from the current tool → field mapping → test import → review → final import | Record counts match, history preserved, owners assigned | Mapping errors, missing attachments, import timeouts |

### Onboarding

Self-serve: welcome message → guided setup → first success moment → teammate invitations → billing setup (account working, teammates active, first project complete, billing confirmed). Sales-led: account setup → admin training → user training → success plan → technical review (sign-in, integrations, permissions, migrated data reviewed).

### Adoption

Self-serve: first-week check-in → weekly usage review → feature adoption review → satisfaction survey. KPIs: weekly active users, time to first value, features in regular use, satisfaction. Sales-led: adoption review per team → admin check-in → usage report for the account owner → plan for teams not yet active. KPIs: share of seats active each week, adoption across teams, satisfaction, open support cases.

### Support

Self-serve: issue reported → help centre and in-app guidance → chat support → escalation → resolution. Issue types: sign-in and access; integrations not syncing; billing and plan questions; missing features; slow pages. Self-service functions: reset password, manage seats, change plan, update payment, export data, check the status page.

Sales-led: ticket logged → priority set → investigation → escalation → resolution → follow-up review. Support desk: tickets, response-time targets, named account contact, incident updates.

### Expansion, retention and advocacy

| | Self-serve | Sales-led |
|---|---|---|
| Expansion path | Free → Starter → Team → more seats → add-ons (relative value: low to high) | Pilot team → department → company-wide → advanced add-ons → services (relative value: medium to high) |
| Retention triggers | Renewal date, low usage, price change, competitor offers | Contract renewal, budget reviews, leadership change, security re-review |
| Retention activities | Usage tips, plan right-sizing, annual-plan offers, feature announcements | Quarterly business reviews → adoption report → roadmap session → renewal proposal |
| Advocacy | Reviews, referral credits, community and social recommendations | Case studies, references, testimonials, partner referrals |

## Prompt to draft a funnel for a new business

```text
Draft standard-processes/business-funnel.md for <ORG>, a <INDUSTRY> business serving <CUSTOMER_TYPES>,
with customer copy in <LOCALE>. Use the stage template in the governance blueprint (07 Business funnel):
one lifecycle with a pathway per customer type; for each stage give triggers, products, activities,
flow, validation checks, common issues, information captured (flag personal information), KPIs and
relative value (low, medium or high). Mark every statement [source] with its source or
[inferred]. Do not invent statistics. This is a draft for the administrator to challenge.
```

Replace the placeholders before pasting: `<ORG>` is the organisation's name or a neutral label; `<INDUSTRY>` its field; `<CUSTOMER_TYPES>` the customer pathways it serves (for example self-serve and sales-led, or individuals, small teams and large organisations); `<LOCALE>` the language and region of customer copy, as set in [08](08-Brand-Tone-and-Regional-Communication.md#regional-communication-locale).

The administrator reviews the draft with people who run sales, operations and support before it is approved through the governance-change procedure.
