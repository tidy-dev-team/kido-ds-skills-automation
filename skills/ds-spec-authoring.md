---
name: ds-spec-authoring
description: >
  Extract and author machine-readable component specs (component.spec.json) from the Kido DS Figma file.
  Use this skill whenever someone wants to: add a new component to the design system spec library,
  update an existing spec after Kido changes, or document what "production-ready" means for a component type.
  Triggers on phrases like "add X to our specs", "spec out a component", "extract from Kido",
  "update the button spec", "document our DS standards for X", or any request to encode design system
  rules into structured format. Always use this skill before the analysis or generation planning skills —
  specs are the foundation everything else depends on.
---

# DS Spec Authoring

Transforms a Kido DS component into a versioned `component.spec.json` that the analysis and generation planning skills can reason from.

The spec is not a mirror of Kido. It encodes:
- **Structure** (what the component is made of)
- **Completeness rules** (what "production-ready" means for this type)
- **Derivation logic** (how missing variants are created from existing ones)
- **Accessibility requirements** (non-negotiable constraints)

The schema evolves. Each spec includes a `schema_version` field. When the schema changes, existing specs are migrated. The current schema version is defined in this skill — treat it as the source of truth.

---

## Current Schema Version: `0.1`

This is a living schema. Fields marked `[REQUIRED]` must be present. Fields marked `[RECOMMENDED]` should be present if the data exists. Fields marked `[OPTIONAL]` are for complex components.

---

## Authoring Process

### Step 1 — Identify the Component

Ask the user for:
- Component name (as it appears in Kido)
- Figma file URL (default: Kido DS — `https://www.figma.com/design/WNOcZbybSn7dAGjTMimIsJ/Kido---DS`)
- Any known quirks or special rules for this component

### Step 2 — Extract from Figma (via MCP)

Use the Figma MCP tools in this order:

1. `get_metadata` — confirm the file and locate the component
2. `get_design_context` — extract layer structure, variant axes, auto-layout config, spacing, typography
3. `get_variable_defs` — extract design tokens / variables referenced by the component
4. `create_design_system_rules` — let Figma's own analysis surface patterns (use as a cross-check, not the source of truth)

**What to capture from the extraction:**
- All variant property names and their values (e.g. `Type: [Primary, Secondary]`, `State: [Default, Hover]`)
- Layer names and hierarchy
- Auto-layout direction, padding, gap
- Text styles used
- Color variables / tokens referenced
- Dimension constraints (min/max width, fixed height)

**What the extraction will miss (fill in manually):**
- Which variants are required vs. optional
- Completeness rules (the definition of "done")
- Accessibility requirements
- Derivation logic (how to create missing variants)
- Design rationale (the "why" behind decisions)

### Step 3 — Enrich Manually

After extraction, walk through these questions with the user. Do not skip this step — it is where expertise gets encoded.

**Completeness questions:**
- Which Type values are required for every client project? Which are optional?
- Which States are required? (Default, Hover, Pressed, Focused, Disabled are almost always required)
- Are all Size × Type × State combinations required, or only certain axes cross?
- Is there a Loading state? An Error state? Any component-specific states?

**Accessibility questions:**
- What is the minimum touch target? (WCAG recommends 44×44px)
- Does this component need a visible focus ring? What are its specs?
- What contrast ratio is required for text? (WCAG AA = 4.5:1, WCAG AAA = 7:1)
- Any ARIA roles or labels that must be documented?

**Derivation questions (critical for the generation planning skill):**
- If a designer provides only the Default state, how is Hover derived? (e.g., darken background 8%)
- If only MD size is provided, how are SM and LG derived? (scale padding/typography)
- If Primary is provided, how is Secondary derived? (ask designer, or use spec default)
- What should never be auto-derived and always requires designer input?

### Step 4 — Write the Spec

Output a single `{component-name}.spec.json` file following the schema below.

