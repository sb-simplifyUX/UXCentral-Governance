---
title: UX design process
description: The author's six-step UX design process as operated by the governance system, with each step's job, activities, outputs, exit criteria, where AI helps and where it is kept out, the human gate and enforcement.
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
source: https://sebastianbourges.com/process/
---

# UX design process

The process is Sebastian Bourges' own, published at [How I solve a problem](https://sebastianbourges.com/process/).

**First find the right problem. Then find the right solution.** Six steps, in order: Understand, Define, Sketch, Decide, Validate, Deliver. Each has one job. For each step this chapter gives its job, where AI helps and where AI is kept out, and how the governance system runs and checks it.

## Two diamonds: the right problem, then the right solution

Each diamond is the same move: open up wide, then narrow down.

- **Diamond one (Understand → Define) finds the right problem:** explore everything that is going on, then narrow to the one problem worth solving.
- **Diamond two (Sketch → Decide) finds the right solution:** come up with as many ideas as the team can stand, then pick one to build.
- **Validate and Deliver wrap the second diamond:** test it, then ship it.

```text
   Diamond one: the right problem          Diamond two: the right solution
  ┌───────────────┬───────────────┐      ┌───────────────┬───────────────┐
  │  UNDERSTAND   │    DEFINE     │  →   │    SKETCH     │    DECIDE     │  →  VALIDATE  →  DELIVER AND AFTER
  │  open up      │  narrow to    │      │  open up      │  narrow to    │
  │               │  one sentence │      │  many ideas   │  one winner   │
  └───────────────┴───────────────┘      └───────────────┴───────────────┘
```

Why the shape matters: do not jump to answers while you should still be asking, and stop inventing new ideas once the question is agreed. Most projects break when those two blur together.

## Principles

- **Keep asking and answering separate.** Open up before narrowing down, in both diamonds.
- **One person decides. Everyone informs the decision.** One accountable decider, agreed at the start.
- **Evidence-based arguments.** Every argument traces back to the Define sentence.
- **AI is an intern, not an oracle.** AI speeds up the volume work (reading, sorting, first drafts); judgement, accountability and one person reading another stay with people.
- Ground decisions in evidence; validate before committing; human judgement overrides AI suggestions.

## The Define sentence

The single most important artefact. Every later decision points back to it.

> "[customer] needs a way to [need], because [what we learned]."

| Status | Meaning | Who sets it |
|---|---|---|
| `not drafted` | Nothing yet | Nobody: the starting status |
| `draft` | Being written | AI or team |
| `candidate` | Offered to the decider | AI may offer 3 to 5, each citing evidence |
| `agreed` | The problem the project solves | **The decider only**, with date |

**Define cannot be skipped.** Sketch, Decide, Validate and Deliver all require the sentence at `agreed`.

## The steps

### 1. Understand

**Stop solving. Start looking.**

| | |
|---|---|
| Job | Understand what customers are really trying to get done, and where they get stuck, before anyone discusses solutions |
| Activities | Stakeholder discovery (outcomes, metrics, constraints, risks, assumptions); customer research using jobs to be done (what customers are trying to achieve, their context, workarounds and frustrations); reading the numbers and the conversations together (analytics, interviews, operations, domain expertise); competitor review |
| Inputs | Brief, stakeholder objectives, business goals, analytics, feedback, interviews, support tickets, competitor analysis, existing journeys |
| Outputs | A clear picture of what customers are really trying to do; where they get stuck, and the workarounds they have built; patterns seen only when the numbers and the conversations are read together; how the competition handles the same problem; a summary of everything already known. In files: research summary, assumption list, JTBD statements, risks and unknowns |
| Exit criteria | Can clearly answer: Who experiences the problem? What are they trying to get done? What evidence supports this? Why does it matter? The research summary is written |
| AI helps with | Reading huge piles of research faster than a person could alone, and spotting what repeats across them. In this system it may also surface assumptions and draft up to three *emerging* problem statements (not Define candidates) |
| AI kept out of | The conversation with the customer. A model cannot read a room or know when to push and when to wait |
| Human gate | Customer conversations stay human. Shortcut to refuse: "We already know our customers." |

