# `ds-doc` — design research

**Status:** ⚠️ **Superseded.** This document captured the pre-implementation research and a single-page slot-template recommendation. The shipped design is the **three-page architecture** documented in [`docs/adr/0001-ds-doc-three-page-architecture.md`](../docs/adr/0001-ds-doc-three-page-architecture.md), with the skill itself at [`skills/ds-doc.md`](../skills/ds-doc.md). Kept here for historical context — do not act on this file's recommendations.
**Date:** 2026-05-12 (research) → superseded the same day by ADR 0001 after designers delivered the three-page layout.

---

## Origin

A new skill to build component documentation pages on the Figma canvas (the eighth `ds-*` skill). Prompted by review of the legacy Tidy Guide Builder plugin at:

`/Users/dmitridmitriev/Documents/the plugin factory/⬇️ not relevant/🪦 All Tidy Guides and Docs plugins/Tidy Guide Builder v1`

---

## What the old plugin did

Figma plugin (Create Figma Plugin + Preact UI). Built doc pages procedurally from auto-layout primitives. Section types it supported:

| Section | What it built |
|---|---|
| Header | Title + canonical example instance |
| Anatomy | Numbered tags pointing to layers, with a legend (custom tag component set) |
| Spacing | Size markers + spacing markers showing padding/gap measurements per size |
| Properties | Per-size strip + boolean prop on/off pairs |
| Variants | Grid of all variant combinations with axis labels |
| Text / Two-columns / List / Link / Image | Generic prose blocks (incl. do/don't pairs) |

Persistence: `figma.root.getSharedPluginData("guide_lite")`.
Layout description: JSON-tree template (`src/templates/templateDefault.ts`) interpreted by `src/figma_functions/documentationBuilder.ts`.
UI: per-component section toggles + saved-data list.

Notable files:
- `src/figma_functions/documentationBuilder.ts` — top-level orchestrator
- `src/figma_doc_sections/build{Anatomy,Spacing,Prop,Var}Section.ts` — per-section builders
- `src/figma_functions/Anatomy/` — custom numbered-tag anatomy callouts
- `src/figma_functions/AnatomySpacing/` — spacing markers
- `src/templates/templateDefault.ts` — JSON tree describing layout

### What's worth keeping

- The **section taxonomy**: header → anatomy → spacing → properties → variants → prose.
- The idea of **size-aware spacing/property strips** (one row per size, not one row per variant).

### What to throw out

- Procedural auto-layout from scratch — fragile, visually drifts from designer intent.
- Custom annotation tag components — Figma now ships native Annotations API.
- The JSON layout-tree template — over-engineered, hides intent.
- `setSharedPluginData` persistence — re-running the skill from spec + notes is the right model.
- Section-toggle UI — agent skill works better with a sensible default set; designer deletes what they don't want.

---

## Conventional wisdom (as of 2026)

Where mainstream DS teams have moved since the old plugin was written:

1. **Figma is not the docs.** Storybook + MDX (or Zeroheight) is canonical. The Figma doc page is a *poster* for designers — usage, do/don't, anatomy, content guidance — not a prop reference. Dev Mode + Code Connect cover engineering.
2. **Component descriptions and Annotations are native now.** The plugin's custom anatomy tags are obsolete — Figma's Annotations API does the numbered-callout job and surfaces in Dev Mode automatically.
3. **Variables + modes resolve token values automatically.** Token tables cite token *names*, not hardcoded hexes. Figma resolves per mode.
4. **Variant galleries auto-derive from the component set.** Read the variant matrix and lay it out.
5. **Prose is markdown-first, rendered to Figma.** Do/Don't, usage rules, a11y notes live in version-controlled markdown next to the spec. Tidy already has this instinct (`specs/*.spec.notes.md`).
6. **Slot/template components beat procedural layout.** Modern DS ships a "Component doc page" master with named slots. Skills clone and fill slots. Visual polish stays in the master.

---

## Recommended shape for `ds-doc`

Slot-template based, spec-driven. Fits next to `ds-storybook`: same trigger point (after polish), same orchestration style.

```
       polished Figma component
                │
        ┌───────┴────────┐
        ▼                ▼
    ds-storybook       ds-doc
    (code docs)        (canvas docs)
        │                │
   stories PR     doc page in Figma
                   + component description set
                   + optional working/.../docs/{component}.md
```

**Inputs:** component name + Figma component-set URL (or selection) + optional `specs/{component}.spec.notes.md` for prose.

### Pipeline (per component)

1. **Read project rules** — `DESIGN-SYSTEM.md` at repo root if present (same Step 0 every skill does).
2. **Resolve source of truth.** Workflow A → `specs/{component}.spec.json`. Workflow B → `working/{project}/library-snapshot.json` + polished Figma set. Spec drives content; polished Figma drives visual values.
3. **Clone the doc-page master** from Kido's DS Figma file. Slots have semantic names: `slot/header`, `slot/anatomy`, `slot/sizes`, `slot/variants`, `slot/tokens`, `slot/dos-donts`, `slot/a11y`, `slot/changelog`.
4. **Fill mechanical sections from spec** (no prose required):
   - Header: name, default-variant instance, one-line description (from `spec.notes.md` H1)
   - Anatomy: instantiate the polished default; use Figma's native Annotations API on layers named in `spec.anatomy.layers`
   - Sizes: side-by-side instances per `spec.sizing` key
   - Variants: read the component set's variants live (don't re-derive from spec — polished Figma is truth) and lay out by axis
   - Tokens used: 2-column table token name → currently-resolved value via `figma_get_variables`
