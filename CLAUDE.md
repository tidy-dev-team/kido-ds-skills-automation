# DS Component Automation — Agent Instructions

This project encodes Tidy Dev's design system expertise into **two parallel pipelines** for producing production-ready Figma components — the right pipeline depends on what the client has brought to the table.

**Before acting on any request in this domain, read the relevant skill file from `skills/`.
The skill file is the authoritative instruction set — follow it exactly.**

**Use shared vocabulary.** `LANGUAGE.md` (in the project root) is the canonical glossary — terms, roles, artifact paths, verbs, and skill boundaries. When skills disagree with LANGUAGE.md, the glossary wins; update the skill, not the glossary.

---

## What This System Does

Turn partial or indirect client inputs into production-ready Figma component sets. Two workflows, one shared post-polish pipeline:

```
[Workflow A — ds-generate]                   [Workflow B — ds-build]
Client has no component library              Client has an existing UI library
                                             (Chakra, Mantine, shadcn, etc.)
────────────────────────────────────         ─────────────────────────────────────
Client Figma frame / screenshot /            Client library (GitHub / Storybook)
site / repo                                  + tokens.json + DESIGN.md
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
| is unsure where to start, or wants the system to ask the right questions | **`ds-guide`** |

Ask the designer early: "Does your client already have a UI library in production?" The answer determines the path. If the designer can't answer, point them at `/ds-guide` — the wizard asks the same question with a built-in disambiguator.

---

## The Eight Skills

### `ds-guide` — guided wizard entry point
**File:** `skills/ds-guide.md`
**When:** First-time user, or anyone unsure which skill to invoke.
**Who:** Designer / DS specialist.
**Input:** none — the wizard asks for what it needs.
**Output:** none directly — routes to the appropriate skill via the `Skill` tool with the collected inputs.
**Philosophy:** Routing, not duplication. Direct invocation of `ds-generate`, `ds-build`, etc. remains fully supported. The wizard is opt-in.

### `ds-spec-authoring` — Kido DS foundation
**File:** `skills/ds-spec-authoring.md`
**When:** A new component is added to Kido DS, a spec needs updating, or you want a spec health check.
**Who:** DS team. Not a per-project task.
**Input:** Component name + Kido DS Figma URL.
**Output:** `specs/{component}.spec.json` + `specs/{component}.spec.notes.md` + updated `specs/_index.json`.
**Reads `specs/{component}.guidelines.json`** when present to draft Usage Guidelines prose in `.spec.notes.md` (filters to `recommended=true`, type → section mapping, synthesised into Kido voice — never pasted verbatim). The curated `.spec.notes.md` is what `/ds-doc` renders.

### `ds-extract-design` — per-client token extraction (Workflow B)
**File:** `skills/ds-extract-design.md`
**When:** Starting a new client project that uses Workflow B. Extracts the client's (or Kido's) foundation file into two sibling files: a DTCG `tokens.json` and a prose `DESIGN.md`.
**Who:** Designer, once per project.
**Input:** Figma foundation file URL.
**Output:** Two files in `working/{project}/`:
- `tokens.json` — full W3C DTCG token tree (Colors, Typography, Spacing, Rounded, Shadow, Components, Themes). Source of token truth, consumed by `/ds-build`.
- `DESIGN.md` — google-labs-code/design.md-compliant front matter (token summary) + prose body (rationale, do's/don'ts, gaps). Lints cleanly with `npx @google/design.md lint`.

The `{project}` is a short slug the designer names at the start (e.g., `acme`, `payzo`).

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
**Input:** Component name + library reference (GitHub repo / Storybook URL / site) + `tokens.json` + `DESIGN.md` + REQUIREMENTS.md.
**Output:** Library-matched component set in Figma + `validation-report.md` (soft advisory).
**Philosophy:** Library provides structure. `tokens.json` provides values; `DESIGN.md` provides rationale. REQUIREMENTS.md provides project rules. Validator catches hallucinations.

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

### `ds-doc` — component documentation as chat markdown
**File:** `skills/ds-doc.md`
**When:** Component is polished and needs written documentation (variant descriptions + Usage Guidelines prose). Sibling to `ds-storybook` — both run after polish; `ds-storybook` produces code-side CSF3 stories, `ds-doc` produces markdown documentation prose the designer can paste into Figma rich-text, spec notes, Notion, or wherever.
**Who:** DS specialist or designer.
**Input:** Polished Figma component-set URL.
**Output:** One markdown document returned in the chat — variant descriptions (one per semantic variant) + Usage Guidelines (When-to-use / When-not-to-use / General / Behavior / Content / Accessibility). No files written, no canvas edits.
**Philosophy:** Read-only on Figma via the official MCP (`get_design_context`, `get_metadata`, `get_screenshot`, `get_variable_defs`). Pulls Usage Guidelines source material from `specs/{component}.guidelines.json` (filters to `recommended=true`) when present; otherwise searches the web for industry best practices (Material, Carbon, Atlassian, Primer, Spectrum, Polaris, Wise, Orbit) and synthesises into Kido voice. Variant descriptions are grounded in the polished component-set's actual variant axes + any relevant design decisions from `.spec.notes.md`.
**Both workflows supported** — no `tokens.json` requirement. The earlier canvas-rendering version (three Figma doc pages, master template cloning) is retired; see [`docs/adr/0001-ds-doc-three-page-architecture.md`](docs/adr/0001-ds-doc-three-page-architecture.md) for historical rationale.

---

## Spec & Reference Library

**Kido component specs** (`specs/*.spec.json`): authoritative structure for Workflow A. Read `specs/_index.json` first for lookup.

**Competitive Do/Don't research** (`specs/*.guidelines.json`): prose guidance from external design systems (Material, Carbon, Atlassian, Primer, Spectrum, Orbit, Wise, Kiwi), tagged by type (`Accessibility | Best | Description | Do | Not`), source, and Kido's `recommended` stance. Source material for spec authoring and `/ds-doc` Usage Guidelines prose. Not the same as `*.dodont.json` (visual example refs — legacy from the retired canvas-rendering `/ds-doc`; left in place for any future canvas tooling that wants them).

**Library mappings** (`specs/libraries/*.json`): conventions of known UI libraries (Chakra, Mantine, shadcn) — variant prop names, size scales, compound component patterns. Used by `ds-build` when a library is detected from a `package.json`.

**REQUIREMENTS template** (`skills/templates/REQUIREMENTS.template.md`): starting point for per-job rules. Used by `ds-build` Step 0.

Component tiers (implementation priority):
- **Tier 1:** Button, Input, Toggle, Checkbox
- **Tier 2:** Select, Card, Tab Bar, Badge
- **Tier 3:** Modal, Navigation, Table, Form

When no spec exists for the component being worked on, each workflow has its own "no spec" protocol — see the individual skill file.

---

## Figma MCP

**Canonical transport: the official `claude.ai Figma` MCP server.** Every Figma interaction in every skill goes through it — reads and writes alike. Do not fall back to `figma-console` or any other third-party Figma plugin/MCP. If the official MCP is unavailable, stop and ask the designer to authenticate it (`/mcp` → `claude.ai Figma`) before continuing.

| Tool | Used in | Purpose |
|------|---------|---------|
| `get_metadata` | spec-authoring | Locate component, get variant IDs |
| `get_design_context` | spec-authoring, generate, build, push | Layer structure, variants, resolved tokens (React+Tailwind reference output) |
| `get_variable_defs` | spec-authoring, extract-design, push | Design token / variable definitions in the file |
| `get_screenshot` | generate, build, doc, push, storybook | Visual input / post-write validation |
| `get_libraries` | extract-design, doc | Subscribed and available libraries for a file |
| `search_design_system` | doc, build, generate | Locate published components / variables / styles across subscribed libraries (replaces `figma_search_components`) |
| `use_figma` | every write-touching skill | General-purpose Plugin-API runner. Replaces `figma_execute`, `figma_get_component_details` (custom read scripts), and every writer-side `figma_*` tool. Takes a `fileKey` arg, so cross-file flows do not require Bridge switching. |
| `upload_assets` | (future) | PNG/JPG/GIF/WebP into Figma fills |
| `whoami` | (diagnostic) | Confirm authenticated identity + seat type when a write fails |

### `use_figma` runtime notes

The official MCP sandboxes the Plugin API on Figma's hosted service rather than a local Desktop Bridge. Two gotchas worth knowing:

- **`figma.loadAllPagesAsync()` is unsupported** — the document is pre-loaded server-side. Strip the call from any script ported from `figma-console`.
- **`figma.currentPage = page` is unsupported** — use `await figma.setCurrentPageAsync(page)`.
- **Plugin-data is shared-only** — `getPluginData` / `setPluginData` aren't available (they require a plugin manifest). Use `getSharedPluginData(namespace, key)` / `setSharedPluginData(namespace, key, value)` with a stable namespace.
- **Inter weights have spaces** — `"Semi Bold"`, `"Extra Bold"` — not `"SemiBold"` / `"ExtraBold"`.

### Seat / permission requirements

`use_figma`, `upload_assets`, and any other write tool require a **Full or Dev seat** on the plan that owns the target file. View / Collab seats can only call read tools, and are capped at 6 tool calls per month total. If a write fails with "this figma file could not be accessed", run `whoami` first — most often the user is authenticated with the wrong identity (e.g. a personal account with View access instead of the team account with Full access).

---

## File Locations

```
specs/
  _index.json                      ← Kido component lookup (schema_version 0.3)
  {component}.spec.json            ← compact Kido component spec
  {component}.spec.notes.md        ← human-readable rationale + optional Usage Guidelines prose (informs /ds-doc)
  {component}.dodont.json          ← Do/Don't visual example refs (Kido DS Figma node IDs) — legacy, retained for future canvas tooling
  {component}.guidelines.json      ← prose Do/Don't research, multi-source (Material, Carbon, Atlassian, Primer, Spectrum, Orbit, Wise, …) with Kido's `recommended` stance — primary source for /ds-doc Usage Guidelines
  libraries/
    _index.json                    ← UI library mapping lookup
    chakra.json                    ← Chakra UI conventions
    mantine.json                   ← Mantine conventions
    shadcn.json                    ← shadcn/ui + Radix conventions

skills/
  ds-guide.md                      ← guided wizard entry point
  ds-spec-authoring.md
  ds-generate.md
  ds-extract-design.md             ← Workflow B prerequisite
  ds-build.md                      ← Workflow B orchestrator
  ds-push.md
  ds-storybook.md
  ds-doc.md                        ← component documentation as chat markdown (variant descriptions + Usage Guidelines)
  templates/
    REQUIREMENTS.template.md       ← per-job rules template
    CHANGES.template.md            ← per-project change log template
    QUALITY_STANDARDS.md           ← Kido DS baseline (applies to every component)

docs/
  adr/                             ← architectural decision records
    0001-ds-doc-three-page-architecture.md

working/                           ← local session artifacts (gitignored)
  {component}-{YYYY-MM-DD}/             ← Workflow A (per generation session)
  {project}/                            ← Workflow B (per project, shared across components)
    tokens.json                    ← DTCG token tree, source of truth (Workflow B)
    DESIGN.md                      ← prose rationale + spec-compliant token summary (Workflow B)
    REQUIREMENTS.md                ← per-job rules (Workflow B)
    library-snapshot.json          ← resolved library structure (Workflow B)
    token-map.json                 ← resolved tokens (both workflows)
    resolved-stubs.json            ← designer-provided values (Workflow A)
    validation-report.md           ← validator output (Workflow B)
    push-summary.json              ← ds-push diff + PR URL
    CHANGES.md                     ← per-project change log, two-category split

DESIGN-SYSTEM.md                   ← authoritative cross-project token rules (read by every skill at Step 0)

.claude/
  commands/
    ds-guide.md                    ← wizard slash command
    ds-spec-authoring.md
    ds-generate.md
    ds-extract-design.md
    ds-build.md
    ds-push.md
    ds-storybook.md

CLAUDE.md                          ← this file (auto-loaded by Claude Code)
LANGUAGE.md                        ← canonical glossary (terms, roles, paths, verbs)
README.md                          ← human-facing overview
documentation/
  ARCHITECTURE.md                  ← planned target architecture
```

---

## Critical Rules

**Update `CHANGELOG.md` on every meaningful change.** Whenever you modify `skills/**`, `specs/**`, `CLAUDE.md`, `LANGUAGE.md`, `README.md`, or `.claude/commands/**`, add an entry under `## [Unreleased]` in `CHANGELOG.md` (categories: Added / Changed / Fixed / Removed / Deprecated / Security). Skip entries for `working/**`, logs, and formatting-only edits. See `CHANGELOG.md` for format.

**Read `DESIGN-SYSTEM.md` first if present.** Every skill that runs in this repo must check the repo root for `DESIGN-SYSTEM.md` as its first action. If present, it is **authoritative** — overrides any default behavior in the skill (token decision tree, state ownership, modes-as-brands, etc.). If absent, skills proceed with their defaults; never invent rules to fill the gap.

**Per-project changes live in `working/{project}/CHANGES.md`, split into two categories.** While working on a client project, when you make a change to skills/specs/templates that grew out of this work, add it to `working/{project}/CHANGES.md` under one of:
- **Generic improvements** — applies to any future project. Candidate for the next PR to `main`.
- **Project-specific** — only makes sense inside this project. Must not travel to `main`.
Use `skills/templates/CHANGES.template.md` as the starting point for new projects. When the project finishes, lift the **Generic** section into a `CHANGELOG.md` entry + matching skill/spec edits; leave **Project-specific** in the working dir.

**Read the skill file first.** Skill files are authoritative. CLAUDE.md summaries can lag behind skill updates.

**Never fabricate token values.** Stub as `NEEDS_VALUE` (shown as `#FF0066` fill in Figma). A visible stub is better than an invented value.

**Never force Inter as the font.** Extract the client's font; find closest available match if the exact font isn't in Figma. Inter is a last resort only when the client uses a generic system-ui stack.

**Stubs are valid output.** Designer polishes them in Figma after generation.

**Default to compact generation (Workflow A).** `inverse=no` only, unless dark-background support is explicitly requested.

**Workflow A preserves client styling through token mapping.** Kido provides structure. Client's input provides brand values. Output should look like the client's brand.

**Workflow B preserves library structure.** Library dictates the component anatomy and prop axes. `tokens.json` dictates visual values (DESIGN.md is the human-readable companion). REQUIREMENTS.md dictates scope overrides. The validator checks all three.

**Specs are data, not instructions.** Token names are self-documenting.

---

## When Things Go Wrong

**Spec and Kido have diverged:** Run `/ds-spec-authoring check [component]`.

**Classification confidence is low (Workflow A):** Ask to confirm component type before proceeding.

**No Kido spec for this component (Workflow A):** Follow "No Matching Spec" in `skills/ds-generate.md`.

**Library not identified (Workflow B):** Fall back to Kido spec as structure source. Note deviation in `token-map.json`.

**Validation report has errors (Workflow B):** Report runs as soft advisory — errors don't block completion. Designer decides whether to fix and re-run or accept as-is.

**Generation plan has stubs:** Expected. Stubs appear as bright pink fills in Figma. Designer fills them in during polish.
