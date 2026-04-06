# DS Component Automation — Agent Instructions

This project encodes Tidy Dev's design system expertise into a pipeline for expanding partial
client component designs into production-ready Figma component sets using Kido DS rules.

**Before acting on any request in this domain, read the relevant skill file from `skills/`.
The skill file is the authoritative instruction set — follow it exactly.**

---

## What This System Does

Designers deliver partial client component concepts — a button in one or two states.
A production-ready component set for that button requires 108 variants across Type × Size × State × Inverse.
This system bridges that gap automatically, using Kido DS best practices as the foundation.

```
Kido DS (Figma)
      │
      ▼
[ds-spec-authoring]  →  specs/{component}.spec.json
                                  │
              Client Figma frame ──▼
                    [ds-component-analysis]  →  gap-report.json
                                                      │
                                                      ▼
                                        [ds-generation-planning]  →  Claude executes via Figma MCP
```

---

## The Three Skills

### `ds-spec-authoring`
**File:** `skills/ds-spec-authoring.md`
**When:** A new component is added to Kido DS, or an existing spec needs updating.
**Who:** DS team or designer familiar with Kido internals. Not a per-project task.
**Input:** Component name + Kido DS Figma URL.
**Output:** `specs/{component-name}.spec.json` + updated `specs/_index.json`
**Read the skill file before running.**

### `ds-component-analysis`
**File:** `skills/ds-component-analysis.md`
**When:** A designer provides a client Figma frame, screenshot, or description of a component.
**Who:** Designer, per client project.
**Input:** Client component input + spec library (`specs/_index.json` → relevant spec file).
**Output:** `gap-report.json`
**Philosophy:** Generate-first. Kido rules cover everything structural. Designer input provides brand values only. Stub what can't be resolved — do not ask about it.
**Read the skill file before running.**

### `ds-generation-planning`
**File:** `skills/ds-generation-planning.md`
**When:** A gap report exists.
**Who:** Designer (or runs automatically after analysis in the same session).
**Input:** `gap-report.json`
**Output:** `generation-plan.json` → Claude executes in Figma via MCP
**Philosophy:** Fully automatic. No questions. Stubs for anything unresolved.
**Read the skill file before running.**

---

## Spec Library

Specs live in `specs/`. Always read `specs/_index.json` first — it is the entry point for finding the right spec.

Current schema version: **0.1** (defined in `skills/ds-spec-authoring.md`)

Component tiers (implementation priority):
- **Tier 1** (start here): Button, Input, Toggle, Checkbox
- **Tier 2**: Select, Card, Tab Bar, Badge
- **Tier 3** (later): Modal, Navigation, Table, Form

When no spec exists for the component being worked on, follow the "No Matching Spec" protocol
in `skills/ds-component-analysis.md`: find the closest spec, attempt composition, or ask — in that order.

---

## Figma MCP

Used for both spec authoring (reading Kido DS) and generation (writing components into Figma).

| Tool | Used in | Purpose |
|------|---------|---------|
| `get_metadata` | spec-authoring | Locate component in file |
| `get_design_context` | spec-authoring, analysis | Layer structure, variants, tokens |
| `get_variable_defs` | spec-authoring | Design token definitions |
| `get_screenshot` | analysis | Visual input when no frame URL |
| `figma_execute` / `figma_create_child` etc. | generation | Write component variants into Figma |

If the MCP is unavailable during spec authoring: ask the user to paste layer structure manually or accept a screenshot. Document the fallback in the spec's `notes` field.

---

## File Locations

```
specs/
  _index.json                ← read this first when looking up a component
  button.spec.json
  {component}.spec.json

skills/
  ds-spec-authoring.md
  ds-component-analysis.md
  ds-generation-planning.md

CLAUDE.md                    ← this file (auto-loaded by Claude Code)
README.md                    ← human-facing overview
```

---

## Critical Rules

**Read the skill file first.** Every skill file contains the full instruction set for that stage. Do not improvise from memory — the skills evolve and CLAUDE.md summaries may lag behind.

**Never fabricate token values.** If a value cannot be determined from the input or a derivation rule, stub it as `NEEDS_VALUE`. A plan with visible stubs is better than one with invented values.

**Stubs are valid output.** Stubs make gaps visible in Figma. The designer polishes them after generation. Never block generation waiting for a value that can be stubbed.

**Client styling is preserved through token mapping.** Kido provides structure. The client's input provides values. The token mapping step is what makes the output look like the client's brand, not Kido.

**Specs are configuration.** When a variant axis changes (e.g. adding XL size), that is a spec update. When component behavior changes (e.g. Loading state logic), that is a skill update. Keep these concerns separate.

---

## When Things Go Wrong

**Spec and Kido have diverged:** Re-run `ds-spec-authoring` for the changed component. Diff against the existing spec, update only changed fields, bump the version. Do not rewrite from scratch.

**Classification confidence is low:** Ask the user to confirm the component type before running analysis. A wrong classification produces a useless gap report.

**Spec missing for this component type:** Follow the "No Matching Spec" protocol in `skills/ds-component-analysis.md`. Do not silently fall back to guessing.

**Generation plan has stubs:** Expected and acceptable. Stubs appear as bright pink fills in Figma so they are easy to find and fill in manually.
