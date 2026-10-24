---
title: Design tool connection
description: How the AI assistant connects to the design system in Figma through MCP, the allowed-library and never-invent rules, token tiers, the design-library map and its fortnightly check, write protection, and alternatives to Figma including a design tool built for an AI agent.
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
source: https://sebastianbourges.com/case-studies/simplifyux-designer/
---

# Design tool connection

## Principle

The design system in the design tool is the **only** source of tokens and components. The AI reads it; it never invents from it, never writes to the library file, and never treats a stale map or an example page as proof.

## Figma through MCP (as built here)

| Item | Setting |
|---|---|
| Connection | Figma's MCP server, supplied by the `figma` plugin in Anthropic's official marketplace, `claude-plugins-official`, which Claude Code adds by default. Install it from the plugin panel (`/plugins` in the Visual Studio Code extension, `/plugin` in a terminal) or with `claude plugin install figma@claude-plugins-official`. See [11](11-Anthropic-Claude-Code-Standards.md#plugins-from-anthropics-official-marketplace) |
| Tool names | A plugin-supplied MCP server's tools are named `mcp__plugin_<plugin>_<server>__<tool>`, so Figma tools are `mcp__plugin_figma_figma__<tool>`. The bare names below are the tool part only; confirm every name against the tools listed in your session before writing a permission rule or a tool list |
| Session | Call the server's session tool (confirm its name in the live tool list) once with the library's file key `<FILE_KEY>` before any other file tool, and reuse the session it returns |
| Search | Every `search_design_system` call is **pinned** with `includeLibraryKeys` set to the allowed library key `<LIB_KEY>` |
| Read tools | `search_design_system`, `get_metadata`, `get_screenshot`, `get_variable_defs`, `get_design_context`, `get_figjam`, `get_libraries` |
| Write tools | `use_figma`, `generate_figma_design`, `send_code_to_figma`, `create_new_file`, `upload_assets`, Code Connect writes, `generate_diagram`, image and shader tools: all are **ask** permissions |
| Companion skills | Supplied by the same `figma` plugin. Load before calling: `figma-use` before `use_figma`; `figma-design-to-code` before `get_design_context`; `figma-generate-design` for screens; `figma-generate-diagram` before `generate_diagram`; `figma-use-figjam` for FigJam |
| Front-end guidance | The `frontend-design` plugin from the same marketplace, which the `ui-build` skill loads (`claude plugin install frontend-design@claude-plugins-official`). There is no separate design plugin |

### Allowed library

`governance/brand/brand-rules.md` holds the only allowed library:

| Library | File key | Library key |
|---|---|---|
| `<DS_LIB>` | `<FILE_KEY>` | `<LIB_KEY>` |

**No other library is allowed.** A token or component found only outside `<DS_LIB>` is treated as missing. Keys are read from the tool (`get_libraries`, the file URL), never guessed.

### Token tiers

This guide describes design tokens in three tiers. Design systems name them in their own way, so record in the design-library map how your library names each tier.

| Tier | Holds | Use |
|---|---|---|
| Primitive | Raw values: colours, dimensions, fonts | **Never bind directly** |
| Semantic | Roles: colour roles, spacing, radius, border, typography, elevation, layout | Bind in designs and code |
| Component | Values that belong to one component | Bind where a component defines them |

Bind the semantic and component tiers, never the primitive tier. Typefaces come only through the typography text styles. Each brand's values are resolved at build, so brand-agnostic code uses one token name for every brand.

### Never invent

> Never invent a token or component. If one is missing, say so and mark any substitute as a **proposal**.

In code and summaries a substitute is labelled `PROPOSAL: <what> (missing from the design-system library)`. In a sketch it becomes a plain placeholder with the same label. A component needed by more than one project gets a card on central's board to build it in `<LIB_SCOPE>/ui`.

### Example pages are references, not sources

A library file often includes example pages (sample screens) that use tokens and styles not in the library. Anything taken from them through `get_variable_defs` or `get_design_context` must be confirmed with a pinned search; if it is not found, it is missing.

## The design-library map

Exploring a design file costs many tool calls, so central keeps a **map**: `governance/brand/design-library-structure.md`. It is a guide, not a source. Before using any name in it, confirm with a pinned search; if the search disagrees, the search wins and the map is out of date.

### What the map records

- File layout: component pages and example pages.
- Published asset naming (icons and logos).
- Variable collections with their modes and counts (for example colour, spacing, radius, typography, motion, breakpoints).
- Styles: text styles, effect styles, grid styles; whether colour is variables-only.
- How a colour resolves: from the role a component binds, through the active modes (such as light and dark), to the reference value.
- Responsive breakpoints: viewport width, grid columns, width, margin and gutter per breakpoint.
- Component inventory per page with node ids and variant counts; open questions.

### Freshness frontmatter

```yaml
figma_verified: 2030-01-12            # last checked against the library file
figma_last_published: 2030-01-09      # newest publish date seen in the library
figma_check_every_days: 14
figma_next_check: 2030-01-26
```

The session-start hook prints a reminder once `figma_next_check` has passed. The `figma-library-check` skill re-verifies: components published since the last date, variables always, pages; reports a diff to the administrator; updates the map; resets the dates. **A map refresh is data**: approved and committed with the message `design-library map: verified YYYY-MM-DD`, but no version bump. Any `use_figma` call in that skill is getters only, and each script is shown to the administrator before it runs.

### Design-library map skeleton

Copy this into `governance/brand/design-library-structure.md` and fill it from the tool, with keys read from the tool and never guessed. Every table after Library starts empty; a list with nothing in it reads `none`.

```markdown
---
title: Design-library map
description: Guide to the allowed design-system library's pages, components, variables, modes and breakpoints; confirm every name with a pinned search before use.
doc_type: governance
project: central
process_step: n/a
brand: all
status: draft
owner: Central administrator
created: YYYY-MM-DD
updated: YYYY-MM-DD
governance_version: 1.0.0
contains_pii: false
figma_verified: YYYY-MM-DD
figma_last_published: YYYY-MM-DD
figma_check_every_days: 14
figma_next_check: YYYY-MM-DD
---

# Design-library map

A guide, not a source. Confirm every name with a pinned search before using it; if the search disagrees, the search wins and this map is out of date.

## Library

| Library | File key | Library key |
|---|---|---|
| <DS_LIB> | <FILE_KEY> | <LIB_KEY> |

## Pages

| Page | Kind (components or example) | Notes |
|---|---|---|

## Components

| Page | Component | Node id | Variants |
|---|---|---|---|

## Variable collections

| Collection | Tier | Modes | Variables |
|---|---|---|---|

## Modes

| Mode | Collections | How a colour resolves (component role to mode to reference value) |
|---|---|---|

## Breakpoints

| Breakpoint | Viewport width | Columns | Width | Margin | Gutter |
|---|---|---|---|---|---|

## Styles and published assets

- Text styles: none
- Effect styles: none
- Grid styles: none
- Asset naming (icons, logos, shapes): none

## Open questions

- none
```

## Write protection

- The library file is a read-only source. Sketches go into a separate Sketch file (Design for screens, FigJam for storyboards and flows) that the decider names or approves creating.
- Every design-tool write tool is an `ask` permission.
- The brand reviewer agent has only read tools for Figma in its tool list.

## Front-end handoff

`ui-build` reads sources in order: own libraries (`<LIB_SCOPE>/tokens`, `<LIB_SCOPE>/ui`), then the design tool through MCP, then nothing. Token values are exported into `<LIB_SCOPE>/tokens` as CSS custom properties generated from the variables, never hand-typed. The falsifier for that export: "A generated token value differs from its design-tool variable value."

## Alternatives to Figma

The rules above are tool-independent: one allowed library, pinned lookups, never invent, a read-only library, a map with a freshness check. What changes is the connector.

| Option | How the AI connects | Notes |
|---|---|---|
| Figma (reference) | Figma's MCP server, supplied by the `figma` plugin from `claude-plugins-official` | Richest read and write tools; design-to-code and code-to-design skills |
| Another design tool with an MCP server | Add its MCP server; map its read tools to "search, metadata, variables, screenshot" and its write tools to `ask` permissions | Check the server's current tool list before writing permissions |
| Open-source design tool | Its API or plugin system, wrapped in an MCP server | Self-hostable; designs stay in-house |
| Tokens in code as the source of truth | Design tokens (for example W3C design-tokens JSON) in `<LIB_SCOPE>/tokens`, components in `<LIB_SCOPE>/ui` with Storybook or Astro pages | No design-tool connector needed; the brand reviewer checks code instead of the tool |
| **A design tool built for an AI agent** | A local desktop design app built for an agent to drive directly | See [below](#a-design-tool-built-for-an-ai-agent) |

### A design tool built for an AI agent

The author's case study of [a desktop design tool built for an AI agent to drive](https://sebastianbourges.com/case-studies/simplifyux-designer/) describes the alternative approach. Design tools are built for a person with a mouse; an agent using one imitates that person by taking screenshots and guessing where to click, which is slow, brittle and leaves no clear record. A tool built for the agent lets it work directly:

- The agent makes the same changes a person makes, so every edit can be seen and undone.
- It runs on your own machine, so designs stay there (helpful for privacy rules).
- Its tools let the agent read, change, check and export a design.
- Accessibility checks run while the design is made, not at the end, and each one says how to fix what it finds.
- **Done is measured, not claimed:** the agent compares the tool's render with a reference and keeps fixing what differs until the comparison passes.

To plug such a tool into this governance: expose it as an MCP server, put its write tools on the `ask` list, give the brand reviewer its read tools, and let the measurement score serve as a card's Evidence line.
