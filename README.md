# DS Component Automation

> A methodology for expanding partial client component designs into production-ready Figma component sets, and syncing the polished result back to code — using Kido DS rules as the foundation.
> Built by Tidy Dev. Powered by Claude + Figma MCP.

---

## The Problem

A client hands you a button in two states. Production-ready means 54 variants minimum — every Type, every Size, every interactive State. Today that expansion is manual, slow, and inconsistent.

And once the DS specialist polishes the component in Figma, someone still has to update the codebase to match. That's another manual step, another opportunity for drift.

This system closes both gaps automatically.

---

## How It Works

Three skills. A closed loop.

```
[DS team — one-time]         [Designer — per project]        [After DS polish]
────────────────────         ────────────────────────        ──────────────────
/ds-spec-authoring           /ds-generate                    /ds-push
        │                           │                               │
        ▼                           ▼                               ▼
  specs/*.spec.json    →    Component set in Figma    →    GitHub PR
                             (54 variants, branded)         (CSS vars + config updated)
                                     │
                               DS specialist
                               reviews & polishes
```

| Skill | Who | When | Output |
|---|---|---|---|
| `/ds-spec-authoring` | DS team | One-time per component type | `specs/{component}.spec.json` |
| `/ds-generate` | Designer | Per client project | Component set in Figma |
| `/ds-push` | DS specialist | After Figma polish | GitHub PR with updated token values |

---

## Skill Workflows

### `/ds-spec-authoring` — DS Team, One-Time Per Component

```
1. Run: /ds-spec-authoring
   "Spec out the Toggle component: [Kido DS Figma URL]"

2. Claude reads the component from Kido DS via Figma MCP:
   variant axes, layer hierarchy, tokens, spacing, typography

3. Claude asks a few lightweight questions:
   - Which combinations are required vs optional?
   - Any known WCAG deviations?
   - Any non-obvious design decisions?

4. Claude writes:
   specs/toggle.spec.json        ← compact token data, variant axes
   specs/toggle.spec.notes.md    ← design rationale, version history
   specs/_index.json             ← updated

5. Review the summary:
   "Toggle: 2 types × 3 sizes × 5 states = 30 default variants. 18 tokens. Ready."
```

**Typical time:** 30–60 minutes for a new component. Health check or update: 10–15 minutes.

---

### `/ds-generate` — Designer, Per Client Project

```
1. Run: /ds-generate
   "Generate this button: [Figma URL / screenshot / site URL / GitHub repo]"

2. Claude identifies the target component, extracts brand values from the input:
   - Figma URL → get_design_context
   - Screenshot/description → visual analysis
   - Live site URL → WebFetch (CSS custom properties, computed styles)
   - GitHub repo → gh api (index.css, tailwind.config, component source)

3. Claude maps client values to Kido token slots:
   primary color, border radius, font, weight, letter spacing, etc.

4. Claude builds the component set directly in Figma via MCP:
   54 variants (inverse=no): 3 types × 3 sizes × 6 states

5. Review in Figma. Stubs appear in bright pink — fill them in if needed.
   DS specialist polishes to production-ready.
```

**Typical time:** under 15 minutes from input to generated component set.

**Need dark-background variants?** Ask: "Generate with inverse variants too." Adds 54 more (108 total).

---

### `/ds-push` — DS Specialist, After Figma Polish

```
1. Polish the generated component in Figma (adjust any tokens, fix stubs, refine details)

2. Run: /ds-push
   "Push the polished button to GitHub: [Figma node URL] [GitHub repo URL]"

3. Claude reads the polished component set via Figma MCP:
   extracts resolved token values from key variants
   (contained idle → primary, contained hover → hover bg, etc.)

4. Claude discovers the repo structure and locates:
   src/index.css (or globals.css)    ← CSS custom properties
   tailwind.config.ts                ← theme colors, custom tokens
   src/components/ui/button.tsx      ← component source (if structural changes)

5. Claude opens a GitHub PR with:
   - Updated CSS variable values
   - Updated Tailwind config color tokens
   - Updated component class names (if radius, weight, or border-width changed)
   - PR description with before/after token diff

6. Merge the PR. Lovable (or your deploy pipeline) picks it up automatically.
```

**Typical time:** under 5 minutes from polished Figma to open PR.

---

## Input Sources for `/ds-generate`

The designer can provide brand context in any of these formats:

| Input type | How Claude reads it |
|---|---|
| Figma URL | `get_design_context` via Figma MCP |
| Screenshot | Visual analysis |
| Description | Text extraction |
| Live site URL | `WebFetch` — CSS custom properties, computed styles |
| GitHub repo URL | `gh api` — `index.css`, `tailwind.config.ts`, component source |

Any frame, page, or component carrying the client's visual style is valid — not just the target component itself.

---

## Spec Library

```
specs/
  _index.json                    ← component index — Claude reads this first
  button.spec.json               ← compact token data and variant axes
  button.spec.notes.md           ← design rationale, derivation patterns, history
```

| Component | Tier | Status | Default variants |
|-----------|------|--------|-----------------|
| Button | 1 | Done | 54 (108 with inverse) |
| Input | 1 | Planned | — |
| Toggle | 1 | Planned | — |
| Checkbox | 1 | Planned | — |
| Select | 2 | Planned | — |
| Card | 2 | Planned | — |
| Tab Bar | 2 | Planned | — |
| Badge | 2 | Planned | — |

Each spec has two files:
- **`.spec.json`** — compact, LLM-optimized. Token values, variant axes, sizing, accessibility requirements.
- **`.spec.notes.md`** — human-readable. Design decisions, token derivation patterns, known gaps, version history.

---

## Working Directory

Session artifacts are saved locally to `working/` (gitignored):

```
working/
  button-2026-04-07/
    token-map.json          ← resolved tokens used for generation
    resolved-stubs.json     ← stub values provided by the designer (if any)
    push-summary.json       ← before/after diff + PR URL (written by ds-push)
```

This lets you pick up where you left off if a session spans multiple days, and gives `ds-push` a baseline to diff against.

---

## Stubs

When a value can't be resolved from the client input, Claude generates a stub:
- **Fill:** bright pink (`#FF0066`) — easy to spot in Figma
- **Label:** describes what's needed, e.g. "NEEDS_VALUE: inverse primary color"

Stubs are expected — not a failure. The DS specialist fills them in during the polish pass. `ds-push` then picks up the resolved values and updates the code.

---

## Repository Structure

```
specs/
  _index.json
  {component}.spec.json
  {component}.spec.notes.md

skills/
  ds-spec-authoring.md     ← /ds-spec-authoring instructions
  ds-generate.md           ← /ds-generate instructions
  ds-push.md               ← /ds-push instructions

working/                   ← local session artifacts (gitignored)

.claude/
  commands/
    ds-spec-authoring.md   ← /ds-spec-authoring slash command
    ds-generate.md         ← /ds-generate slash command
    ds-push.md             ← /ds-push slash command

CLAUDE.md                  ← agent instructions (auto-loaded by Claude Code)
README.md                  ← this file
```

---

## Requirements

| Requirement | Purpose |
|-------------|---------|
| Claude Code | Runs the workflow; auto-loads CLAUDE.md |
| Figma MCP connected | Reads Kido DS for spec authoring; reads/writes Figma for generate and push |
| `gh` CLI authenticated | Used by ds-generate (repo reads) and ds-push (repo writes + PR) |

---

## Design Principles

**Kido rules are the default.** The designer's input provides brand values only — primary color, border radius, font if non-standard. States, sizes, spacing, and derivation come from the spec automatically.

**Closed loop.** Design changes in Figma propagate to code via `ds-push`. Code changes are sourced for future generation via `ds-generate`. No manual translation step either direction.

**Compact generation by default.** `inverse=no` variants only unless dark-background support is explicitly needed.

**Stubs over blocking.** Generate with gaps visible. Polish in Figma after, not before. Push after polish.

**Specs are data.** Token names are self-documenting. Claude reasons about derivation — the spec doesn't need to explain it.

**Client styling preserved through token mapping.** Kido provides structure. The client's input provides values. The output looks like the client's brand, not Kido.

---

## Limitations

- **No spec = no full automation.** Novel components need `/ds-spec-authoring` first. One-time cost.
- **Inverse variants are almost always stubbed.** Expected — clients rarely provide dark-background designs upfront.
- **Figma MCP required.** Screenshot input works but extracts less precise values. Figma URL is strongly preferred.
- **`gh` CLI required for ds-push.** Must be authenticated (`gh auth login`) before running `/ds-push`.
- **Kido drift.** Run `/ds-spec-authoring check [component]` if Kido DS has changed. Stale specs produce off-brand output.

---

*The goal: generating the tenth client button takes five minutes. Keeping code and Figma in sync takes five more.*
