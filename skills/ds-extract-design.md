---
name: ds-extract-design
description: >
  Extract design tokens from a Figma foundation file and emit a DESIGN.md file in DTCG format.
  Use this at the start of a Workflow B project (before /ds-build) to produce the token reference
  that will be mapped onto the client's UI library structure.
  Triggers on phrases like "extract tokens from this Figma file", "create DESIGN.md from Figma",
  "read the foundation file", "extract design decisions", or when a designer shares a Kido or
  client foundation Figma URL and says they need it in a token format.
---

# DS Extract Design

**Figma foundation file → DESIGN.md with DTCG-formatted tokens.**

The output is a per-project reference consumed by `/ds-build`. Not committed. Fresh extraction per project.

---

## When to Use This

Run this once per project, before the first `/ds-build` call.

- A new client project is starting and the client uses an existing UI library (Workflow B).
- You need to capture the visual foundations (Kido's or the client's) as machine-readable tokens.
- The foundation Figma file has changed and you need to re-extract.

Do **not** run this for Workflow A projects (`/ds-generate`) — that flow extracts brand values on-the-fly from client input.

---

## Inputs

- **Figma foundation file URL** — the file to extract from. Usually Kido's foundation. Can be a client's own foundation if they have one.
- **Project name** (required) — a short slug used as the working directory name (e.g., `acme`, `payzo`, `quirky`). All `/ds-build` runs for this project will reference the same directory, so the name should be recognizable and reusable.

**If the designer doesn't provide a project name, ask for one.** Don't derive it silently — it's the key that ties all Workflow B artifacts together for this project.

---

## Output

`working/{project}/DESIGN.md`

Hybrid Markdown + DTCG JSON blocks. Markdown headings for human navigation; JSON for programmatic consumption by `/ds-build`.

---

## Step 1 — Read the Foundation File

Use the highest-fidelity extraction available. Preferred order:

1. **`figma_get_design_system_kit`** — one-call extraction of variables + styles + components. Use if available for the file.
2. **`figma_get_variables`** + **`figma_get_styles`** — separate calls, combine the results.
3. **`figma_get_variable_defs`** + **`get_design_context`** — fallback when the other tools aren't available.

Extract every mode the file contains (light, dark, brand variations). Record the mode name as a theme layer in the output.

---

## Step 2 — Categorize Tokens

Place every extracted value into one of these categories. Use the token name from Figma as the source of truth for categorization (e.g. `color/primary/500` → color; `space/md` → spacing).

| Category | DTCG `$type` | Examples |
|---|---|---|
| Color | `color` | Primary, secondary, semantic (success/warning/error), surfaces, text |
| Typography | `typography` (composite) | Font family, size, weight, line height, letter spacing |
| Spacing | `dimension` | Padding, margin, gap scales |
| Sizing | `dimension` | Component heights, icon sizes |
| Radius | `dimension` | Border radius scale |
| Shadow | `shadow` | Elevation tokens |
| Border | `dimension` / `color` | Border width, border color |
| Duration | `duration` | Animation durations |
| Easing | `cubicBezier` | Animation curves |

Separate **primitives** (raw values like `#615fff`) from **semantic** (purposeful names like `color.interactive.primary`). In DTCG, semantic tokens reference primitives using `{token.path}` syntax.

---

## Step 3 — Write DESIGN.md

Template structure:

~~~markdown
# DESIGN.md

> Extracted from: **{Figma file name}** ({Figma URL})
> Extraction date: {YYYY-MM-DD}
> Modes: {list of modes}
> Token count: {total}

This file is a DTCG-formatted snapshot of the design foundations. Used by `/ds-build` to style components against the client's UI library structure.

## Colors

### Primitives

```json
{
  "color": {
    "brand": {
      "500": { "$type": "color", "$value": "#615fff", "$description": "Base brand color" },
      "600": { "$type": "color", "$value": "#4f39f6", "$description": "Brand one step darker" }
    },
    "neutral": {
      "0":   { "$type": "color", "$value": "#ffffff" },
      "100": { "$type": "color", "$value": "#f5f5f5" }
    }
  }
}
```

### Semantic

```json
{
  "color": {
    "interactive": {
      "primary":      { "$type": "color", "$value": "{color.brand.500}" },
      "primaryHover": { "$type": "color", "$value": "{color.brand.600}" }
    },
    "text": {
      "default":  { "$type": "color", "$value": "{color.neutral.900}" },
      "muted":    { "$type": "color", "$value": "{color.neutral.600}" },
      "onBrand":  { "$type": "color", "$value": "{color.neutral.0}" }
    }
  }
}
```

## Typography

```json
{
  "typography": {
    "body": {
      "$type": "typography",
      "$value": {
        "fontFamily": "Inter",
        "fontSize": "16px",
        "fontWeight": 400,
        "lineHeight": "24px",
        "letterSpacing": "0"
      }
    }
  }
}
```

## Spacing

```json
{
  "space": {
    "xs": { "$type": "dimension", "$value": "4px" },
    "sm": { "$type": "dimension", "$value": "8px" },
    "md": { "$type": "dimension", "$value": "16px" }
  }
}
```

## Radius

```json
{
  "radius": {
    "sm": { "$type": "dimension", "$value": "4px" },
    "md": { "$type": "dimension", "$value": "8px" },
    "lg": { "$type": "dimension", "$value": "12px" }
  }
}
```

## Shadow

```json
{
  "shadow": {
    "elevation-1": {
      "$type": "shadow",
      "$value": { "color": "#0000001f", "offsetX": "0", "offsetY": "1px", "blur": "2px", "spread": "0" }
    }
  }
}
```

## Themes

For each non-default mode (e.g. `dark`), emit a section under "Themes" that overrides semantic token values:

```json
{
  "theme": {
    "dark": {
      "color": {
        "interactive": {
          "primary": { "$type": "color", "$value": "{color.brand.400}" }
        }
      }
    }
  }
}
```

## Gaps and Notes

Free text. Record anything that couldn't be extracted cleanly (missing tokens, unusual naming, composite variables that don't map to DTCG).
~~~

---

## Step 4 — Report

```
DESIGN.md extracted.

Source: {Figma file name}
Output: working/{project}/DESIGN.md
Modes:  light, dark
Tokens: 247 (184 primitive, 63 semantic)

Ready to run /ds-build.
```

---

## Critical Rules

**Never invent token values.** If a referenced variable is missing, note it in the "Gaps" section — do not fabricate a value.

**Preserve the client's naming when possible.** If the Figma uses `color/main/primary`, keep that path. Do not rename to Kido conventions unless explicitly requested.

**Resolve aliases in both directions.** DTCG allows `"$value": "{color.brand.500}"` references. Preserve these — don't flatten everything to raw values. `/ds-build` can follow them.

**Capture all modes.** A file with `light` and `dark` modes needs both represented. Missing a mode breaks multi-theme component generation.

**DESIGN.md is per-project.** Overwrite the previous file if it exists. Do not append.
