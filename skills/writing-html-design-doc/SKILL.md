---
name: writing-html-design-doc
description: Use after brainstorming spec is approved, before writing-plans, when a visual browser-openable design artifact is needed for human review
---

# Writing HTML Design Doc

## Overview

Generate a self-contained HTML design document with inline SVG diagrams from the approved spec. The document gives human reviewers a visual, browser-openable artifact to understand and approve before implementation begins.

**Announce at start:** "I'm using the `writing-html-design-doc` skill to create the visual design document."

**Save to:** `docs/superpowers/designs/YYYY-MM-DD-<feature-name>-design.html`

## When NOT to Use

- Trivial changes (config updates, single-line fixes) with no meaningful architecture to diagram
- Projects with no components to visualize (single-function scripts, pure data files)

## Process

```dot
digraph html_design_doc {
    "Read approved spec" [shape=box];
    "Determine optional diagrams" [shape=box];
    "Generate HTML with SVGs" [shape=box];
    "Save and report path" [shape=box];
    "Human reviews HTML?" [shape=diamond];
    "Update HTML" [shape=box];
    "Invoke writing-plans" [shape=doublecircle];

    "Synthesize content" [shape=box];
    "Read approved spec" -> "Synthesize content";
    "Synthesize content" -> "Determine optional diagrams";
    "Determine optional diagrams" -> "Generate HTML with SVGs";
    "Generate HTML with SVGs" -> "Save and report path";
    "Save and report path" -> "Human reviews HTML?";
    "Human reviews HTML?" -> "Invoke writing-plans" [label="approved"];
    "Human reviews HTML?" -> "Update HTML" [label="changes requested"];
    "Update HTML" -> "Human reviews HTML?";
}
```

1. Read the approved spec from `docs/superpowers/specs/` (the file written by `brainstorming`). If no spec file exists, derive content from the brainstorming conversation context instead.
2. Synthesize from spec + conversation context: goal, components, data structures, interactions, key decisions
3. Determine which conditional diagrams apply (see Conditional Sections table below)
4. Generate the HTML file with all applicable sections and SVGs
5. Save and tell the user: *"Design document written to `<path>`. Please open it in a browser and review it."*
6. Wait for explicit human approval. If changes requested, update the HTML (targeted update to changed sections only) and re-present
7. Once approved, invoke `superpowers:writing-plans`

## HTML Structure

Single self-contained `.html` file — no CDN, no external assets. Opens offline in any browser.

### Always-Present Sections

| Section | Content |
|---|---|
| Header | Feature name, date, "Awaiting Review" status badge |
| Overview | Goal (1–2 sentences), tech stack table, key constraints |
| Architecture | Inline SVG — components as labeled boxes, dependency arrows. Always generated. |
| Key Decisions | Table: decision, chosen option, alternatives considered, reason |
| Components | 1–3 sentence description per major component |

### Conditional Sections

| Diagram | Include when | If omitted, add note |
|---|---|---|
| Data Flow | System moves or transforms data between components | *"No distinct data transformation layer — data flow omitted."* |
| Sequence | Meaningful multi-step interactions exist (user actions, API calls, async flows) | *"No complex interaction sequences — sequence diagram omitted."* |
| Data Model | System stores or structures persistent data | *"No persistent data layer — data model omitted."* |

## SVG Diagram Guidelines

Write all SVGs using primitives only: `rect`, `text`, `line`, `path`, `marker`. No external libraries.

**Color palette:**
- Component boxes: `fill="#dbeafe" stroke="#3b82f6"` (blue)
- Relationship lines/arrows: `stroke="#6b7280"` (grey)
- Labels: `fill="#111827"` (near-black)
- SVG background: `fill="#f9fafb"` (light grey)

**Minimum SVG size:** `width="800"`, scale height to content. Each diagram has a visible `<p class="caption">` below it.

**Architecture:** Left-to-right or top-to-bottom. Each component is a `rect` (min 120×50px) with centered `text`. Arrows use `line` with arrowhead `marker`. Include `<title>` element for accessibility.

**Data Flow:** Left-to-right. Inputs on left, outputs on right, transforms in center. Use `rx="8"` on transform rects.

**Sequence:** Vertical dashed lifelines, horizontal message arrows with labels above. Actor/component names at top of each lifeline.

**Data Model:** Entity `rect` with a filled header bar and field rows below. Relationship lines connect entities with cardinality labels (1, *, 1..*).

## HTML Template

Start from `template.html` in this skill directory. Fill in all bracketed placeholders with real content from the spec and conversation.

## Review Gate

Do **not** invoke `writing-plans` until the human explicitly approves the HTML. Partial approval ("update the sequence diagram") triggers a targeted update to that section only, not a full regeneration. Re-present after each update.
