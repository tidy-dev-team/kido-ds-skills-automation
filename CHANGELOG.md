# Changelog

All notable changes to skills, specs, templates, requirements, and quality standards in this repo.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Dates are `YYYY-MM-DD`.

## When to add an entry

Add an entry whenever a change touches any of:

- `skills/**` — skill files and templates
- `specs/**` — Kido component specs and library mappings
- `CLAUDE.md`, `LANGUAGE.md`, `README.md`
- `.claude/commands/**`
- Anything else that changes how a designer or downstream agent uses this repo

Skip entries for: `working/**`, internal logs, formatting-only edits, and changes scoped to a single client project that should not travel to main.

## Entry format

Group entries under a dated section. Use these categories: **Added**, **Changed**, **Fixed**, **Removed**, **Deprecated**, **Security**. Each line should name the affected file(s) and explain the change in one sentence — link related issues/PRs where useful.

```
## [Unreleased]

### Changed
- `skills/ds-generate.md` — added pre-flight checklist and self-audit step (#1)
```

---

## [Unreleased]

### Changed
- `skills/ds-generate.md` — added Step 3 (Pre-flight Checklist & Execution Plan) as a hard gate before generation, and Step 5 (Self-Audit) that reads the produced component set back from Figma and ticks the plan; renumbered prior Step 3 → 4 and Step 4 → 6. Pre-flight derives from the spec; component-property type inference is heuristic until spec schema 0.2 lands (#8). (#1)
- `skills/ds-generate.md` — documented the auto-layout sizing pitfall in Step 4 (`resize()` resets both axes to FIXED; assigning `layoutMode` resets both to AUTO) and added a sizing-mode regression check to the Step 5 self-audit + the Step 3 pre-flight derivation table. Fixes Button variants shipping at fixed minimum width instead of hugging content. (#2)
- `skills/ds-generate.md`, `skills/ds-build.md`, `CLAUDE.md` — added Step 0 hook reading `DESIGN-SYSTEM.md` from the repo root as authoritative cross-project rules, and codified the per-project `working/{project}/CHANGES.md` two-category split (Generic improvements vs Project-specific) as a standing rule. (#6)
- `skills/ds-generate.md` — defined the icon-missing protocol: search the target file for an icon library; if absent, insert bright-pink `#FF0066` placeholder frames per icon slot (never draw inline); self-audit verifies every icon slot is populated; final report surfaces missing-icon gaps separately from token stubs. (#3)
- `specs/*.spec.json` — bumped to schema 0.2: added structured `anatomy.component_properties` (TEXT / BOOLEAN / INSTANCE_SWAP) to button, button-icon, button-text, banner-contained, banner-outlined; explicit empty array on checkbox-icon (variant-driven). Removed prose property descriptions from `anatomy.notes`. Banner specs migrated their pre-existing top-level `component_properties` into the new shape. Bumped `_index.json` schema_version 0.1 → 0.2. (#8)
- `skills/ds-spec-authoring.md` — documented the 0.2 schema, the BOOLEAN+INSTANCE_SWAP pairing convention for optional icon slots, the always-visible single-INSTANCE_SWAP convention for required icons, and the empty-array convention for variant-driven components. (#8)
- `skills/ds-generate.md` — Step 3 derivation now consumes `anatomy.component_properties` directly; removed the heuristic bridge added in #1. Pre-0.2 specs surface a `NEEDS_SPEC` plan item instead of inferring property types. (#8)
- `skills/ds-generate.md`, `skills/ds-push.md`, `skills/ds-storybook.md` — added explicit Intake gates so each skill prompts the designer for missing required inputs (component name + client input / Figma URL + repo URL / component name + source) instead of assuming args are present. Required so the `ds-guide` thin router can safely invoke targets with empty args. (#9)
- `skills/ds-extract-design.md` — reconciled DTCG body token paths with the design.md spec front matter: `color` → `colors`, `radius` → `rounded`, `component` → `components`; updated all `{...}` references to match; flattened the Layout JSON so `spacing` sits at the root (mirroring the front matter `spacing:` key). Front matter and body now describe the same tree; the file lints cleanly against `@google/design.md`.
- `skills/ds-extract-design.md` — documented `borderColor` and `borderWidth` as Kido extension component properties (warning-only when linted by `@google/design.md`, fully accepted by `/ds-build`). Added an explicit "Allowed properties (spec)" vs "Kido extension properties" note above the Components JSON.
- `skills/ds-extract-design.md` — clarified the design.md compatibility claim. Front matter conforms to the spec and lints cleanly; the DTCG body is explicitly a Kido extension consumed by `/ds-build`, not part of the spec. Added a "Compatibility claim" callout warning that `npx @google/design.md export --format dtcg` operates on the front matter only and will not roundtrip the curated body. Updated the skill's frontmatter `description` to match.
- `skills/ds-extract-design.md`, `skills/ds-build.md`, `skills/ds-guide.md`, `skills/templates/REQUIREMENTS.template.md`, `skills/templates/QUALITY_STANDARDS.md`, `CLAUDE.md`, `LANGUAGE.md`, `README.md` — **split DESIGN.md into two sibling files: `tokens.json` + `DESIGN.md`.** Aligns with the google-labs-code/design.md spec, which treats the markdown body as prose rationale. `tokens.json` is the full W3C DTCG token tree (machine source of truth, consumed by `/ds-build`); `DESIGN.md` keeps the spec-compliant front matter plus a prose body (overview, palette philosophy, typography intent, do's/don'ts, gaps — no JSON blocks). The two files share identical top-level keys (`colors`, `typography`, `spacing`, `rounded`, `components`) and always travel together. `/ds-build` now reads `tokens.json` for token values; `DESIGN.md` is consulted for rationale only. Added a new `tokens.json` glossary entry in `LANGUAGE.md`.
- `.claude/commands/ds-extract-design.md`, `.claude/commands/ds-build.md`, `documentation/ARCHITECTURE.md` — caught up with the split: slash-command descriptions now mention both output files; the ARCHITECTURE diagram and per-project artifact list show `tokens.json` alongside `DESIGN.md`. Also tightened the `ds-guide` prerequisite-list line in `LANGUAGE.md` to include `tokens.json`.
- `skills/ds-guide.md`, `.claude/commands/ds-guide.md`, `LANGUAGE.md` — extended the wizard to give `ds-extract-design` and `ds-spec-authoring` direct clickable routes. Q1 is now four action buckets (Generate / Extract tokens / Author Kido spec / Post-polish); Generate triggers Q2 to disambiguate A vs B, and Post-polish triggers Q2 to disambiguate push vs stories. Every skill is now reachable in at most two clicks. The "Other" free-text path still maps off-script phrasing to the right route.

### Added
- `skills/templates/CHANGES.template.md` — template for per-project change logs with the two-category split. (#6)
- `skills/ds-guide.md` + `.claude/commands/ds-guide.md` — new wizard skill and slash command. Single `AskUserQuestion` (4 visible routes + auto Other for `ds-extract-design` / `ds-spec-authoring`); inline disambiguation in option descriptions for Workflow A vs B. Pure router: invokes the target skill with no args and lets it run its own intake — no upfront input collection, no prerequisite checks, no session state. Direct invocation of every other skill remains unchanged. Updated `CLAUDE.md`, `LANGUAGE.md`, `README.md`, `documentation/ARCHITECTURE.md` to reflect the seven-skill list. (#9)
- `DESIGN-SYSTEM.md` (repo root) — authoritative cross-project rules: token decision tree (`fg`/`bg`/`border` × `static`/`interactive` × `brand`/`muted`/`base`/`secondary`), disabled family rule, modes-as-brands, Bootstrap state-name mapping, **State Ownership Rule** with the radio button as canonical reference, and the interactive state token mapping (stroke-primary vs fill-primary vs bg-zone). Project-specific overrides (variable IDs, active modes) live in a stub at the file's tail. Read at Step 0 by every skill via the hook added in #6. (#5)
- `skills/ds-generate.md` — documented the **Variable Binding** mechanism in Step 4: read local Figma variable collections, build a token-name → variable map, and `setBoundVariable` at write-time instead of writing raw values. Field-name reference table covers fills, strokes, corner radius, padding, item spacing. Self-audit (Step 5) extended to flag any slot whose spec token name matches a collection entry but came out as a raw hex. (#4)
- `skills/ds-build.md` — added the same Variable Binding mechanism (points at `ds-generate` as the canonical procedure) with two paths: **at-write-time** binding when generation has the token map, and a **post-generation binding pass** for the library-structure case where slots were written as raw values. Validator extended with category **2a (Unbound-but-bindable slots)** that flags raw values when a matching variable existed; total validator categories now 10. Updated `LANGUAGE.md` to reflect the new count. (#4)

