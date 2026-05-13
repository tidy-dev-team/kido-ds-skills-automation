# ADR 0001 — `/ds-doc` three-page architecture

**Status:** Accepted
**Date:** 2026-05-12
**Deciders:** Dmitri (DS engineering), Kido design team
**Supersedes:** `documentation/ds-doc-research.md` (research-phase recommendation; the single-page slot-template plan was abandoned in favor of the three-page layout that designers delivered)

---

## Context

The DS automation system needs a skill that builds component documentation on the Figma canvas — the eighth `ds-*` skill, sibling to `/ds-storybook` (code-side docs). Earlier research recommended a single doc page composed of named slots filled from spec + polished Figma + prose. Designers then delivered a richer vision in the Kido <> Claroty engagement file: **three separate doc pages per component**, each with its own structure, status lifecycle, and visual language.

The choice between the single-page research plan and the three-page designer vision is hard to reverse once master templates are authored in the Kido DS canonical Figma file and projects start rendering docs against them. This ADR captures why the three-page architecture won and what trade-offs we accepted.

## Decision

`/ds-doc` produces **three Figma doc pages per component**, all in one run, into a dedicated `📄 Documentation` page in the target file. The three pages are:

1. **Component Breakdown** — variants grid (rows × columns derived from the spec's `doc.variants_grid` axis declaration), max width/height, min width/height, icon-position variations
2. **Mode** — a grid of cells, one per `theme.*` entry in `tokens.json`, each showing the polished component rendered in that mode/brand
3. **Usage Guidelines** — prose-heavy: When-to-use / When-not-to-use / General / Behavior (Do/Don't pairs) / Content (Do/Don't) / Look & Feel (Do/Don't)

Each page is a clone of a canonical Kido DS master template (`doc-template / component-breakdown`, `doc-template / mode`, `doc-template / usage-guidelines`). Masters contain layers named `slot/{section-id}` that the agent fills. Repeating slots (Do/Don't pairs) clone a `pair-template` component N times.

Data sources per section:

| Section | Source |
|---|---|
| Header title / status pill | Hardcoded literal + page-type lookup (status renders as `IDEATION`) |
| Variants grid axes | `specs/{component}.spec.json` → `doc.variants_grid.rows/cols` |
| Variants grid cells | Polished Figma component-set variant matrix (live read) |
| Max / Min width-height | `specs/{component}.spec.json` → `sizing.{size}.max_*` / `min_*` |
| Icon positions | Derived from `anatomy.component_properties` BOOLEAN + INSTANCE_SWAP pairs (auto-detect) |
| Mode grid cells | `working/{project}/tokens.json` → iterate `theme.*` |
| Usage Guidelines prose | `specs/{component}.spec.notes.md` top sections (When to use / When not to use / General guidelines / Accessibility / Behavior / Content / Look & Feel) |
| Do/Don't visual examples | `specs/{component}.dodont.json` mapping rule slugs → Kido DS Figma node IDs |

## Considered alternatives

### Single doc page with named slots (the research-phase plan)

The original recommendation. One page per component with header, anatomy callouts, variants gallery, tokens table, do/don'ts, accessibility.

Rejected because designers delivered a richer artifact. The three-page layout separates concerns the single-page plan had collapsed: visual variants, mode coverage, and usage rules each get their own canvas. This matches how designers actually present component docs (one decision-supporting artifact per question type), and lets each page evolve independently. It costs more master-authoring upfront and triples the surface area /ds-doc has to render, but the per-page rendering logic is simpler than a single page that has to interleave all section types.

### Per-project templates (each engagement designs its own)

Each client project file authors its own doc-page masters; /ds-doc reads whatever templates exist in the target file.

Rejected because docs would drift across clients and Kido couldn't evolve a consistent doc UX. The Claroty file is treated as the design-in-progress for the canonical Kido masters — once approved, they promote to Kido DS and every project clones from there. Hybrid override (Kido default + per-project) was considered but adds template-resolution complexity for a benefit no client has yet asked for.

### Procedural builder (port of the old Tidy Guide Builder plugin)

Build doc pages from auto-layout primitives, no master templates. Generates everything fresh each run.

Rejected for the same reason the original research rejected it: procedural layout is brittle and visually drifts from designer intent. The old plugin's biggest failure mode was visual divergence between hand-authored and auto-built sections. Slot-based template cloning keeps the polish in designer hands.

### Usage Guidelines prose in a separate `{component}.doc.md` file

Keep `.spec.notes.md` Kido-internal (Design Decisions, Token Derivation); put user-facing prose in a sibling file.

Rejected because it doubles the per-component file count for marginal separation. `.spec.notes.md` already accepts new sections; extending it preserves one-file-per-component and keeps the doc-authoring workflow in the same file the DS team already maintains. Trade-off: `.spec.notes.md` now mixes audiences (Kido rationale + user-facing prose), but heading conventions make the boundary explicit.

### Agent-generated Do/Don't counter-examples

Have /ds-doc invent Don't examples programmatically by perturbing the component (wrong color, missing label, etc.).

Rejected as too risky. Counter-examples for nuanced UX rules ("don't use a primary action in a side-panel") can't be derived from the component alone — they require product context. Curated pairs in `dodont.json` referencing Kido DS Figma nodes keep the examples designer-controlled. Missing entries render as `NEEDS_EXAMPLE` pink stubs (matching existing `NEEDS_VALUE` / `NEEDS_CONTENT` conventions).

### Workflow A coverage in v1

Have /ds-doc work for both workflows; skip the Mode page when `tokens.json` is absent (Workflow A doesn't produce one).

Rejected for v1. The Claroty engagement is Workflow B and that's where the immediate need is. Workflow A docs can be added once the v1 templates prove out — the Mode page in particular only makes sense when there's a `theme.*` block to iterate. Forcing single-mode rendering for Workflow A produces awkward one-cell pages.

## Consequences

### Positive

- Each doc page has a single coherent purpose; designers can polish them independently
- Master templates stay in Kido DS as the visual source of truth; consistent UX across client projects
- `.spec.notes.md` becomes the per-component prose home — natural fit with existing DS-team workflow
- `dodont.json` sidecar lets designers manage Do/Don't examples without touching prose
- Schema 0.3 additions are minimal: `doc.variants_grid` + optional `sizing.{size}.max_*`
- Re-run overwrites in place — designer can iterate without manual cleanup
- Missing data fails loudly (pink stubs) rather than silently — fits existing failure-mode conventions

### Negative

- Three master templates to author and maintain in Kido DS (vs one)
- Skill can't run end-to-end until masters are promoted from the Claroty file to Kido DS canonical
- `.spec.notes.md` now mixes Kido-internal rationale with user-facing prose; reader must understand the heading taxonomy
- Spec schema 0.3 bump requires touching all six existing specs to declare `doc.variants_grid`
- `dodont.json` becomes a new per-component artifact the DS team must author for full doc coverage

### Neutral

- Workflow B-only v1 caps reach; Workflow A coverage is a known follow-up
- Status pill lifecycle (IDEATION → READY → PUBLISHED) is convention-only; no enforcement
- Designer manually advances status; agent always renders IDEATION

## Open questions / follow-ups

- Workflow A support — when there's demand, define how the Mode page behaves with no `theme.*` block
- Anatomy callouts (numbered annotations on a default instance) — not in the v1 layout; if designers add a fourth page (Anatomy), this ADR will need an update
- Tokens table on the doc page — not in v1; Dev Mode + Code Connect cover the engineering audience for now
- Accessibility section — **added 2026-05-13** to the Usage Guidelines page between General guidelines and Behavior, prompted by the Tidy Do's and Don'ts sheet import (which had `Accessibility` as a first-class category). Master template `doc-template / usage-guidelines` needs a new `slot/accessibility` layer; designer task, pending.
