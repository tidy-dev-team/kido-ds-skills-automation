# DS Component Automation — Agent Instructions

This project encodes Tidy Dev's design system expertise into a pipeline for expanding partial
client component designs into production-ready Figma component sets using Kido DS rules.

**Before acting on any request in this domain, read the relevant skill file from `skills/`.
The skill file is the authoritative instruction set — follow it exactly.**

---

## What This System Does

Designers deliver partial client component concepts — a button in one or two states.
A production-ready component set requires 54–108 variants across Type × Size × State × Inverse.
This system bridges that gap automatically, using Kido DS best practices as the foundation.

```
[DS team — one-time]    [Designer — per project]    [After DS polish]          [Documentation]
────────────────────    ────────────────────────    ─────────────────          ───────────────
ds-spec-authoring       Client input                ds-push                    ds-storybook
      │                 (Figma/screenshot/               │                           │
      ▼                  site/repo)                      ▼                           ▼
specs/*.spec.json  →  ds-generate  →  Figma  →  DS review  →  GitHub PR  →  stories PR
                                      component   & polish     (tokens)      (CSF3 file)
```

---

## The Four Skills

### `ds-spec-authoring`
**File:** `skills/ds-spec-authoring.md`
**When:** A new component is added to Kido DS, an existing spec needs updating, or you want to run a spec health check.
**Who:** DS team. Not a per-project task.
**Input:** Component name + Kido DS Figma URL.
**Output:** `specs/{component}.spec.json` + `specs/{component}.spec.notes.md` + updated `specs/_index.json`
**Read the skill file before running.**

### `ds-generate`
**File:** `skills/ds-generate.md`
**When:** A designer provides a client Figma frame, screenshot, description, live site URL, or code repository for a component to generate.
**Who:** Designer, per client project.
**Input:** Client component input (Figma URL, screenshot, description, live site URL, or GitHub repo).
**Output:** Component set built directly in Figma via MCP.
**Philosophy:** Classify → extract brand values → map tokens → execute. No intermediate files. No approval gates. Stubs for anything unresolved.
**Read the skill file before running.**

### `ds-push`
**File:** `skills/ds-push.md`
**When:** A DS specialist has polished the generated Figma component and the code needs to be updated to match.
**Who:** DS specialist or designer, after Figma review.
**Input:** Polished Figma component set URL + GitHub repo URL.
**Output:** GitHub PR with updated CSS variables, Tailwind config, and/or component source.
**Philosophy:** Read polished Figma → extract resolved token values → diff → surgically update repo files → open PR. Never rewrites structure — only values.
**Read the skill file before running.**

### `ds-storybook`
**File:** `skills/ds-storybook.md`
**When:** A component is production-ready (post-polish) and needs Storybook documentation.
**Who:** DS specialist or developer.
**Input:** Component name + GitHub repo URL. Reads Kido DS spec for variant axes and repo source for the prop interface.
**Output:** GitHub PR with a CSF3 `{component}.stories.tsx` file. Includes Storybook setup instructions in the PR body if Storybook is not yet installed.
**Philosophy:** Map Figma variant axes → React props → named stories. Axes that have no prop equivalent (hover, pressed, focus) become pseudo-state stories. 8–12 named stories is the target — not 54.
**Read the skill file before running.**

---

## Spec Library

Always read `specs/_index.json` first — it is the entry point for finding the right spec.
Spec files: `specs/{component}.spec.json` (compact, LLM-optimized data)
Notes files: `specs/{component}.spec.notes.md` (human-readable rationale and history)

Component tiers (implementation priority):
- **Tier 1:** Button, Input, Toggle, Checkbox
- **Tier 2:** Select, Card, Tab Bar, Badge
- **Tier 3:** Modal, Navigation, Table, Form

When no spec exists for the component being worked on, follow the "No Matching Spec" protocol
in `skills/ds-generate.md`.

---

## Figma MCP

Used for spec authoring (reading Kido DS) and generation (writing components into Figma).

| Tool | Used in | Purpose |
|------|---------|---------|
| `get_metadata` | spec-authoring | Locate component, get variant IDs |
| `get_design_context` | spec-authoring, generate | Layer structure, variants, tokens |
| `get_variable_defs` | spec-authoring | Design token definitions |
| `get_screenshot` | generate | Visual input when no frame URL |
| `figma_execute` / layer tools | generate | Write component variants into Figma |

---

## File Locations

```
specs/
  _index.json                      ← read this first when looking up a component
  {component}.spec.json            ← compact token data, variant axes, sizing
  {component}.spec.notes.md        ← design rationale, derivation notes, history

skills/
  ds-spec-authoring.md
  ds-generate.md
  ds-push.md
  ds-storybook.md

working/                           ← local session artifacts (gitignored)
  {component}-{YYYY-MM-DD}/
    token-map.json
    resolved-stubs.json
    push-summary.json              ← before/after diff + PR URL (written by ds-push)

.claude/
  commands/
    ds-spec-authoring.md           ← /ds-spec-authoring
    ds-generate.md                 ← /ds-generate
    ds-push.md                     ← /ds-push
    ds-storybook.md                ← /ds-storybook

CLAUDE.md                          ← this file (auto-loaded by Claude Code)
README.md                          ← human-facing overview
```

---

## Critical Rules

**Read the skill file first.** Skill files are the authoritative instruction set. CLAUDE.md summaries may lag behind skill updates.

**Never fabricate token values.** Stub as `NEEDS_VALUE`. A plan with visible stubs is better than one with invented values.

**Stubs are valid output.** The designer polishes them in Figma after generation. Never block generation waiting for a value that can be stubbed.

**Default to compact generation.** Generate `inverse=no` variants only unless the designer explicitly requests dark-background support.

**Client styling is preserved through token mapping.** Kido provides structure. The client's input provides brand values. The output should look like the client's brand, not Kido.

**Specs are data, not instructions.** Token names are self-documenting — Claude reasons about derivation. The spec does not need to explain how to derive states.

---

## When Things Go Wrong

**Spec and Kido have diverged:** Run `/ds-spec-authoring check [component]`. Diff, update changed fields, bump version. Do not rewrite from scratch.

**Classification confidence is low:** Ask to confirm the component type before proceeding. Wrong classification = wrong spec = wrong output.

**No spec for this component:** Follow "No Matching Spec" in `skills/ds-generate.md`. Do not guess.

**Generation plan has stubs:** Expected. Stubs appear as bright pink fills in Figma.
