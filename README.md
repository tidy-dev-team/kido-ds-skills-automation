# DS Component Automation

> A methodology for expanding partial client component designs into production-ready Figma component sets, using Kido DS rules as the foundation.
> Built by Tidy Dev. Powered by Claude + Figma MCP.

---

## The Problem

A client hands you a button in two states. Production-ready means 54 variants minimum — every Type, every Size, every interactive State. Today that expansion is manual, slow, and inconsistent.

This system encodes Kido DS expertise into compact specs. Claude reads the client's design, maps their brand values to Kido token slots, and builds the full component set in Figma automatically. The designer reviews and polishes — not the process.

---

## How It Works

Two skills. Two audiences. One pipeline.

```
[DS team — one-time per component]      [Designer — per client project]
───────────────────────────────         ────────────────────────────────
/ds-spec-authoring                      /ds-generate
      │                                        │
      ▼                                        ▼
specs/{component}.spec.json    →    Component set in Figma
specs/{component}.spec.notes.md
```

---

## Spec Authoring Workflow (DS Team — One-Time Per Component)

```
1. Open Claude Code in this project directory

2. Provide the component name and Figma URL:
   /ds-spec-authoring
   "Spec out the Toggle component: [Kido DS Figma URL]"

3. Claude extracts from Figma via MCP:
   variant axes, layer hierarchy, tokens, spacing, typography

4. Claude asks a few lightweight questions:
   - Which combinations are required vs optional?
   - Any known WCAG deviations?
   - Any non-obvious design decisions?

5. Claude writes specs/toggle.spec.json + specs/toggle.spec.notes.md
   and updates specs/_index.json

6. Review the summary and confirm:
   "Toggle: 2 types × 3 sizes × 5 states = 30 default variants. 18 tokens. Ready."
```

**Typical time: 30–60 minutes** for a new component. Health check or update: 10–15 minutes.

---

## Designer Workflow (Per Client Project)

```
1. Open Claude Code in this project directory

2. Share the client's Figma frame:
   /ds-generate
   "Generate this button: [Figma URL or attach screenshot]"

3. Claude classifies, extracts brand values, and confirms:
   "Got it — Button. Primary color #E53E3E detected.
    Generating 54 variants (inverse=no set).
    Ready."

4. Claude builds the component set directly in Figma via MCP

5. Review in Figma. Stubs appear in bright pink — fill them in if needed.
```

**Typical time: under 15 minutes** from frame to generated component set.

**Need dark-background variants?** Ask explicitly: "Generate with inverse variants too." Adds 54 more variants (108 total).

---

## Spec Library

```
specs/
  _index.json                    ← component index — Claude reads this first
  button.spec.json               ← compact token data and variant axes
  button.spec.notes.md           ← design rationale, derivation patterns, history
```

| Component | Status | Default variants |
|-----------|--------|-----------------|
| Button | Done | 54 (108 with inverse) |
| Input | Planned | — |
| Toggle | Planned | — |
| Checkbox | Planned | — |

Each spec has two files:
- **`.spec.json`** — compact, LLM-optimized. Token values, variant axes, sizing, accessibility requirements.
- **`.spec.notes.md`** — human-readable. Design decisions, token derivation patterns for client brands, known gaps, version history.

---

## Working Directory

Session artifacts are saved locally to `working/` (gitignored — not committed to the repo):

```
working/
  button-2026-04-05/
    token-map.json        ← resolved tokens used for this generation
    resolved-stubs.json   ← stub values provided by the designer (if any)
```

This lets you pick up where you left off if a session spans multiple days.

---

## Stubs

When a value can't be resolved from the client input, Claude generates a stub:
- **Fill:** bright pink (`#FF0066`) — easy to spot in Figma
- **Label:** describes what's needed, e.g. "NEEDS_VALUE: inverse primary color"

Inverse variants are almost always stubbed — clients rarely provide dark-background designs upfront. This is expected, not a failure.

---

## Repository Structure

```
specs/
  _index.json
  {component}.spec.json
  {component}.spec.notes.md

skills/
  ds-spec-authoring.md     ← full instructions for spec authoring
  ds-generate.md           ← full instructions for component generation

working/                   ← local session artifacts (gitignored)

.claude/
  commands/
    ds-spec-authoring.md   ← /ds-spec-authoring slash command
    ds-generate.md         ← /ds-generate slash command

CLAUDE.md                  ← agent instructions (auto-loaded by Claude Code)
README.md                  ← this file
```

---

## Requirements

| Requirement | Purpose |
|-------------|---------|
| Claude Code | Runs the workflow; auto-loads CLAUDE.md |
| Figma MCP connected | Reads Kido DS for spec authoring; writes components for generation |

---

## Design Principles

**Kido rules are the default.** The designer's input provides brand values only — primary color, border radius, font if non-standard. States, sizes, spacing, and derivation come from the spec automatically.

**Compact generation by default.** `inverse=no` variants only unless dark-background support is explicitly needed. Avoids unnecessary stubs on every project.

**Stubs over blocking.** Generate with gaps visible. Polish in Figma after, not before.

**Specs are data.** Token names are self-documenting. Claude reasons about derivation — the spec doesn't need to explain it. Smaller spec = faster reads = less context consumed.

**Client styling preserved through token mapping.** Kido provides structure. The client's input provides values. The output looks like the client's brand, not Kido.

---

## Limitations

- **No spec = no full automation.** Novel components need `/ds-spec-authoring` first. One-time cost.
- **Inverse variants are almost always stubbed.** Expected behavior — clients rarely provide dark-background designs.
- **Figma MCP required.** Screenshot input works but extracts less precise token values. Figma URL is strongly preferred.
- **Kido drift.** Run `/ds-spec-authoring check [component]` if Kido DS has changed. Stale specs produce off-brand output.

---

*The goal: generating the tenth client button takes five minutes, not five hours.*
