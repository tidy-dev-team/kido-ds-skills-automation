# DS Component Automation — Agent Instructions

This project encodes Tidy Dev's design system expertise into **two parallel pipelines** for producing production-ready Figma components — the right pipeline depends on what the client has brought to the table.

**Before acting on any request in this domain, read the relevant skill file from `skills/`.
The skill file is the authoritative instruction set — follow it exactly.**

---

## What This System Does

Turn partial or indirect client inputs into production-ready Figma component sets. Two workflows, one shared post-polish pipeline:

```
[Workflow A — ds-generate]                   [Workflow B — ds-build]
Client has no component library              Client has an existing UI library
                                             (Chakra, Mantine, shadcn, etc.)
────────────────────────────────────         ─────────────────────────────────────
Client Figma frame / screenshot /            Client library (GitHub / Storybook)
site / repo                                  + Kido's DESIGN.md (extracted tokens)
       │                                     + REQUIREMENTS.md (job rules)
       ▼                                            │
ds-generate                                         ▼
       │                                     ds-build
       ▼                                            │
Kido-structured component set                       ▼
(Kido variants + client brand colors)        Library-matched component set
                                             (library structure + DESIGN tokens)
                                                    │
                                                    ▼
                                             Validator subagent
                                             (soft-advisory checklist)

              ─────── Shared post-polish ───────
                     DS review & polish in Figma
                              │
                              ▼
                     ds-push → GitHub PR (token sync)
                              │
                              ▼
                     ds-storybook → stories PR (CSF3)
```

---

## Choosing the Workflow

| If the client… | Use |
|---|---|
| has no existing UI library — they want Kido's structure styled with their brand | **`ds-generate`** |
| already ships a UI library (Chakra/Mantine/shadcn/custom) and wants Figma to match it | **`ds-build`** |
| is updating a Kido DS component (rare — DS team only) | **`ds-spec-authoring`** |

Ask the designer early: "Does your client already have a UI library in production?" The answer determines the path.

---

## The Six Skills

### `ds-spec-authoring` — Kido DS foundation
**File:** `skills/ds-spec-authoring.md`
**When:** A new component is added to Kido DS, a spec needs updating, or you want a spec health check.
**Who:** DS team. Not a per-project task.
**Input:** Component name + Kido DS Figma URL.
**Output:** `specs/{component}.spec.json` + `specs/{component}.spec.notes.md` + updated `specs/_index.json`.

### `ds-extract-design` — per-client token extraction (Workflow B)
**File:** `skills/ds-extract-design.md`
**When:** Starting a new client project that uses Workflow B. Extracts the client's (or Kido's) foundation file into a DTCG-formatted DESIGN.md.
**Who:** Designer, once per project.
**Input:** Figma foundation file URL.
**Output:** `working/{project}/DESIGN.md` — Markdown + DTCG JSON blocks (Colors, Typography, Spacing, Radius, Shadow, Themes). The `{project}` is a short slug the designer names at the start (e.g., `acme`, `payzo`).

### `ds-generate` — Workflow A orchestrator
**File:** `skills/ds-generate.md`
**When:** Client has no UI library. Designer provides a Figma frame, screenshot, description, live site URL, or code repository.
**Who:** Designer, per client project.
**Input:** Any client input carrying brand values (colors, radius, font).
**Output:** Kido-structured component set built in Figma via MCP (54 or 108 variants).
**Philosophy:** Classify → extract brand values → map tokens → execute. No intermediate files. Stubs for unknowns.

### `ds-build` — Workflow B orchestrator
**File:** `skills/ds-build.md`
**When:** Client has an existing UI library. Designer wants Figma components that mirror the library's structure.
**Who:** Designer, per client project (per component).
**Input:** Component name + library reference (GitHub repo / Storybook URL / site) + DESIGN.md + REQUIREMENTS.md.
**Output:** Library-matched component set in Figma + `validation-report.md` (soft advisory).
**Philosophy:** Library provides structure. DESIGN.md provides style. REQUIREMENTS.md provides project rules. Validator catches hallucinations.

### `ds-push` — sync polished Figma back to code
**File:** `skills/ds-push.md`
**When:** After DS specialist polishes the generated component in Figma.
**Who:** DS specialist or designer.
**Input:** Polished Figma component set URL + GitHub repo URL.
**Output:** GitHub PR with updated CSS variables, Tailwind config, and/or component source.
**Philosophy:** Read resolved values from Figma → diff → surgically update repo files. Never rewrites structure — only values.
**Works for both workflows.**

### `ds-storybook` — Storybook documentation
**File:** `skills/ds-storybook.md`
**When:** Component is production-ready and needs Storybook documentation.
**Who:** DS specialist or developer.
**Input:** Component name + GitHub repo URL.
**Output:** GitHub PR with CSF3 `{component}.stories.tsx` + setup instructions if Storybook is missing.
**Philosophy:** Map Figma variant axes → React props → 8–12 named stories. Pseudo-state axes become decorators.
**Works for both workflows.**

