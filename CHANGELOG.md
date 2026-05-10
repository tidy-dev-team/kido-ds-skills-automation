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

