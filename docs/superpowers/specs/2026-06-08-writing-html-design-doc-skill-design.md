# Writing HTML Design Doc: Visual Design Artifact Before Implementation

**Date:** 2026-06-08
**Status:** Approved
**Scope:** `skills/writing-html-design-doc/` (new), `skills/brainstorming/SKILL.md` (one-line edit)

## Problem

After brainstorming produces a spec, the transition to implementation loses the visual understanding built up in conversation. The spec is markdown — useful for the AI, but hard for a human reviewer to quickly grasp architecture, data flow, and component relationships. There is no artifact that a non-technical stakeholder or a developer new to the project can open and immediately understand.

The result: implementation starts before humans have meaningfully reviewed the design, which defeats the purpose of the approval gate at the end of brainstorming.

## Goal

Introduce a `writing-html-design-doc` skill that sits between `brainstorming` and `writing-plans`. It generates a self-contained HTML file — visual, diagram-rich, browser-openable — that a human reviewer approves before any implementation planning begins.

## Workflow Slot

```
brainstorming → [spec.md approved] → writing-html-design-doc → [HTML approved] → writing-plans → execution
```

`brainstorming/SKILL.md` step 9 currently reads "invoke writing-plans skill." This single line changes to "invoke `writing-html-design-doc` skill (which transitions to writing-plans after HTML approval)."

## Design

### Skill Identity

- **Name:** `writing-html-design-doc`
- **File:** `skills/writing-html-design-doc/SKILL.md`
- **Description trigger:** "Use after brainstorming spec is approved, before writing-plans — generates self-contained HTML design document with inline SVG diagrams for human review"

### Skill Behavior (ordered steps)

1. Announce: "I'm using the `writing-html-design-doc` skill to create the visual design document."
2. Read the approved spec from `docs/superpowers/specs/` (the file written by `brainstorming`).
3. Synthesize content from spec + brainstorming conversation context: goal, components, data structures, interaction sequences, key decisions.
4. Determine which optional diagrams apply (see Diagram Rules below).
5. Generate the HTML file with all applicable sections and inline SVG diagrams.
6. Save to `docs/superpowers/designs/YYYY-MM-DD-<feature>-design.html`.
7. Tell the user: *"Design document written to `<path>`. Please open it in a browser and review it."*
8. Wait for explicit human approval. If changes requested, update the HTML and re-present.
9. Once approved, invoke `superpowers:writing-plans`.

### HTML Output Structure

The output is a single self-contained `.html` file — no CDN, no external assets, no JavaScript frameworks. Opens offline in any browser.

| Section | Always / Conditional | Content |
|---|---|---|
| Header | Always | Feature name, date, static status badge ("Awaiting Review") |
| Overview | Always | Goal (1–2 sentences), tech stack table, key constraints |
| Architecture | Always | Inline SVG — components as labeled boxes, arrows for dependencies |
| Data Flow | Conditional | Inline SVG — inputs/outputs/transforms as left-to-right flow |
| Sequence | Conditional | Inline SVG — lifelines and message arrows for the main interaction path |
| Data Model | Conditional | Inline SVG — entities with fields, relationship lines |
| Key Decisions | Always | Table of trade-offs from brainstorming (what was chosen and why) |
| Components | Always | Short prose description of each major component |

### Diagram Rules

**Architecture is always generated.** It is the minimum viable design artifact.

The remaining three diagrams are conditional:

| Diagram | Include when |
|---|---|
| Data Flow | System moves or transforms data between components |
| Sequence | There are meaningful multi-step interactions (user actions, API calls, async flows) |
| Data Model | System stores or structures persistent data |

When a conditional diagram is omitted, the HTML includes a one-line note stating why (e.g., *"No persistent data layer — data model omitted."*) so the reviewer knows it was a deliberate choice.

### SVG Generation Approach

Diagrams are written as hand-authored inline SVG using primitive elements: `rect`, `text`, `line`, `path`, `marker`. No external SVG libraries. Diagrams are schematic — clarity over pixel-perfection. Each diagram has a visible caption below it. Color palette: neutral background, blue for components, grey for relationships, black for labels.

### File Naming

`docs/superpowers/designs/YYYY-MM-DD-<feature-name>-design.html`

The `docs/superpowers/designs/` directory is created if it does not exist.

### Review Gate

The skill does **not** invoke `writing-plans` until the human explicitly approves the HTML. This is the primary purpose of the skill — enforcing human review before implementation begins. Partial approval ("looks good except the sequence diagram") triggers a targeted update and re-presentation, not a full regeneration.

## What Is Not In Scope

- Interactive HTML (no JavaScript interactions, hover states, or collapsible sections)
- Automatic diagram updates when spec changes post-approval
- PDF export
- Diagram generation for sub-components not mentioned in the spec

## Files Changed

| Action | Path |
|---|---|
| Create | `skills/writing-html-design-doc/SKILL.md` |
| Edit (1 line) | `skills/brainstorming/SKILL.md` — step 9: invoke `writing-html-design-doc` instead of `writing-plans` |
| Create (dir) | `docs/superpowers/designs/` |