Always write the spec to: `specs/{component-name}.spec.json` relative to the project root (or wherever the user's spec library lives).

### Step 4b — Update the Index

After writing the spec, update `specs/_index.json`. This file is the LLM entry point for all component lookup — always keep it in sync.

**If `specs/_index.json` does not exist**, create it:
```json
{
  "schema_version": "0.1",
  "last_updated": "YYYY-MM-DD",
  "components": []
}
```

**Add or update the entry for this component:**
```json
{
  "file": "specs/{component-name}.spec.json",
  "component": "ComponentName",
  "aliases": [],
  "category": "category-name",
  "description": "One sentence: what this component is and what it does",
  "anatomy_keywords": ["key layer names and structural terms"],
  "related": ["OtherComponent"]
}
```

**Field guidance:**
- `aliases` — alternate names designers or developers might use (e.g. `["Btn", "CTA"]` for Button). Ask the user if unsure.
- `category` — use one of: `actions`, `forms`, `feedback`, `navigation`, `layout`, `data-display`. Ask if none fit.
- `description` — written for semantic search, not humans. Describe behavior and structure: "Interactive element that triggers an action; contains a label and optional leading/trailing icon."
- `anatomy_keywords` — exact Figma layer names plus structural synonyms. Pull from the spec's `anatomy.layers`.
- `related` — components that are structurally similar or commonly confused with this one.

Update `last_updated` to today's date whenever the index changes.

### Step 5 — Review

Read the spec back to the user in a human-readable summary:
- "This Button spec defines 5 Types, 3 Sizes, and 6 States = up to 90 variants"
- "Required combinations: Primary + Secondary × all Sizes × all States"
- "3 derivation rules will auto-generate Hover, Pressed, and size variants"
- "2 values will always require designer input: Secondary style and Destructive color"

Ask: "Does this match your expectations? Any rules that feel wrong or missing?"

Iterate until the user confirms.

---

## Spec Schema (v0.1)

```json
{
  "schema_version": "0.1",          // [REQUIRED] always "0.1" for now
  "component": "ComponentName",      // [REQUIRED] PascalCase, matches Kido name
  "version": "1.0.0",               // [REQUIRED] semver, start at 1.0.0
  "last_updated": "YYYY-MM-DD",     // [REQUIRED]
  "source": {                        // [REQUIRED]
    "figma_file": "URL",
    "figma_node_id": "node id if known"
  },

  "anatomy": {                       // [REQUIRED]
    "layers": [
      {
        "name": "LayerName",         // exact Figma layer name
        "type": "frame|text|instance|vector|group",
        "required": true,
        "notes": "optional description"
      }
    ],
    "autolayout": {
      "direction": "horizontal|vertical|none",
      "alignment": "center|start|end|space-between",
      "padding": "describe or reference token",
      "gap": "describe or reference token"
    }
  },

  "variants": {                      // [REQUIRED]
    "AxisName": {
      "values": ["Value1", "Value2"],
      "required": ["Value1"],        // must exist in every production component
      "optional": ["Value2"],        // include if client needs it
      "default": "Value1",           // used when axis is not specified
      "notes": "any special rules"
    }
  },

  "design_tokens": {                 // [RECOMMENDED]
    "colors": {
      "VariantValue": {
        "StateName": {
          "token_name": "token.reference.or.hex"
        }
      }
    },
    "spacing": {
      "SizeName": {
        "paddingX": 0,
        "paddingY": 0,
        "gap": 0
      }
    },
    "typography": {
      "SizeName": {
        "style": "token reference or description",
        "size": 0,
        "weight": 0,
        "lineHeight": 0
      }
    },
    "min_dimensions": {
      "SizeName": { "height": 0, "width": 0 }
    }
  },

  "accessibility": {                 // [REQUIRED]
    "min_touch_target": 44,
    "focus_ring": {
      "width": 2,
      "offset": 2,
      "color": "token or hex"
    },
    "contrast": "WCAG AA 4.5:1 for text",
    "required_states": ["Focused", "Disabled"],
    "aria_notes": "optional"
  },

  "naming": {                        // [REQUIRED]
    "component_set": "ComponentName",
    "variant_format": "Axis={Value}, Axis={Value}",
    "notes": "any naming exceptions"
  },

  "derivation_rules": {              // [RECOMMENDED] — used by ds-generation-planning
    "rules": [
      {
        "id": "rule-id",
        "description": "Human-readable description of this rule",
        "trigger": "when this variant/state is missing",
        "source": "derive from this variant/state",
        "method": "darken|lighten|scale|swap_tokens|ask_designer",
        "params": {},                // method-specific parameters
        "confidence": "high|medium|low",
        "fallback": "ask_designer if method fails"
      }
    ],
    "always_ask": [
      "Describe values that must never be auto-derived"
    ]
  },

  "completeness_rules": {            // [REQUIRED]
    "minimum_variants": "Human-readable description of minimum required set",
    "required_combinations": [
      "Type=Primary × all Sizes × all required States"
    ],
    "validation": [
      "Each validation check as a plain-English assertion"
    ]
  },

  "notes": "Any additional context, known issues, or future considerations"
}
```

---

## Handling Ambiguity During Authoring

**If the Figma MCP can't access the file:** Ask the user to paste the component's variant properties and layer structure manually. A screenshot is also acceptable — describe what you see and ask the user to confirm or correct.

**If variant axes are inconsistent in Kido:** Note the inconsistency in the spec's `notes` field. Do not "fix" Kido's inconsistencies in the spec — document them accurately and flag for the design systems engineer.

**If a token reference is unknown:** Use a descriptive placeholder like `"token.button.primary.bg"` and add it to a `"token_tbd": []` array at the top level. This makes gaps visible without blocking progress.

**If the designer hasn't decided something yet:** Add it to `derivation_rules.always_ask` rather than guessing. A spec with honest gaps is better than a spec with wrong assumptions.

---

## Spec Maintenance (Updating an Existing Spec)

When Kido changes and a spec needs updating:

1. Re-run `get_design_context` and `get_variable_defs` for the component
2. Diff the new extraction against the existing spec — what changed?
3. Update only the changed fields. Bump the `version` (patch for minor changes, minor for new variants, major for structural changes)
4. Update `last_updated`
5. Note what changed in a `changelog` field if the change is significant

Do not rewrite the whole spec from scratch on updates — preserve the manually-enriched fields (derivation rules, completeness rules, accessibility notes).

---

## Output

The final deliverable is `specs/{component-name}.spec.json`.

After writing the file, summarize:
- How many variant combinations the spec defines
- How many are required vs. optional
- How many derivation rules were captured
- How many values require designer input
- Any fields that were left as `token_tbd`

Then ask: "Ready to test this against a real component input? That's what the `ds-component-analysis` skill is for."
