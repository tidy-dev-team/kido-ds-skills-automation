---
name: ds-doc
description: >
  Build canvas documentation for a polished component. Produces three Figma frames on a
  📄 Documentation page in the target file — Component Breakdown (variants grid, max/min
  size, icon positions), Mode (one cell per theme.* in tokens.json), and Usage Guidelines
  (When-to-use / When-not-to-use / General / Accessibility / Behavior / Content / Look & Feel).
  Reads prose from specs/{component}.spec.notes.md and Do/Don't visual examples from
  specs/{component}.dodont.json. Workflow B only for v1 — requires working/{project}/tokens.json.
  Triggers on phrases like "document this component", "build doc pages for X",
  "add Figma documentation", or when a designer has finished polish and asks for
  canvas docs (distinct from /ds-storybook which produces code-side stories).
---

# DS Doc

**Polished component + spec + prose → three doc pages on the Figma canvas.**

Used after a component is built (via `/ds-generate` or `/ds-build`) and polished. Sibling to `/ds-storybook` — both run after polish; `/ds-storybook` produces code-side CSF3 stories, `/ds-doc` produces canvas-side doc pages.

For the architectural rationale (three pages vs one, Kido-canonical templates vs per-project, prose-source decisions), see [`docs/adr/0001-ds-doc-three-page-architecture.md`](../docs/adr/0001-ds-doc-three-page-architecture.md).

---

## Workflow Scope

**v1 supports Workflow B only.** The Mode page iterates `tokens.json` `theme.*` entries — Workflow A doesn't produce a `tokens.json`, so the skill fails early if it's missing. Workflow A coverage is a known follow-up.

---

## Prerequisites

Before running, `/ds-doc` expects:

1. **Polished Figma component-set** — the URL or selection of the component to document. Must already be built and polished (pink stubs filled, anatomy correct).
2. **`working/{project}/tokens.json`** — produced by `/ds-extract-design`. Source of the Mode page's grid (iterates every `theme.*` entry).
3. **`specs/{component}.spec.json`** — schema 0.3 or later. Reads `doc.variants_grid` (axis layout), `sizing.{size}.max_*` and `min_*` (size bounds), `anatomy.component_properties` (icon-position detection).
4. **`specs/{component}.spec.notes.md`** — reads the new top sections for Usage Guidelines prose. Missing sections render as pink `NEEDS_CONTENT` stubs (skill doesn't fail).
5. **`specs/{component}.dodont.json`** — optional. Maps rule slugs to Kido DS Figma node IDs for Do/Don't visual examples. Missing entries render as pink `NEEDS_EXAMPLE` stubs per pair.
6. **Three Kido DS master templates** (must exist in the canonical Kido DS Figma file):
   - `doc-template / component-breakdown`
   - `doc-template / mode`
   - `doc-template / usage-guidelines`

If `tokens.json` is missing, the skill exits early: "No `tokens.json` at `working/{project}/`. Run `/ds-extract-design` first."

If a master template is missing in Kido DS, the skill exits with the missing name and a note that docs can't render until the masters are promoted.

---

## Intake

Confirm two inputs before Step 0:

1. **Component name** — read from the request text (e.g., "document the Button"). Used to look up the spec.
2. **Polished Figma component-set URL or selection** — the component to document.

If either is missing, ask the designer. Don't classify the absence as `NEEDS_VALUE` and don't pick a default component.

---

## Step 0 — Read Project-Level Rules

Check the repo root for **`DESIGN-SYSTEM.md`**. If present, read it. Its contents are **authoritative** — they override any default behavior in this skill (slot conventions, status lifecycle, page placement, etc.). If absent, proceed with this skill's defaults.

Silent step — no output unless the file is present and changes behavior the designer should know about.

---

## Step 1 — Resolve Inputs

1. Load `specs/_index.json` and look up the component. If absent, stop and tell the designer: "No Kido spec for `{component}`. Run `/ds-spec-authoring` first."
2. Load `specs/{component}.spec.json`. Verify schema_version ≥ 0.3. If older, warn: "`{component}` spec is schema 0.{N}. `doc.variants_grid` will fall back to a heuristic; max width-height section may be empty."
3. Load `specs/{component}.spec.notes.md`. Parse markdown headings — collect content for the new top sections (When to use / When not to use / General guidelines / Behavior / Content / Look & Feel). Missing sections are recorded for `NEEDS_CONTENT` stubs.
4. Load `specs/{component}.dodont.json` if present. Build a slug → `{ do: nodeId, dont: nodeId }` map. Missing entries are recorded for `NEEDS_EXAMPLE` stubs.
5. Resolve project name (from the working directory the polished component lives in, or by asking).
6. Load `working/{project}/tokens.json`. Build the list of modes from `theme.*` keys. If `tokens.json` is missing or has no `theme.*`, fail early.

---

## Step 2 — Locate the Master Templates

Find these three master frames in the Kido DS file (default: `https://www.figma.com/design/WNOcZbybSn7dAGjTMimIsJ/Kido---DS`, or read from `DESIGN-SYSTEM.md`):

- `doc-template / component-breakdown`
- `doc-template / mode`
- `doc-template / usage-guidelines`

Use `figma_search_components` to locate by exact name. If any master is missing, stop with the name and:

```
Master template missing: doc-template / {page-id}

/ds-doc can't render until all three masters are promoted to Kido DS. Designers are iterating
the templates in the Claroty engagement file — once approved, ask them to move the masters
to the canonical Kido DS Figma file.
```

---

## Step 3 — Find or Create the Documentation Page

In the target file (where the polished component lives):

1. Search for a page named `📄 Documentation`.
2. If absent, create it.
3. The three doc frames go on this page, placed side by side (Component Breakdown left, Mode middle, Usage Guidelines right).

If frames for this component already exist on the page (matched by name — see Step 6), overwrite them in place instead of creating new ones.

---

## Step 4 — Render Component Breakdown

Clone the `doc-template / component-breakdown` master into the Documentation page. Fill its slots:

### Slot reference (master must define these as named layers)

| Slot name | Content | Source |
|---|---|---|
| `slot/title` | Page-type literal "Component Breakdown" | hardcoded |
| `slot/component-name` | Component name (e.g., "Button") | input |
| `slot/status` | "IDEATION" | hardcoded for v1; designer advances manually |
| `slot/variants-grid` | A grid populated per `doc.variants_grid` | spec |
| `slot/max-size` | One row per size showing the component at max bounds | spec |
| `slot/min-size` | One row per size showing the component at min bounds | spec |
| `slot/icon-positions` | Three example cells: No icon, Icon left, Icon right (when applicable) | spec |
| `slot/footer` | Brand strip | hardcoded |

### Variants grid

Read `spec.doc.variants_grid.rows` and `.cols`. For Button: `rows = ["type", "size"]`, `cols = ["state"]` produces a 9 × 6 grid (3 types × 3 sizes × 6 states = 54 cells).

For each cell:
- Compute the variant property values from the row + column indices
- Instantiate the polished component-set with those variant props bound
- Place into the grid cell

Read variant axis values live from the polished component-set in Figma (do not re-derive from spec — polished Figma is truth for actual values).

If the spec is pre-0.3 (no `doc.variants_grid`), fall back: put any `state`-like axis on columns, everything else cross-product on rows. Warn the designer.

### Max / Min size sections

For each `size` in `spec.sizing`:
- If `max_width` or `max_height` is set, render a cell showing the component sized to those bounds.
- Otherwise, omit the row for that size in the Max section.
- For Min, use `min_width` / `min_height` (always present in schema 0.2+).

Header row labels each cell with the size key.

### Icon positions

Detect icon-position support from `spec.anatomy.component_properties`:
- Look for BOOLEAN+INSTANCE_SWAP pairs where the BOOLEAN name matches `Show icon L` / `Show icon R` (or similar — match by `bound_layers` containing `Icon L` / `Icon R`).
- If both pairs exist: render three cells — No icon (both off), Icon left (L on, R off), Icon right (L off, R on).
- If only one icon slot exists: render two cells — No icon, With icon.
- If no icon slots: omit the section entirely.

---

## Step 5 — Render Mode

Clone the `doc-template / mode` master. Fill its slots:

| Slot name | Content | Source |
|---|---|---|
| `slot/title` | "Mode" | hardcoded |
| `slot/component-name` | Component name | input |
| `slot/status` | "IDEATION" | hardcoded |
| `slot/mode-cells` | One cell per `theme.*` entry in tokens.json | tokens.json |
| `slot/footer` | Brand strip | hardcoded |

For each `theme.{mode}` key in `tokens.json`:
- Create a labeled cell (cell label = the mode key, e.g., "Industrial, Light")
- Instantiate the polished component-set's default variant
- Apply the mode via Figma variables (bind the cell's frame to the mode collection if it exists)
- Repeat for each component-set variant the designer wants shown per mode — for v1, just the default variant (idle, primary). If the spec declares a `doc.mode_page_variants` field in a future schema bump, iterate that instead.

Cell grid layout: 2-column. Mode keys with shared prefixes group visually (Industrial Light + Industrial Dark on the same row when possible).

---

## Step 6 — Render Usage Guidelines

Clone the `doc-template / usage-guidelines` master. Fill its slots:

| Slot name | Content | Source |
|---|---|---|
| `slot/title` | "Usage Guidelines" | hardcoded |
| `slot/component-name` | Component name | input |
| `slot/status` | "IDEATION" | hardcoded |
| `slot/when-to-use` | Bulleted list | `.spec.notes.md` `## When to use` |
| `slot/when-not-to-use` | Bulleted list | `.spec.notes.md` `## When not to use` |
| `slot/general-guidelines` | Bulleted list | `.spec.notes.md` `## General guidelines` |
| `slot/accessibility` | Bulleted list | `.spec.notes.md` `## Accessibility` |
| `slot/behavior` | Repeating Do/Don't pairs | `.spec.notes.md` `## Behavior` + `.dodont.json` |
| `slot/content` | Repeating Do/Don't pairs | `.spec.notes.md` `## Content` + `.dodont.json` |
| `slot/look-feel` | Repeating Do/Don't pairs | `.spec.notes.md` `## Look & Feel` + `.dodont.json` |
| `slot/footer` | Brand strip | hardcoded |

### Bulleted slots (When-to-use / When-not-to-use / General / Accessibility)

Each `.spec.notes.md` section under the matching heading contains bullets:

```markdown
## When to use

- Triggering a clear action
- Establishing action hierarchy
- Guiding users through a flow

## When not to use

- Navigation to another page
- When the action is unclear or denoted
- Overloading a page with primary buttons
```

Render each bullet into a row inside the slot, with the appropriate icon (green check for "to-use" / "general", red X for "not-to-use").

If a section is missing, render a single `NEEDS_CONTENT` pink stub in the slot.

### Do/Don't pair slots (Behavior / Content / Look & Feel)

Each section under the matching heading contains rule-keyed sub-sections:

```markdown
## Behavior

### Place icon on the right
**Slug:** `place-icon-right`
{Description of the rule.}

### Keep all buttons in a group the same width
**Slug:** `same-width-in-group`
{Description.}
```

For each rule:
1. Look up the slug in `.dodont.json`:
   ```json
   {
     "place-icon-right": {
       "do":   { "fileKey": "WNOcZbybSn7dAGjTMimIsJ", "nodeId": "98:5001" },
       "dont": { "fileKey": "WNOcZbybSn7dAGjTMimIsJ", "nodeId": "98:5002" }
     }
   }
   ```
2. Clone the master's `pair-template` component into the slot.
3. Set the pair's title to the rule heading, description to the body text.
4. For each side (Do / Don't):
   - If the slug has a `do` (or `dont`) ref: instance-swap the visual slot to that Kido DS component.
   - If absent: render a `NEEDS_EXAMPLE` pink frame in that side.

A section with no rules under it renders a single `NEEDS_CONTENT` stub in the slot.

---

## Step 7 — Position the Three Pages

Place the three frames side by side on the `📄 Documentation` page in this order:

1. Component Breakdown (leftmost)
2. Mode
3. Usage Guidelines (rightmost)

Horizontal gap: 200px. Top-align all three. If frames for this component already existed (Step 3), update them in place — same names, same positions, replace contents.

Frame naming convention (used for re-run lookup):

- `{Component} — Component Breakdown`
- `{Component} — Mode`
- `{Component} — Usage Guidelines`

E.g., `Button — Component Breakdown`.

---

## Step 8 — Report

Summarize to the designer:

```
Documentation rendered for {Component}.

Figma:    {fileKey} → 📄 Documentation page
Pages:    Component Breakdown · Mode · Usage Guidelines
Modes:    {N} cells (from tokens.json theme.*)
Variants: {rows × cols} grid

Stubs:    {N} NEEDS_CONTENT (missing prose in .spec.notes.md)
          {N} NEEDS_EXAMPLE (missing entries in .dodont.json)

Status:   All three pages set to IDEATION. Advance manually as docs mature.
```

If stubs are present, list which sections need attention by name:

```
NEEDS_CONTENT:
  - Usage Guidelines / Behavior (no rules in .spec.notes.md)
  - Usage Guidelines / Look & Feel (no rules in .spec.notes.md)

NEEDS_EXAMPLE:
  - Behavior / place-icon-right (no entry in .dodont.json)
  - Content / max-3-words (no entry in .dodont.json)
```

---

## Re-run Semantics

Re-running `/ds-doc` for a component whose pages already exist:

- Finds existing frames by name (`{Component} — Component Breakdown` etc.)
- Overwrites their contents in place
- Preserves frame position on the canvas (designer may have moved them)
- Preserves the status pill value (designer-edited; don't reset to IDEATION on re-run)
- Replaces variants grid, max/min, icon positions, mode cells, all Usage Guidelines slots

Manual edits inside the frames are lost on re-run unless they're inside designated slots. Tell the designer to put any keep-on-re-run content into a `slot/notes` layer (if the master supports it) or in a separate frame outside the doc page.

---

## Failure Modes

| Condition | Behavior |
|---|---|
| Missing `tokens.json` | Fail early; tell designer to run `/ds-extract-design` |
| Missing master template in Kido DS | Fail early; name the missing master |
| Missing spec (pre-0.3) | Warn; fall back to heuristic for `doc.variants_grid` |
| Missing `.spec.notes.md` section | Render `NEEDS_CONTENT` pink stub in that slot |
| Missing `.dodont.json` entry | Render `NEEDS_EXAMPLE` pink stub in that pair side |
| Polished component variants don't match spec axes | Use polished Figma values; flag mismatch in the report |

---

## Status Pill Lifecycle

Three stages, advanced manually by the designer:

1. **IDEATION** — default. `/ds-doc` always renders pages with this status on first run.
2. **READY** — polished, reviewed, awaiting final approval.
3. **PUBLISHED** — approved and shared with the client.

The skill never advances the status. The lifecycle is a convention for designers to track which docs are shippable.

---

## Critical Rules

**Templates are visual sources of truth.** Don't draw doc layouts procedurally — clone the masters. If the master's layout looks wrong, update the master in Kido DS; don't override in code.

**Never invent prose or examples.** Missing `.spec.notes.md` sections → stubs. Missing `.dodont.json` entries → stubs. The agent's job is to surface gaps, not to fabricate content.

**Polished Figma is truth for variant values.** The variants grid renders the polished component-set's actual variants, not the spec's declared variants. If they diverge, that's a polish gap to flag in the report — but render what's polished.

**Status pill stays where the designer left it.** Re-runs preserve the current status (don't reset to IDEATION).

**Workflow A is out of scope for v1.** If `tokens.json` is missing, exit. No "render with one default cell" fallback — that misrepresents the project's mode coverage.

**Doc pages are per-component, not per-project.** Each component gets its own three frames on the shared `📄 Documentation` page. Naming convention encodes the component; re-runs find by name.
