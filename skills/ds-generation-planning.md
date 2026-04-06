---
name: ds-generation-planning
description: >
  Convert a gap report from ds-component-analysis into a generation plan the Figma Figma MCP can execute.
  Use this skill immediately after ds-component-analysis produces a gap report.
  Triggers on phrases like "generate the plan", "ready to generate", "create the component",
  "build it", "execute in Figma", or when the designer confirms they're ready to proceed.
  Never use without a gap report — run ds-component-analysis first if one doesn't exist.
---

# DS Generation Planning

**Philosophy: fully automatic.**

The gap report contains all decisions. The spec contains all rules. This skill assembles them into executable Figma MCP instructions without asking the designer anything. Stubs make gaps visible in Figma — the designer polishes after generation, not before.

---

## Prerequisite

A gap report from `ds-component-analysis` exists. That's it.

No Q&A required. No approvals required. Stubs are acceptable output.

---

## How the Plan is Built

### 1. Resolve the Token Map

Take `gap_report.token_mapping` and produce a flat, fully-resolved map:
- `extracted` values → use as-is
- `derived` values → use as-is (already computed in analysis)
- `kido_defaults_applied` → pull resolved hex values from the spec's `resolved_token_values`
- `stubbed` values → keep as `"NEEDS_VALUE"`

Every token in the map must have a source entry. This is the audit trail.

```json
{
  "token_mapping": {
    "resolved": {
      "system/bg/primary":                   "#E53E3E",
      "system/fg/inverse":                   "#ffffff",
      "system/bg/disabled":                  "#62748e1f",
      "system/fg/disabled":                  "#cad5e2",
      "system/border/interactive/focused":   "#E53E3E80",
      "components/button/outlined/bg-hover": "#E53E3E33",
      "radius/semantic/large-controls":      "12px",
      "system/bg/primary-inverse":           "NEEDS_VALUE"
    },
    "sources": {
      "#E53E3E":    "designer input — primary fill on contained/idle",
      "#62748e1f":  "Kido default — system/bg/disabled",
      "#E53E3E80":  "derived — primary at 50% opacity (focus ring)",
      "#E53E3E33":  "derived — primary at 20% opacity (outlined hover bg)",
      "NEEDS_VALUE": "inverse brand color not in input — stub"
    }
  }
}
```

**Deriving missing values from a known primary color:**
- Focus ring: primary color at 50% opacity
- Outlined hover bg: primary color at 20% opacity
- Outlined pressed bg: primary color at 38% opacity
- Hover overlay on contained: `#0000001f` (Kido constant — not brand-specific)

### 2. Build Variant Instructions

One instruction per variant, ordered for execution: base variants first, derived variants after.

**Instruction types:**

`create_from_input` — build directly from designer's provided design
```json
{
  "action": "create_from_input",
  "variant_id": "contained-m-idle-no",
  "variant_properties": { "type": "contained", "size": "m", "state": "idle", "inverse": "no" },
  "layer_instructions": [
    { "layer": "Container", "property": "fills",        "token": "system/bg/primary" },
    { "layer": "Label",     "property": "color",        "token": "system/fg/inverse" },
    { "layer": "Container", "property": "cornerRadius", "token": "radius/semantic/large-controls" }
  ]
}
```

`derive_from_variant` — modify an existing variant using a spec derivation rule
```json
{
  "action": "derive_from_variant",
  "variant_id": "contained-m-hover-no",
  "variant_properties": { "type": "contained", "size": "m", "state": "hover", "inverse": "no" },
  "source_variant_id": "contained-m-idle-no",
  "rule_id": "hover-contained",
  "changes": [
    { "layer": "Container", "property": "overlay", "token": "system/overlays/overlay-hover-on-color" }
  ]
}
```

`scale_size` — build a size variant from a base size using spec spacing tokens
```json
{
  "action": "scale_size",
  "variant_id": "contained-s-idle-no",
  "variant_properties": { "type": "contained", "size": "s", "state": "idle", "inverse": "no" },
  "source_variant_id": "contained-m-idle-no",
  "size_tokens": { "paddingX": 12, "paddingY": 9, "gap": 4, "typography": "label/m", "min_height": 32, "min_width": 60 }
}
```

`apply_state_modifier` — apply a standard state (disabled, focused, loading) to a base variant
```json
{
  "action": "apply_state_modifier",
  "variant_id": "contained-m-disabled-no",
  "variant_properties": { "type": "contained", "size": "m", "state": "disabled", "inverse": "no" },
  "source_variant_id": "contained-m-idle-no",
  "modifier": "disabled",
  "modifier_params": {
    "background_token": "system/bg/disabled",
    "label_token":      "system/fg/disabled",
    "border_token":     null
  }
}
```

`create_stub` — create a visible placeholder for a variant with missing values
```json
{
  "action": "create_stub",
  "variant_id": "contained-m-idle-yes",
  "variant_properties": { "type": "contained", "size": "m", "state": "idle", "inverse": "yes" },
  "stub_fill": "#FF0066",
  "stub_label": "NEEDS_VALUE: inverse brand color",
  "reason": "inverse=yes brand values not in input"
}
```

**Execution order:**
1. `create_from_input` — no dependencies
2. `scale_size` — depends on base size existing
3. `derive_from_variant` — depends on source variant existing
4. `apply_state_modifier` — depends on base state existing
5. `create_stub` — last, no dependencies

### 3. Assembly Instructions

How the Figma MCP groups variants into a component set.

```json
{
  "assembly": {
    "component_set_name": "button",
    "variant_format": "size={size}, type={type}, state={state}, inverse={inverse}",
    "layout": {
      "arrange_by": ["type", "size"],
      "group_spacing": 40,
      "variant_spacing": 16
    },
    "component_properties": [
      { "name": "Label",  "type": "text",    "default": "Button" },
      { "name": "Icon L", "type": "boolean", "default": false },
      { "name": "Icon R", "type": "boolean", "default": false }
    ]
  }
}
```

Assembly config comes from the spec's `naming` section. Use it exactly — do not rename properties.

### 4. Validation Checklist

Pulled directly from the spec's `completeness_rules.validation`. The Figma MCP runs these after generation.

---

## Full Plan Output

```json
{
  "generation_plan": {
    "schema_version": "0.1",
    "timestamp": "ISO8601",
    "component": "Button",
    "spec_file": "specs/button.spec.json",
    "spec_version": "1.0.0",

    "token_mapping": { },    // resolved map with sources
    "variants": [ ],         // ordered instruction array
    "assembly": { },
    "validation": { },

    "stubbed": [
      {
        "variant": "all inverse=yes variants",
        "reason": "inverse brand colors not in input",
        "count": 18
      }
    ],

    "manual_notes": [
      "Border radius uses 12px (input) instead of spec standard 8px"
    ]
  }
}
```

---

## Summary to the Designer

After outputting the JSON, give a one-paragraph summary. Keep it short.

```
Button component plan ready.

108 variants total:
  • 1 built from your design
  • 89 auto-generated from Kido rules
  • 18 stubs (inverse variants — fill in brand colors after generation)

Pass to the Figma Figma MCP to run. Stubs will appear in bright pink so they're easy to find.
```

Do not walk through the JSON with the designer. It's for the Figma MCP.

---

## Stubs

A stub is a valid output. It means: "this variant exists in the component set, its value is a placeholder, the designer fills it in Figma."

- Stubs use a bright fill (`#FF0066`) so they stand out visually
- Stub label text describes what's needed: "NEEDS_VALUE: inverse brand color"
- Never silently drop a required variant — always stub it if it can't be resolved

The designer polishes stubs after generation. This is the intended workflow.
