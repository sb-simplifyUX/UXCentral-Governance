---
title: UX central governance blueprint
description: What you need, the credit, the reading order and the placeholders for the business-agnostic blueprint that lets a person or an AI rebuild the UX central governance system for any organisation, on macOS or Linux.
doc_type: readme
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

# UX central governance blueprint

This folder describes a complete, working system for running UX design work with an AI coding assistant (built in Claude Code) under one central set of rules. It covers the order to build it in, the rules it enforces, the design process at its heart, and a step-by-step guide to rebuild it for any organisation.

## What you need

You need two things: Visual Studio Code with the Claude Code extension, and a Claude account on a plan that includes Claude Code. Claude installs everything else for you in [Phase 1](16-Implementation-Guide.md#phase-1-prerequisites) and asks before each install: the Claude Code command-line tool (the extension keeps its own copy for its chat panel, but the setup runs `claude` commands in a terminal), Node.js (the hooks, installer and checker are Node.js scripts), Git (central is a Git repository), and pnpm (only if you build your own libraries in Phase 9). If an install needs your computer password, Claude shows you the exact command to run.

To start, open a folder in Visual Studio Code, open the Claude Code panel and paste the setup prompt from [Appendix D §4](appendices/D-Prompt-Presets.md#4-set-up-the-governance-system-new-machine-or-new-business). Claude then follows the [implementation guide](16-Implementation-Guide.md) phase by phase and stops for your approval where the guide says so.

## Credit

UX Central Governance is a method designed and owned by Sebastian Bourges. He created its six-step design process, its rule that AI is an intern and not an oracle, and its kanban method with falsifiers and doors, and he uses the method to build and run his own software.

Licence: CC BY 4.0. The terms are in `LICENCE.txt`, beside this file.

- The design process: [How I solve a problem](https://sebastianbourges.com/process/), the author's process page.
- Software built and run under the method, each written up as a case study:
  - [A task board that a person and several AI sessions share](https://sebastianbourges.com/case-studies/simplifyux-kanban/), which runs on the kanban method.
  - [A desktop design tool built for an AI agent to drive](https://sebastianbourges.com/case-studies/simplifyux-designer/).
  - [One console to create, watch, change and roll back websites](https://sebastianbourges.com/case-studies/simplifyux-console/).
  - [A notes app that keeps every note as a plain file](https://sebastianbourges.com/case-studies/simplifyux-notes/).

Every name, brand, organisation, place and example in this guide is invented, except the method's author and the author's published case studies, above. The author is the only real person named. The tools the guide uses (Claude Code, Visual Studio Code, Node.js, Git, pnpm, Astro and Figma) are named because you install, connect or build with them. Your own values go where the [placeholders](#placeholders) stand.

## Who this is for

| Reader | Start with |
|---|---|
| A person who wants the system set up on their machine | [What you need](#what-you-need), then the setup prompt in [Appendix D §4](appendices/D-Prompt-Presets.md#4-set-up-the-governance-system-new-machine-or-new-business) |
| An AI assistant whose user asked, in their own message, to set this up on this machine | [16 Implementation guide](16-Implementation-Guide.md), then follow its links |
| An administrator deciding whether to adopt it | [01 Overview and architecture](01-Overview-and-Architecture.md), [02 Build order and lessons](02-Build-Order-and-Lessons.md) |
| A designer or decider working on a project | [04 UX design process](04-UX-Design-Process.md), [05 Kanban method](05-Kanban-Method.md), [06 PRD and project template](06-PRD-and-Project-Template.md) |
| Someone porting it to another AI tool | [18 Porting to other AI tools](18-Porting-To-Other-AI-Tools.md) |

## Reading order

| # | Document | What it covers |
|---|---|---|
| 01 | [Overview and architecture](01-Overview-and-Architecture.md) | What the system is, its layers, folder layout, how projects inherit rules |
| 02 | [Build order and lessons](02-Build-Order-and-Lessons.md) | The build order version by version, the working method, decisions and lessons |
| 03 | [Roles and authorisation](03-Roles-and-Authorisation.md) | User, decider, administrator; who approves what; how approvals are recorded |
| 04 | [UX design process](04-UX-Design-Process.md) | The six steps, exit criteria, AI boundaries, the Define sentence |
| 05 | [Kanban method](05-Kanban-Method.md) | Releases, outcomes, nine-section cards, the FALSIFIER, five doors |
| 06 | [PRD and project template](06-PRD-and-Project-Template.md) | Required PRD sections, project folder structure, phase index files |
| 07 | [Business funnel](07-Business-Funnel.md) | How to build the industry-specific customer lifecycle funnel, with an invented worked example |
| 08 | [Brand, tone of voice and regional communication](08-Brand-Tone-and-Regional-Communication.md) | Multi-brand rules, tone files, locale rules, per-brand copy |
| 09 | [Design tool connection](09-Design-Tool-Connection.md) | Connecting to Figma through MCP, library rules, the library map, and alternatives |
| 10 | [Privacy, accessibility, stack and Markdown standards](10-Privacy-Accessibility-Stack-Markdown.md) | Research data handling, WCAG 2.2 AA, stack rules, frontmatter standard |
| 11 | [Anthropic and Claude Code standards](11-Anthropic-Claude-Code-Standards.md) | The Claude Code features used and how (CLAUDE.md, settings, permissions, plugins, models), and the official references |
| 12 | [Skills catalogue](12-Skills-Catalogue.md) | All eleven skills: purpose, trigger, steps, gates |
| 13 | [Agents and reviewers](13-Agents-and-Reviewers.md) | Six read-only reviewers, one mechanical helper, the review protocol |
| 14 | [Hooks, permissions and enforcement](14-Hooks-Permissions-Enforcement.md) | The four hooks, when to refuse and when to ask, the permission lists, the rule-to-enforcement matrix |
| 15 | [Versioning and change management](15-Versioning-and-Change.md) | Semantic versions, changelog, retrofit, governance changes |
| 16 | [Implementation guide](16-Implementation-Guide.md) | The ordered build plan for macOS and Linux, with checks after each phase |
| 17 | [Adapting to a new business](17-Adapting-To-A-New-Business.md) | Discovery questionnaire and every value to replace |
| 18 | [Porting to other AI tools](18-Porting-To-Other-AI-Tools.md) | Mapping each layer to tools without plugins, hooks or subagents |
| A | [Appendix A: file templates](appendices/A-File-Templates.md) | Copy-ready core rules, user CLAUDE.md, settings, central and project files, plugin manifests |
| B | [Appendix B: skill and agent sources](appendices/B-Skill-and-Agent-Sources.md) | Full text of every skill and agent, generalised |
| C | [Appendix C: hook and tool sources](appendices/C-Hook-and-Tool-Sources.md) | Hook scripts, installer, Markdown checker, new-project tool, retrofit tool specification |
| D | [Appendix D: prompt presets](appendices/D-Prompt-Presets.md) | Five copy-and-paste prompts: three project presets (new project, retrofit, PRD from research), set up the system, adopt on another machine |
| E | [Appendix E: glossary](appendices/E-Glossary.md) | Every term used in the system |

## Placeholders

Replace these when you implement. [17 Adapting to a new business](17-Adapting-To-A-New-Business.md) explains how to choose each value.

| Placeholder | Meaning | Example value |
|---|---|---|
| `<ORG>` | The organisation | Example Co |
| `<BRAND_1>`, `<BRAND_2>`… | The brand's display name; add one per extra brand if you run more than one | Brand A |
| `<brand-1>`, `<brand-2>`… | Each brand's slug: lowercase, used in file names and in the `brand` field (which holds `all` or one slug) | `brand-a` |
| `<INDUSTRY>` | Business field | Subscription software for teams |
| `<LOCALE>` | Language and region for customer copy | `en-GB`, `en-US`, `es-ES` |
| `<Project name>` | A project's name, which is also its folder name | `Team onboarding` |
| `<ROOT>` | Parent work folder; it must be a folder inside your home folder | `~/ux-work` |
| `<CENTRAL>` | Central governance repository | `<ROOT>/000-central` |
| `<PROJECTS>` | Folder holding all projects | `<ROOT>/001-projects` |
| `<WORK>` | Local-only reports and backups | `<ROOT>/_governance-work` |
| `<PLUGIN>` | Plugin name | `ux-governance` |
| `<MARKETPLACE>` | Local plugin marketplace name | `ux-central` |
| `<LIB_SCOPE>` | npm scope for own libraries | `@ux-central` |
| `<ADMIN>` | Administrator's full name | Sam Example |
| `<ORG_EMAIL_DOMAIN>` | The organisation's own email domain, which the PII guard allows | `example.org` |
| `<OPUS_MODEL_ID>` | The full Opus model ID that `/model` or `/status` shows; the key under `modelSettings` and the value in `Agent(model:<OPUS_MODEL_ID>)` | `claude-opus-5-5` |
| `<LTS>` | The major version of the current Node.js LTS release | Read when you install, in Phase 1 |
| `<PNPM_VERSION>` | An exact pnpm version, never a range | Read when you install, in Phase 1 or 9 |
| `<DS_LIB>` | The design-system library | `<Your Design System>` |
| `<FILE_KEY>`, `<LIB_KEY>` | Design tool file and library keys | Read from the tool in Phase 8, never guessed |
| `{{...}}` tokens, such as `{{USER_NAME}}`, `{{CENTRAL_POSIX}}`, `{{PROJECT_NAME}}` | Filled by the installer, the new-project tool, or the AI when it drafts a new project's README and PRD; leave them in the templates | See [Appendix A](appendices/A-File-Templates.md) |

## Conventions in these documents

- "I" in quoted rules is the AI assistant speaking, because the core rules file is written to the assistant.
- **Hard**, **Checked** and **Guided** describe how strongly a rule is enforced (see [14](14-Hooks-Permissions-Enforcement.md)).
- Paths use forward slashes and `~` for the home folder. Commands are for bash or zsh on macOS or Linux; where the two systems differ, both are shown.
- Every file in this guide follows the guide's own Markdown standard ([10 §4](10-Privacy-Accessibility-Stack-Markdown.md#4-markdown-standard)): standard frontmatter and one H1 that matches the title.
- The `governance_version: 1.4.0` in each file's frontmatter is the version of this blueprint. Your own install starts its history at **1.0.0**: every copy-ready `VERSION`, manifest, first changelog entry and PRD template in this guide says `1.0.0`.
- Version numbers of third-party tools are examples; always pick current versions using the release-age rule in [10](10-Privacy-Accessibility-Stack-Markdown.md).