---

## Spec & Reference Library

**Kido component specs** (`specs/*.spec.json`): authoritative structure for Workflow A. Read `specs/_index.json` first for lookup.

**Library mappings** (`specs/libraries/*.json`): conventions of known UI libraries (Chakra, Mantine, shadcn) — variant prop names, size scales, compound component patterns. Used by `ds-build` when a library is detected from a `package.json`.

**REQUIREMENTS template** (`skills/templates/REQUIREMENTS.template.md`): starting point for per-job rules. Used by `ds-build` Step 0.

Component tiers (implementation priority):
- **Tier 1:** Button, Input, Toggle, Checkbox
- **Tier 2:** Select, Card, Tab Bar, Badge
- **Tier 3:** Modal, Navigation, Table, Form

When no spec exists for the component being worked on, each workflow has its own "no spec" protocol — see the individual skill file.

---

## Figma MCP

Used across all workflows for reading Kido/client foundations and writing components into Figma.

| Tool | Used in | Purpose |
|------|---------|---------|
| `get_metadata` | spec-authoring | Locate component, get variant IDs |
| `get_design_context` | spec-authoring, generate, build | Layer structure, variants, tokens |
| `get_variable_defs` / `figma_get_variables` | spec-authoring, extract-design | Design token definitions |
| `figma_get_styles` | extract-design | Color/text/effect styles |
| `figma_get_design_system_kit` | extract-design | Combined token + component extraction |
| `get_screenshot` | generate, build | Visual input/validation |
| `figma_execute` / `use_figma` | generate, build | Write component variants into Figma |
| `figma_get_component_details` | build (validator) | Read back applied values |

---

## File Locations

```
specs/
  _index.json                      ← Kido component lookup
  {component}.spec.json            ← compact Kido component spec
  {component}.spec.notes.md        ← human-readable rationale
  libraries/
    _index.json                    ← UI library mapping lookup
    chakra.json                    ← Chakra UI conventions
    mantine.json                   ← Mantine conventions
    shadcn.json                    ← shadcn/ui + Radix conventions

skills/
  ds-spec-authoring.md
  ds-generate.md
  ds-extract-design.md             ← new — Workflow B prerequisite
  ds-build.md                      ← new — Workflow B orchestrator
  ds-push.md
  ds-storybook.md
  templates/
    REQUIREMENTS.template.md       ← new — per-job rules template

working/                           ← local session artifacts (gitignored)
  {component}-{YYYY-MM-DD}/             ← Workflow A (per generation session)
  {project}/                            ← Workflow B (per project, shared across components)
    DESIGN.md                      ← extracted per-project (Workflow B)
    REQUIREMENTS.md                ← per-job rules (Workflow B)
    library-snapshot.json          ← resolved library structure (Workflow B)
    token-map.json                 ← resolved tokens (both workflows)
    resolved-stubs.json            ← designer-provided values (Workflow A)
    validation-report.md           ← validator output (Workflow B)
    push-summary.json              ← ds-push diff + PR URL

.claude/
  commands/
    ds-spec-authoring.md
    ds-generate.md
    ds-extract-design.md           ← new
    ds-build.md                    ← new
    ds-push.md
    ds-storybook.md

CLAUDE.md                          ← this file (auto-loaded by Claude Code)
README.md                          ← human-facing overview
```

---

## Critical Rules

**Read the skill file first.** Skill files are authoritative. CLAUDE.md summaries can lag behind skill updates.

**Never fabricate token values.** Stub as `NEEDS_VALUE` (shown as `#FF0066` fill in Figma). A visible stub is better than an invented value.

**Never force Inter as the font.** Extract the client's font; find closest available match if the exact font isn't in Figma. Inter is a last resort only when the client uses a generic system-ui stack.

**Stubs are valid output.** Designer polishes them in Figma after generation.

**Default to compact generation (Workflow A).** `inverse=no` only, unless dark-background support is explicitly requested.

**Workflow A preserves client styling through token mapping.** Kido provides structure. Client's input provides brand values. Output should look like the client's brand.

**Workflow B preserves library structure.** Library dictates the component anatomy and prop axes. DESIGN.md dictates visual values. REQUIREMENTS.md dictates scope overrides. The validator checks all three.

**Specs are data, not instructions.** Token names are self-documenting.

---

## When Things Go Wrong

**Spec and Kido have diverged:** Run `/ds-spec-authoring check [component]`.

**Classification confidence is low (Workflow A):** Ask to confirm component type before proceeding.

**No Kido spec for this component (Workflow A):** Follow "No Matching Spec" in `skills/ds-generate.md`.

**Library not identified (Workflow B):** Fall back to Kido spec as structure source. Note deviation in `token-map.json`.

**Validation report has errors (Workflow B):** Report runs as soft advisory — errors don't block completion. Designer decides whether to fix and re-run or accept as-is.

**Generation plan has stubs:** Expected. Stubs appear as bright pink fills in Figma. Designer fills them in during polish.
