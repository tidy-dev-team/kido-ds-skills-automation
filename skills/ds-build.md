---
name: ds-build
description: >
  Build a production-ready Figma component set that matches a client's existing UI library
  (Chakra, Mantine, shadcn, or custom), styled with tokens from DESIGN.md and scoped by
  REQUIREMENTS.md. Runs a validator subagent at the end to catch hallucinations.
  Triggers on phrases like "build a button for this library", "create component matching Chakra",
  "build from storybook", "build component using our library", or when a designer shares a
  library reference (GitHub repo / Storybook URL) alongside a component name.
---

# DS Build

**Library structure + DESIGN.md tokens + REQUIREMENTS.md rules → Figma component set.**

Used when the client has an existing UI library and wants the Figma components to mirror it.
For the flow where the client has **no** library (Kido structure, client brand), use `/ds-generate` instead.

---

## Prerequisites

Before running, `/ds-build` expects:

1. **DESIGN.md** — at `working/{project}/DESIGN.md`. Produced by `/ds-extract-design`. If missing, the skill asks the designer to run that first. The `{project}` is a short slug set during extraction (e.g., `acme`, `payzo`).
2. **Library reference** — GitHub repo URL, Storybook URL, or package name. At least one.
3. **Component name** — the component to build (e.g., "Button", "Input"). Looked up in library conventions first, Kido specs second.

REQUIREMENTS.md is **not** a prerequisite — if it's missing, Step 0 interviews the designer and writes it.

---

## Step 0 — Project Name + REQUIREMENTS Check

**First, resolve the project name.** All Workflow B artifacts for a project share a single `working/{project}/` directory. If the designer hasn't named the project yet:
- Ask: "What's the project name? (A short slug like `acme` or `payzo` — used for the working directory.)"
- Once set, all subsequent `/ds-build` and `/ds-extract-design` calls for this project use the same `working/{project}/` directory.

**Then, check for REQUIREMENTS.md** at `working/{project}/REQUIREMENTS.md`.

**If present** → read it. Proceed.

**If missing** → run a 5-question interview, then write the answers to `working/{project}/REQUIREMENTS.md` using the template at `skills/templates/REQUIREMENTS.template.md`.

Interview questions:

1. **Figma build target?** Which Figma file + page should the component be built on? Ask for a URL or page name. (required — no default)
2. **Library?** What UI library does the client use? GitHub repo URL / Storybook URL / library name (Chakra, Mantine, shadcn, custom). (required — no default)
3. **Color modes required?** light / dark / both (default: both if DESIGN.md has dark mode, else light only)
4. **Locales / RTL?** single locale / multi-locale with RTL (e.g., en + he) (default: single)
5. **Naming prefix?** Any prefix required on Figma node names or variables (e.g., `pz_`)? (default: none)
6. **Accessibility target?** WCAG AA (default) / AAA
7. **Component-specific constraints?** Free text — anything else the designer knows about this job (optional)

Keep questions short. Offer a single "accept defaults" option if the designer is in a hurry (questions 3–7 only — build target and library are always required).

---

## Step 1 — Resolve the Library

Priority chain. Stop at the first source that resolves.

### 1a. Storybook URL provided

`WebFetch` `{storybookUrl}/stories.json` (Storybook 7) or `{storybookUrl}/index.json` (Storybook 8+).

Extract:
- Story entries matching the component (e.g., all stories under `Button/*`)
- `argTypes` → variant axes + prop controls
- Default `args` → default values
- Compound structure (e.g., `ButtonGroup` as a parent of `Button`)

Save to `working/{project}/library-snapshot.json` as `{source: "storybook", stories: [...], argTypes: {...}}`.

### 1b. GitHub repo URL provided

```bash
gh api repos/{owner}/{repo}/contents/package.json --jq '.content' | base64 -d
```

Parse `dependencies` and `devDependencies`. Look for:
- `@chakra-ui/react` → Chakra
- `@mantine/core` → Mantine
- `@radix-ui/*` or presence of `components/ui/{component}.tsx` → shadcn/ui

If a known library is detected, load `specs/libraries/{library}.json` for the variant/prop conventions.

Then read the component source:
```bash
gh api repos/{owner}/{repo}/contents/src/components/{Component}.tsx
```

or the library's equivalent path. Extract:
- Prop interface (TypeScript `interface` or `type Props = ...`)
- Variant definitions (CVA calls, styled-components, theme.ts entries)
- Compound component children

Save to `working/{project}/library-snapshot.json` as `{source: "github", library: "chakra", props: {...}, variants: {...}}`.

### 1c. Package/library name only

If the designer only named a library ("build for Chakra"), load `specs/libraries/{library}.json` as the convention source. Proceed with generic component anatomy from the library mapping. Note the low-fidelity source in `token-map.json`.

### 1d. Fallback — no library resolvable

Load `specs/{component}.spec.json` (Kido spec). Use Kido's variant axes and anatomy. Mark `token-map.json` with `"structure_source": "kido_spec_fallback"` and list the deviation.

---

## Step 2 — Map DESIGN.md Tokens to Library Slots

For each visual slot in the library's component anatomy (background, text, border, focus ring, disabled state, etc.), find the matching semantic token in DESIGN.md and record the mapping.

Mapping priority:

1. **Exact name match** — e.g., library's `bg="brand"` → DESIGN.md `color.brand.500`.
2. **Semantic match** — library's `primary` variant → DESIGN.md `color.interactive.primary`.
3. **Primitive fallback** — library uses a hex directly → keep the hex, note in `token-map.json` that it wasn't mapped to a semantic.
4. **Stub** — no match possible → `NEEDS_VALUE` (pink `#FF0066` at render time).

