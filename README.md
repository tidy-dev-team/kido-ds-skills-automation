# DS Component Automation

> A methodology for expanding partial client component designs into production-ready Figma component sets, using Kido DS rules as the foundation.
> Built by Tidy Dev. Powered by Claude + Figma MCP.

---

## The Problem

A client hands you a button in two states. Production-ready means 108 variants — every Type, every Size, every State, every Inverse variant. Today that expansion is manual, slow, and inconsistent.

This system encodes Kido DS expertise into structured specs. Claude reads a client's design, maps their brand values to Kido token slots, and generates the full component set automatically. The designer reviews and polishes the result — not the process.

---

## How It Works

Three skills, three artifacts, one pipeline.

```
[Kido DS — one-time setup]        [Per client project]
─────────────────────────         ────────────────────────────────────────
ds-spec-authoring                 Designer's Figma frame
      │                                      │
      ▼                                      ▼
specs/{component}.spec.json  →  ds-component-analysis  →  gap-report.json
                                                                  │
                                                                  ▼
                                                    ds-generation-planning
                                                                  │
                                                                  ▼
                                                       generation-plan.json
                                                                  │
                                                                  ▼
                                                     Claude executes via Figma MCP
```

---

## The Three Skills

### `ds-spec-authoring` — DS team only

**When:** A new component is added to Kido DS, or an existing component changes significantly.
**Who:** DS team or designer familiar with Kido internals.
**Input:** Component name + Kido DS Figma URL.
**Output:** `specs/{component}.spec.json`

Extracts the component's full structure from Figma (via MCP) and encodes:
- All variant axes and their values
- Required vs. optional combinations
- Design tokens with resolved values
- Derivation rules (how to build every missing state/size from a base)
- Accessibility requirements
- The definition of "production-ready" for this component type

This is infrastructure. Done once, used on every project that needs this component.

---

### `ds-component-analysis` — Designer, per project

**When:** A designer has a partial client component and wants to generate the full set.
**Who:** Designer, in Claude app or Claude Code.
**Input:** A Figma frame URL, screenshot, or description of the client's component.
**Output:** `gap-report.json`

Claude reads the input, identifies the component type, extracts the client's brand values (primary color, border radius, font if non-standard), and maps them to Kido token slots. All structural decisions — states, sizes, spacing, derivation — are handled by the spec automatically.

**What Claude asks the designer:**
- "This looks like a Button — correct?" (only if classification confidence is below 85%)
- The specific brand value (only if it's genuinely not visible in the input and can't be stubbed)

Everything else is automatic. Claude does not ask about states, sizes, spacing, derivation logic, or anything Kido already defines.

Variants that can't be fully resolved (e.g. inverse brand colors not in the input) become **stubs** — the generation proceeds anyway, and stubs are easy to find and fill in Figma after.

---

### `ds-generation-planning` — Designer, immediately after analysis

**When:** A gap report exists.
**Who:** Designer (or runs automatically after analysis).
**Input:** `gap-report.json`
**Output:** `generation-plan.json`

Fully automatic. Takes the resolved token map and applies spec rules to produce ordered plugin instructions. No questions, no approvals. Stubs for anything unresolved.

---

## Spec Authoring Workflow (DS Team — One-Time Per Component)

```
1. Open Claude app or Claude Code in this project directory

2. Provide the component name and Figma URL:
   "Spec out the Toggle component: [Kido DS Figma URL]"

3. Claude extracts structure from Figma via MCP:
   variant axes, layer hierarchy, tokens, spacing, typography

4. Claude walks you through enrichment questions:
   - Which variants are required vs. optional?
   - Derivation rules: how is Hover derived from Idle? How is SM derived from MD?
   - Accessibility: touch targets, focus ring specs, contrast requirements
   - What must never be auto-derived and always requires designer input?

5. Claude writes specs/{component}.spec.json and updates specs/_index.json

6. Review the summary:
   "Toggle spec defines 2 Types × 3 Sizes × 5 States = 30 variants.
    4 derivation rules encoded. 1 value always requires designer input."
   Confirm or request corrections.
```

**Typical time: 30–60 minutes** for a new component. Update-only is 10–15 minutes.

