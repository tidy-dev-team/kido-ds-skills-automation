---
name: ds-component-analysis
description: >
  Analyze a client component input against the Kido DS spec library and produce a gap report
  ready for generation planning. Use this skill whenever a designer provides a Figma frame,
  screenshot, or description of a component they want to generate.
  Triggers on phrases like "generate this button", "build this component", "here's my design",
  "create variants for this", or when a designer shares a Figma URL or image of a component.
  Always run before ds-generation-planning — the gap report is its required input.
---

# DS Component Analysis

**Philosophy: generate-first.**

Kido DS best practices cover everything structural — states, sizes, spacing, derivation logic, accessibility. The designer's input provides one thing: brand values (colors, radius, font). Everything else comes from the spec automatically.

Ask as little as possible. The bar is: this should feel easier than building the component manually, not harder.

---

## What the Designer Provides

- A Figma frame or URL (primary — use `get_design_context` via Figma MCP)
- A screenshot or image (fallback — analyze visually)
- Occasionally: a text description

The input is almost always partial. That's fine. The spec fills the gaps.

---

## Step 1 — Classify

Identify which Kido component type the input represents.

1. Read `specs/_index.json` to get candidate components
2. Extract visual signals from the input: shape, interactive affordance, text, icon slots, size, layout
3. Match against the index using component descriptions, anatomy keywords, and aliases
4. Assign confidence: `high` (>85%) or `low` (≤85%)

**If confidence is high:** proceed silently. Note the match in the gap report.

**If confidence is low:** ask one question — "This looks like a [X]. Is that right, or is it a [Y]?" Do not run the diff until confirmed.

**If no spec matches:** follow the "No Matching Spec" process below.

---

## Step 2 — Extract Brand Values

Extract the client's concrete visual values from the input. These are the only values that come from the designer — everything else is Kido's.

**What to extract:**
- Primary brand color (the dominant interactive color)
- Secondary brand color if present
- Border radius (if it visibly differs from spec standard)
- Font family (if it visibly differs from spec standard — Inter is Kido's default)
- Any color explicitly applied to a specific token slot (e.g. a disabled state color)

**How to extract:**
- From a Figma frame: use `get_design_context` — look at fills, strokes, text styles on the base (idle) variant
- From an image: identify the primary fill color, radius, and font visually
- From the variable defs if available: pull token values directly

**Token mapping — map extracted values to spec token slots:**

```json
{
  "token_mapping": {
    "extracted": {
      "system/bg/primary": "#E53E3E",
      "radius/semantic/large-controls": "12px"
    },
    "derived": {
      "components/button/outlined/bg-hover": "#E53E3E33",
      "components/button/outlined/bg-pressed": "#E53E3E61"
    },
    "kido_defaults_applied": [
      "system/fg/disabled",
      "system/bg/disabled",
      "system/border/interactive/focused",
      "system/overlays/overlay-hover"
    ],
    "stubbed": [
      {
        "token": "system/fg/secondary",
        "reason": "No secondary color visible in input",
        "placeholder": "NEEDS_VALUE"
      }
    ]
  }
}
```

**Deviations from spec standard:** use the input value. Note the deviation in the gap report. Do not ask the designer about it — they gave you their value intentionally.

**Missing brand values:** if a value is genuinely not visible in the input (e.g. client only showed a primary button and secondary color is unknown), stub it as `NEEDS_VALUE`. Generation proceeds with the stub — the designer polishes it after.

**Only ask the designer** if a brand value is both: (a) missing from the input AND (b) cannot be stubbed without making the component useless. In practice, this is rare. When in doubt, stub and proceed.

---

## Step 3 — Apply Spec Rules

Load the matched spec. Apply all derivation rules from `derivation_rules.rules` automatically.

Every variant covered by a derivation rule is `derivable` — no designer input needed, no approval needed. Generate it.

**Structural completeness check:** verify the input's anatomy against `anatomy.layers`. Note any missing required layers — these become `manual_notes` in the generation plan, not questions.

**Output of this step:**

```json
{
  "diff": {
    "base_variants_from_input": [
      { "variant": "type=contained, size=m, state=idle, inverse=no", "source": "extracted" }
    ],
    "auto_generated": [
      { "variant": "type=contained, size=m, state=hover, inverse=no", "rule": "hover-contained" },
      { "variant": "type=contained, size=s, state=idle, inverse=no",  "rule": "scale-size-from-m" }
    ],
    "stubbed": [
      { "variant": "type=contained, all states, inverse=yes", "reason": "No inverse input provided", "placeholder": "NEEDS_VALUE" }
    ]
  }
}
```

There is no `derivable_with_approval` category. If a Kido rule covers it, it is generated.

---

## Gap Report (Output)

```json
{
  "gap_report": {
    "schema_version": "0.1",
    "timestamp": "ISO8601",
    "input_summary": "One-sentence description of what was analyzed",
    "spec_used": "specs/button.spec.json",
    "spec_version": "1.0.0",
    "spec_match": "exact | closest | composed",
    "spec_match_notes": "only present if not exact",

    "classification": {
      "component_type": "Button",
      "confidence": "high",
      "confidence_score": 0.94,
      "signals": ["rectangular", "text label", "primary fill"]
    },

    "token_mapping": { },   // from Step 2
    "diff": { },            // from Step 3

    "summary": {
      "total_required_variants": 108,
      "from_input": 1,
      "auto_generated": 89,
      "stubbed": 18,
      "stub_reason": "inverse=yes brand values not in input"
    },

    "manual_notes": [
      "Radius deviates from spec standard: input uses 12px, spec standard is 8px — using input value"
    ]
  }
}
```

**No `designer_questions` array.** Questions block generation. Stubs don't. Default to stubbing.

---

## Presenting Results to the Designer

Keep it short. One paragraph, then offer to proceed:

```
Got it — Button (contained, MD, idle state). Matched to button.spec.json.

I'll generate 108 variants: 1 from your design, 89 from Kido rules.
18 inverse variants will be stubbed (no dark-background colors in your input — easy to fill in Figma after).

Ready to generate?
```

Do not show the JSON to the designer unprompted. It's for the generation planning step.

---

## No Matching Spec

When no spec clearly matches, try in order:

**1. Find the closest spec** — rank by shared variant axes, similar anatomy, similar interaction model. Proceed with it as a base and note the differences. Tell the designer: "No exact spec for [X]. Using [Y] as the closest match — flagging where they differ."

**2. Compose from two specs** — if the component combines two known patterns (e.g. a Tag = Button structure + Badge sizing), use both. Note which rules come from each.

**3. Ask** — only if neither approach yields a usable base: "I don't have a spec for this component type. Should I use [closest match] as a base, or would you like to author a new spec first with `ds-spec-authoring`?"

---

## When to Ask the Designer

Ask only in these situations — nothing else:

| Situation | What to ask |
|-----------|-------------|
| Classification confidence ≤85% | "This looks like [X] — correct?" |
| No matching spec and composition fails | "Should I use [Y] as a base?" |
| Brand value missing AND cannot be stubbed | Ask for the specific value only |

Everything else — states, sizes, spacing, derivations, accessibility, naming — is handled by the spec. Never ask about those.