Apply REQUIREMENTS overrides:
- If `naming_prefix` is set, prefix all Figma node names.
- If `modes: ["dark"]` only, skip building light variants.
- If locales include RTL, set `textDirection: RTL` on label text nodes for RTL variants.

Write `working/{project}/token-map.json`:
```json
{
  "component": "Button",
  "library": "chakra",
  "structure_source": "storybook",
  "modes": ["light", "dark"],
  "variant_axes": { "variant": ["solid", "outline", "ghost"], "size": ["sm", "md", "lg"] },
  "slot_mapping": {
    "solid.bg":      "color.interactive.primary",
    "solid.text":    "color.text.onBrand",
    "outline.border":"color.interactive.primary"
  },
  "stubs": []
}
```

---

## Step 3 — Build in Figma

Use `figma_execute` or `use_figma` to create the component set. Structure:

- **Component set name** — follow library convention (e.g., Chakra → `Button`; shadcn → `button`)
- **Variant properties** — exactly the library's axes (from `library-snapshot.json`)
- **Component properties** — non-variant props (boolean `isLoading`, text `children`) as instance swap / boolean / text properties, not variants
- **Anatomy** — mirror the library's compound structure. Button with Icon becomes a parent frame with `LeftIcon`, `Label`, `RightIcon` children matching the library's naming
- **Styling** — apply resolved token values from `slot_mapping`
- **Modes** — if REQUIREMENTS requests multiple modes, use Figma variables for mode-bound tokens so switching modes works at the component level
- **Font handling** — per the font rules in `skills/ds-generate.md` Step 2 "Font resolution": extract the client's font from DESIGN.md; find closest available match in Figma; never default silently to Inter

Generate all combinations implied by `variant_axes` × modes. For locales with RTL, generate mirrored variants where the library's behavior requires them.

Save the resulting component set node ID to `token-map.json` under `figma_node_id`.

---

## Step 4 — Validator Subagent

Invoke a validator via the `Agent` tool. Hand it:

- Figma node ID of the generated component set
- `library-snapshot.json`
- `token-map.json`
- `REQUIREMENTS.md`
- `DESIGN.md`

Validator prompt template:

> You are a design-system validator. Review the Figma component set at node `{nodeId}` in file `{fileKey}`. Check it against the checklist below. For each finding, output severity (`error` / `warning` / `info`), category, and a one-sentence description. Write the result to `working/{project}/validation-report.md`.
>
> **Checklist:**
> 1. **Structure fidelity** — does the component tree match the library anatomy in `library-snapshot.json`? Slot count, nesting, compound children all present?
> 2. **Token application** — read the Figma fills / strokes / typography back (via `figma_get_component_details`). Do they match `slot_mapping` in `token-map.json`?
> 3. **Variant completeness** — are all combinations from `variant_axes` × modes present?
> 4. **Naming conventions** — do all node names honor the `naming_prefix` from REQUIREMENTS?
> 5. **Accessibility** — focus ring on every focused variant, WCAG contrast met per REQUIREMENTS target, touch target ≥ 44×44 on interactive elements.
> 6. **Mode/locale constraints** — only the modes/locales listed in REQUIREMENTS are present. No extras.
>
> Be concise. Group findings by category. Report all 6 categories even if one has no findings ("No issues found").

**Soft advisory** — the validator produces a report. `/ds-build` continues regardless of findings. The designer reads the report and decides.

---

## Step 5 — Report

Surface a concise summary to the designer:

```
Button built — matches {library} conventions.

Figma:      {fileKey}:{nodeId}
Variants:   {N} variants across {axes}
Modes:      {modes}
Tokens:     {resolved_count} resolved, {stub_count} stubs
Validator:  {errors} errors, {warnings} warnings, {infos} notes
            → working/{project}/validation-report.md

Artifacts saved to working/{project}/:
  library-snapshot.json
  token-map.json
  validation-report.md
```

If stubs are present, remind the designer: "Pink stubs in Figma — fill them in during polish, or re-run /ds-build once the missing values are added to DESIGN.md."

If the validator flagged errors, do not claim success. Report errors plainly and let the designer decide.

---

## No Matching Component

If the requested component name doesn't exist in the library:

1. Check `specs/libraries/{library}.json` for the library's component catalog. If the name has a different canonical form there (e.g., designer said "Dropdown", library calls it "Menu"), ask: "Did you mean `{libraryName}`?"
2. If the library doesn't have the component, fall back to `specs/{component}.spec.json` (Kido spec) and warn: "No `{component}` in {library}. Using Kido spec for structure. Tokens from DESIGN.md still apply."
3. If neither exists, stop. Ask: "No library or Kido spec for `{component}`. Do you want me to author a Kido spec first (`/ds-spec-authoring`)?"

---

## Critical Rules

**Library structure wins over Kido structure.** Kido specs are fallback only. If Chakra uses `variant` and `colorScheme` as axes and Kido uses `type`, use Chakra's.

**DESIGN.md tokens win over library default tokens.** Chakra's default `gray.500` is overridden by the semantic token in DESIGN.md that maps to "border subtle". The library supplies the structure; DESIGN.md supplies the values.

**REQUIREMENTS.md overrides both.** If REQUIREMENTS says "dark mode only", ignore library light variants. If it says "prefix `pz_`", apply to every node name. Explicit job rules beat library conventions.

**Preserve the client's font.** Do not force Inter. Resolve via DESIGN.md `typography.*.fontFamily`. Apply the font resolution logic from `skills/ds-generate.md` Step 2 (exact match → category fallback → note substitution).

**Stubs stay pink.** Unresolved values get `#FF0066`. Never invent a value to avoid a stub.

**The validator is advisory, not a gate.** Errors in the report do not stop completion. The designer decides the follow-up.

**DESIGN.md and REQUIREMENTS.md are per-job.** Do not commit them. Do not reuse across clients.