---

## Designer Workflow (Per Project)

```
1. Open Claude app or Claude Code in this project directory

2. Share your client's Figma frame:
   "Generate this button: [Figma URL or attach screenshot]"

3. Claude classifies the component, extracts brand values, and confirms:
   "Got it — Button. I'll generate 108 variants.
    18 inverse variants will be stubbed (no dark-background colors in input).
    Ready to generate?"

4. Confirm → Claude produces the generation plan and executes it directly in Figma via MCP

5. Review output. Stubs appear in bright pink — easy to find and fill in.
```

**Typical time: under 15 minutes** from frame to generated component set.

---

## Spec Library

Specs live in `specs/`. The index at `specs/_index.json` is the entry point for component lookup.

| Component | File | Status |
|-----------|------|--------|
| Button | `specs/button.spec.json` | Done — 108 variants |
| Input | — | Planned |
| Toggle | — | Planned |
| Checkbox | — | Planned |

**To add a new component spec:** use the `ds-spec-authoring` skill. This is a DS-team task, not a per-project task.

---

## Generation Plan Format

The `generation-plan.json` is the structured set of instructions Claude uses to execute the component build in Figma via MCP.

```json
{
  "generation_plan": {
    "component": "Button",
    "spec_file": "specs/button.spec.json",
    "token_mapping": {
      "resolved": { "system/bg/primary": "#E53E3E", ... },
      "sources":  { "#E53E3E": "designer input — primary fill on contained/idle", ... }
    },
    "variants": [
      { "action": "create_from_input", ... },
      { "action": "scale_size", ... },
      { "action": "derive_from_variant", ... },
      { "action": "apply_state_modifier", ... },
      { "action": "create_stub", ... }
    ],
    "assembly": { "component_set_name": "button", ... },
    "validation": { "checks": [...], "expected_variant_count": 108 },
    "stubbed": [ { "reason": "inverse brand colors not in input", "count": 18 } ]
  }
}
```

Variants are ordered for execution: `create_from_input` → `scale_size` → `derive_from_variant` → `apply_state_modifier` → `create_stub`.

---

## Repository Structure

```
specs/
  _index.json                ← component index for fast lookup (read this first)
  button.spec.json
  {component}.spec.json      ← one file per component

skills/
  ds-spec-authoring.md       ← DS team: extract and encode Kido components as specs
  ds-component-analysis.md   ← Designer: analyze client input against specs
  ds-generation-planning.md  ← Designer: produce generation plan, Claude executes in Figma

CLAUDE.md                    ← agent instructions (Claude reads this at session start)
README.md                    ← this file
```

---

## Requirements

| Requirement | Purpose |
|-------------|---------|
| Claude app or Claude Code | Where the designer interacts |
| Figma MCP connected | Enables reading Figma frames directly via URL |
| Figma MCP connected | Enables Claude to read designs and write components directly into Figma |

---

## Design Principles

**Kido rules are the default.** The designer's input provides brand values only. States, sizes, spacing, derivation logic, and accessibility come from the spec — not from the designer, and not from Claude's judgment.

**Questions block progress; stubs don't.** When a value can't be resolved, Claude creates a stub and proceeds. The designer polishes stubs in Figma after generation, not before.

**Client styling is preserved through token mapping.** Kido defines the structure (what variants exist and how they relate). The client's input provides the values (their colors, radius, font). The output looks like the client's brand, not Kido.

**Specs are infrastructure.** One well-authored spec pays for itself across every project that needs that component type. The better the spec, the more automatic the generation.

---

## Limitations

- **No spec = no full automation.** Components without a spec trigger `ds-spec-authoring` first. This is a one-time setup cost, not a per-project cost.
- **Inverse variants almost always stub.** Client designs rarely show dark-background variants explicitly. Stubs are expected output — not a failure.
- **Figma MCP required for frame input.** Screenshot input works but extracts less precise token values. Figma URL input is strongly preferred.
- **Kido drift.** If Kido DS changes and specs aren't updated, output may not match the current standard. Re-run `ds-spec-authoring` for any changed Kido component.

---

*The goal: adding the tenth component to a client project takes five minutes, not five hours.*