### 2. Define

**Boil it down to one problem, in one sentence.**

| | |
|---|---|
| Job | Turn the evidence into one problem worth solving, written as one sentence the whole team can stand behind |
| Activities | Affinity mapping (grouping notes); journey analysis (pain points, decision moments, friction, breakdowns); customer types based on what customers are trying to do, not who they are on paper; point-of-view (POV) sentences; How Might We (HMW) questions. Done with the team, not for them |
| Outputs | The Define sentence (the customer, the need and what we learned); a short list of customer types based on what they are trying to do; a map of the customer's path from first contact to the moment that matters; sharper questions for the next step. In files: affinity clusters, key insights, candidate Define sentences with evidence, HMW questions |
| Exit criteria | The customer, the need and what we learned fit in the single sentence; it cites evidence from Understand; the team can say it without looking; the decider has agreed it |
| AI helps with | Grouping notes across mountains of interviews, and drafting candidate sentences for people to argue with (3 to 5, each citing evidence); challenging weak reasoning |
| AI kept out of | Writing the customer types. A model averages everyone into bland, generic descriptions |
| Human gate | People write the customer types (personas). The decider chooses and agrees the sentence |

### 3. Sketch

**Lots of ideas first. Judge them later.**

| | |
|---|---|
| Job | Open the second diamond: come up with as many different ideas as the team can stand, well past the comfortable first ones, without judging them |
| Activities | Silent, timed individual sketching (it quiets the loudest and most senior voices; the decider stays out of the sketching session); Crazy 8s; brainwriting; comic-strip storyboards (situation, trigger, interaction, outcome); concept exploration (alternative flows, mental models, changes to the product or the process); low-fidelity screens |
| Outputs | A wall of different ideas, deliberately more than feels comfortable (the system asks for at least 16 across different mental models); a few of them drawn as short comic strips, for ideas the decider picks; rough sketches of the screens an idea would need; simple diagrams of how a customer moves through it |
| Exit criteria | Multiple genuinely different concepts exist; the team has not converged early |
| AI helps with | Throwing in raw ideas, especially when the team keeps circling the same one. One more voice, not the conclusion |
| AI kept out of | Deciding which idea is good. A model has no taste, no stakes and no history with the customers |
| Human gate | No ranking, scoring or "best" language. Picking ideas to draw as comic strips is not ranking or dropping |

### 4. Decide

**One person decides. Everyone informs the decision.**

| | |
|---|---|
| Job | Pick one direction to build, on evidence. One person makes the final call, agreed at the start; everyone else informs it |
| Activities | Review the options against the Define sentence; silent, simultaneous voting; weigh each option on two questions: how much could it help, and how much will it cost to build; risk assessment (customer, business, technical) |
| Outputs | The team's options, silently ranked; a shortlist of the two or three worth committing to, picked by the decider and weighed by likely benefit and likely cost; one named person who signs off; a written record of why the winner won and why the others did not (the PRD decision record) |
| Exit criteria | One clear direction; reasoning recorded and traced to the Define sentence; outstanding assumptions known |
| AI helps with | Stress-testing the chosen idea against cases the team might have missed: "how does this fail for a customer in situation X?" In this system it may also draft likely benefit and cost per option for the decider to challenge |
| AI kept out of | The decision itself. A model has nothing at stake and answers to no one. The person who signs off has both |
| Human gate | The decision and the reasoning belong to the decider |

### 5. Validate

**Test it with real people before it gets expensive.**

| | |
|---|---|
| Job | Find what the team got wrong while it is still cheap to fix. The point is not to pass a test: the prototype is a question, not a product |
| Activities | Define research questions and the riskiest assumptions; choose fidelity (rough early: paper or quick click-throughs; polished later: watched as people think out loud, often alongside a larger group running it on their own for comparable numbers); synthesis |
| Facilitation rules | Never ask "wasn't that easy?"; never defend the design when someone struggles; every stumble is information, not the customer's fault |
| Outputs | Fast, cheap evidence from the rough early version; evidence from the polished version, watched with real people and measured at larger scale; a short report ranking what to fix before launch by what it is costing; a list of the assumptions that were wrong (the most valuable output). In files: test plans, discussion guides, findings |
| Exit criteria | Evidence that customers understand the solution, key tasks are achievable and major risks are addressed, or design changes have been identified |
| AI helps with | Drafting the test plan, and pulling the recurring patterns out of a batch of sessions |
| AI kept out of | Running the session. A real person reading another real person is not work a model can do |
| Human gate | People run the sessions |

