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

### Added
- `skills/templates/CHANGES.template.md` — template for per-project change logs with the two-category split. (#6)

