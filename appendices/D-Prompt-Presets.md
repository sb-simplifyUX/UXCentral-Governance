---
title: "Appendix D: prompt presets"
description: Five copy-and-paste prompts, namely three project presets for creating a new project, retrofitting an existing one and drafting a PRD from research, plus one to set up the whole governance system and one to adopt it on another machine.
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

# Appendix D: prompt presets

This appendix holds five prompts. The first three are the project presets (new project, retrofit, PRD from research): store them in central `prompts/project-presets.md`. The last two set up the system and adopt it on another machine; they live only here.

Copy the block, fill the parts in `<angle brackets>`, paste it as your whole message. Write `TBC` if unknown; the AI asks for what it needs.

The three project presets start with a slash command on line one. Keep it there: those jobs run only when a person starts them.

## Before you start

- For the setup prompt (4) and the adopt prompt (5) you need only Visual Studio Code with the Claude Code extension and a Claude account on a plan that includes Claude Code. Claude installs everything else and asks before each install.
- For the project presets, central governance is installed on your machine (the installer asks your name and how to address you). Restart Visual Studio Code afterwards. In a session opened in a project folder, Claude then has a line starting "UX governance:" in its session context; ask Claude to quote that line to check.
- Every project has one **decider**. It can be you or someone else; if someone else, the AI prepares the work and stops for their approval at each step.
- Open the session in the right folder: `<PROJECTS>` for a new project; the project's own folder for retrofit and PRD from research.
- Use the most capable model for the session; the AI hands mechanical work to cheaper models itself.
- **Never type or paste customer details into these prompts.** Point to the file instead and refer to people as P01, P02…

## 1. New project

```text
/<PLUGIN>:new-project <Project name>

Decider (full name): <who approves everything in this project>
How to address the decider: <name, and pronouns if they want>
Summary (one line): <what this project is about>
Background: <what prompted it: data, feedback, business goal>
Brand: <all brands | one brand: which>
Known brand exception: <none, or what differs and why>
Objectives: <what we want to achieve>
Goals: <measurable or observable aims>
In scope: <what is included>
Not in scope: <what is excluded>
Hard stops: <what must not happen>
Outputs: <what the project will produce>
Definition of done: <how we know it is finished>
Success measures: <measure, baseline, target, source; or TBC>
Research I already have: <kind and count, for example "6 interview transcripts, 1 support-ticket export"; no file names that include a person's name>
```

What happens: the AI asks for anything missing (and stops for the decider if that is not you), creates the folder, drafts README and PRD, runs the Markdown check and three reviewers, reports what is TBC, and stops. The project starts at Understand with no Define sentence and no release.

## 2. Retrofit an existing project

```text
/<PLUGIN>:retrofit-project <Project name>

Decider (full name): <name, or "as in the PRD">
How to address the decider: <address form, or "as in the PRD">
Brand: <all brands | one brand: which>
Where I think the project is: <Understand | Define | Sketch | Decide | Validate | Deliver | not sure>
Agreed problem statement, if any: <the Define sentence, or "none yet">
Agreed by (decider) and date: <name and YYYY-MM-DD, or "not yet">
Do not touch: <files or folders to leave exactly as they are, or "nothing">
Notes: <renamed files, duplicates, which files hold customer details (file names only, no details)>
```

What happens: dry run into `<WORK>/reports/…`, reviewers check it; the decider approves all, some or none; apply backs up to `<WORK>/backups/…` and changes only what was approved; check and report. The project cannot be set past Define without an agreed sentence. The retrofit tools are built as their own card on central's board the first time a project needs them ([16 Phase 4](../16-Implementation-Guide.md#phase-4-tools)).

## 3. PRD from research

```text
/<PLUGIN>:prd-from-research <Project name>

Research to use: <"all research folders", or specific files or folders>
Do not use: <files to leave out, or "nothing">
Already decided: <scope, constraints or goals fixed whatever the research says, or "nothing">
Questions I want answered: <what you hope the research tells you, or "none">
Notes: <for example, which files hold customer details (file names only)>
```

What happens: the AI lists the research, reads it with customer details kept out, prepares a summary with themes and up to three emerging problem statements, drafts PRD sections marked [research], [inferred] or TBC, waits for the decider, then writes only what was approved. No personas, no solutions, no Define sentence, no step change.

## 4. Set up the governance system (new machine or new business)

Open Visual Studio Code in the folder that will hold `<ROOT>` (your home folder, in the reference layout), open the Claude Code panel and paste:

```text
Set up the UX central governance system on this machine.
The blueprint's files are in <folder>, which I copied there myself from the guide's repository.
Read README.md, then follow 16-Implementation-Guide.md phase by phase.

Resume at: <Phase 0, or the phase to continue from after a restart>
Organisation: <name or neutral label>
Administrator (full name / how to address them): <…> / <…>
Brands (display names and slugs): <…>
Industry and customer types: <…>
Locale for customer copy: <LOCALE, for example en-US>
Work root folder, inside your home folder: <for example ~/ux-work>
Design tool and design system: <tool; design system library name>
Git host for central: <your private git host and repository, or "local only for now">
Operating system: <macOS | Linux>

Rules: run the Phase 0 questionnaire for anything missing; present the full plan for my approval before creating
anything; ask before installing, downloading, deleting, pushing or editing ~/.claude files; write files only with
your file tools and read them back; run each phase's Check before moving on; delegate mechanical implementation
to Sonnet.
```

The blueprint's files are in the guide's public repository, https://github.com/sb-simplifyUX/UXCentral-Governance. Copy the repository into a new, empty folder yourself and give the assistant that folder. Getting the files is your step, not the assistant's: the governance settings refuse `curl` and `wget`.

## 5. Adopt on another machine (central already exists)

```text
Install the existing UX central governance on this machine. The blueprint's files are in <folder>, copied there
as for the setup prompt. Central is at <git URL on your private git host>; clone it
to <CENTRAL>. Follow 16-Implementation-Guide.md: check the prerequisites in Phase 1 and install what is missing,
asking before each install; run the installer with --name "<full name>" --address "<address form>" (Phase 7);
then install the frontend-design and figma plugins and let me authorise Figma (Phase 8, step 1). Finish with the
Phase 7 Check and the Phase 8 Check.
```

## If something goes wrong

| You see | Do this |
|---|---|
| Claude cannot quote a "UX governance:" line from its session context | Wrong folder, or not installed; reopen in the right folder or run the installer |
| "already exists" when creating | Use the retrofit prompt, or choose another name |
| "has no PRD.md" when drafting from research | Retrofit the project first |
| Slash command not recognised | `/reload-plugins`, or restart Visual Studio Code |
| The AI starts design work after setup | Stop it. Setup, retrofit and PRD drafting never include ideas, personas or decisions |