### 6. Deliver and after

**Launch isn't the finish line. It's the next set of data.**

| | |
|---|---|
| Job | Two jobs in one step: hand the work to the engineers well, then watch what happens after launch and feed it into the next Understand |
| Activities | Handover as an ongoing conversation, not a moment: stay close through the build and settle the small calls between the mockup and the real thing; a reusable kit (colours, spacing and parts already agreed); specs for every state; design QA; post-launch measurement (where customers drop off, what gets used, what they say; adoption, task success, satisfaction, business outcomes); learning loop |
| Outputs | A package the engineers can build from without guessing; every screen specified, including the awkward in-between states; a way to measure whether what shipped is doing its job; recordings of real customers (personal information: they stay in the research folders); an open channel for feedback after launch |
| Every state | default, focus, hover, pressed, disabled, empty, loading, error, success, partial, each with its accessible name and announcement |
| Exit criteria | Launched; success measures tracked against baseline; learnings documented; new questions logged for the next Understand |
| AI helps with | Drafting the documentation for the reusable kit, and spotting patterns in the post-launch numbers |
| AI kept out of | The conversation with the engineers. It is about what is actually possible and what the business needs first. A model can prepare for it; it cannot have it |
| Human gate | The engineering conversation is for people |

## AI boundaries, summarised

AI is an intern, not an oracle. Every "kept out of" line above says the same thing: AI stays out wherever the work needs judgement, accountability, or one person reading another.

| AI may | AI must not |
|---|---|
| Summarise and cluster research | Replace customer conversations |
| Draft candidate problem statements for the decider to challenge | Write customer types (personas) |
| Add raw ideas | Judge which idea is good |
| Stress-test a chosen idea | Make the decision |
| Draft test plans | Run test sessions |
| Spot patterns in sessions or post-launch data | Replace conversations with engineers |
| Draft design-kit documentation | Invent evidence or treat its own output as truth |

When a step needs a forbidden activity, the AI **prepares the materials** (interview guide, shortlist matrix, test plan, spec), **names the gate and who acts**, and **stops**.

## AI behaviour in every step

Always: state the current step; stay within its job; challenge assumptions; reference evidence; make reasoning visible; separate facts from assumptions; highlight risks; recommend next steps.

Never: skip research; jump to solutions in Understand; converge in Sketch; make final decisions; invent evidence; treat AI output as truth.

## Skipping steps

If asked for work from a later step, the AI names every skipped step and what it would normally produce, and asks the decider. On approval it records `skipped (<decider>, <date>)` in the PRD step table so the gap stays visible. Define is never skippable.

## How the system enforces the process

| Rule | Enforced by |
|---|---|
| State the current step | Session-start hook prints `process_step`; `process-gate` skill `status` |
| Steps in order; skipped steps named and recorded | `process-gate` skill `request`; `process-gate-reviewer` check 2 |
| Exit criteria evidenced by files | `process-gate` skill `check`; phase `_index.md` checklists; reviewer check 3 |
| Define sentence format and status | PRD template; reviewer check 4 |
| Sketch only after Define is agreed; no ranking | `figma-sketch` skill gate; reviewer check 6 |
| Decide has one decider, 2 or 3 shortlisted options, recorded reasoning | PRD decision record; reviewer check 7 |
| Deliver covers every state and success measures | `phases/06-deliver/_index.md`; accessibility reviewer; reviewer check 8 |
| AI boundaries | Core rules; `process-gate` human gates; reviewer check 9 |
| Prototypes only after Decide | `ui-build` skill refuses before an agreed Define and a recorded winner |

The session-start hook only prints the step; it blocks nothing. Step order is kept by the skill and checked by the reviewer, not stopped mechanically.