5. **Fill prose sections from `{component}.spec.notes.md`** — parse markdown headings (`## Do`, `## Don't`, `## Usage`, `## Accessibility`, `## Content`). Missing sections become `NEEDS_CONTENT` stubs (pink fill, matches existing `NEEDS_VALUE` convention).
6. **Set Figma's native component description** on the component set — surfaces in Dev Mode for free.
7. **Place** the doc frame on a `📄 Documentation` page in the same file. Create page if absent.
8. **Optional emit** `working/{project}/docs/{component}.md` — the prose source, editable outside Figma.

**Outputs:** doc page frame + updated component description + (optional) prose markdown file.

---

## Key design decisions

| Decision | Why |
|---|---|
| Slot components, not procedural builders | Designers own visual polish in Figma, not the agent. The old plugin's biggest failure mode was visual drift between manual and auto-built sections. |
| Spec is structure-of-truth; polished Figma is value-of-truth | We already have specs — use them. Old plugin re-derived everything from the live component each run. |
| Native Annotations API for anatomy callouts | Replaces ~6 files of custom tag-layout code. Ships in Dev Mode automatically. |
| Markdown-first prose | Version-controlled, reviewable, AI-editable. Figma renders, doesn't edit. |
| `NEEDS_CONTENT` stubs match `NEEDS_VALUE` pattern | Consistent with the rest of the system — visible, pink, deliberate. |
| Runs after polish, order vs `ds-push` irrelevant | Doc references resolved token values; needs polish to be final. Doc lives in Figma, PR lives in code — no ordering constraint. |
| One shared "Doc Page Template" master in Kido DS file | One-time DS-team prerequisite. Then every project benefits. |

---

## What to discard from the old plugin

- `setSharedPluginData` persistence — spec + notes file is the persistence.
- Section-toggle UI — sensible defaults; designer deletes what's unwanted.
- Custom marker components for spacing — use Figma's built-in measurement/redline or a single shared spacing-marker component in the DS file.
- Inter-everywhere defaults — match `tokens.json` typography tokens (same rule as `ds-build`).

---

## Open questions (resume here)

1. **Does Kido DS already have a doc-page master in Figma?** Designers are producing the layout now. When delivered, inspect it and shape slots around it. If no master exists yet, building one is a one-time `ds-spec-authoring`-adjacent task.
2. **Workflow B coverage.** For client libraries, do we want canvas docs at all, or is the client's own Storybook enough? Lean: Workflow A only at first.
3. **Prose source.** Strong preference for reusing `specs/{component}.spec.notes.md` — but this changes the notes file's role (rationale doc → structured doc source). Alternative: separate `specs/{component}.doc.md`.
4. **`ds-storybook` overlap.** Variant gallery + stories list overlap. Decide whether `ds-doc` consumes `ds-storybook`'s output (linking to live stories) or duplicates.

---

## Pointers

- Old plugin entry: `src/main.ts` + `src/figma_functions/documentationBuilder.ts`
- Layout tree: `src/templates/templateDefault.ts`
- Anatomy tag builder (replaced by native Annotations API): `src/figma_functions/Anatomy/buildAtomTags.ts`
- Spec example to drive content: `specs/button.spec.json` (variants, anatomy.layers, sizing, tokens)
- Sibling skill to model orchestration style: `skills/ds-storybook.md`
