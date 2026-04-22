---
name: ds-extract-design
description: >
  Extract design tokens from a Figma foundation file and emit a DESIGN.md file in hybrid format:
  YAML front matter (design.md CLI-compatible) + DTCG JSON body (W3C standard, consumed by /ds-build).
  Section order follows the google-labs-code/design.md canonical sequence.
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

- **Figma foundation file URL(s)** — one or more Figma file URLs to extract from. Foundations are often split across multiple files (e.g., colors in one file, spacing/typography/radius in another, icons in a third). Pass all of them — they will be extracted in sequence and merged into a single DESIGN.md.
- **Project name** (required) — a short slug used as the working directory name (e.g., `acme`, `payzo`, `quirky`). All `/ds-build` runs for this project will reference the same directory, so the name should be recognizable and reusable.

**If the designer doesn't provide a project name, ask for one.** Don't derive it silently — it's the key that ties all Workflow B artifacts together for this project.

**Multi-file foundations are the norm, not the exception.** If the designer provides only one URL but the project has separate icon or color files, ask: "Is this the only foundation file, or are there others (e.g. a separate colors file or icon library)?"

---

## Output

`working/{project}/DESIGN.md`

Hybrid Markdown + DTCG JSON blocks. Markdown headings for human navigation; JSON for programmatic consumption by `/ds-build`.

---

## Step 1 — Read the Foundation File(s)

Foundations are frequently split across multiple Figma files. Extract each file separately, then merge the results before writing DESIGN.md.

**Common split patterns:**
| File | Typical contents |
|---|---|
| Colors file | All color primitives + semantic color variables |
| Foundations / Tokens file | Spacing, radius, typography, shadow, sizing |
| Icons file | Icon components (referenced, not extracted as tokens) |
| Components file | Component library (may contain token references) |

**For each file**, use the highest-fidelity extraction available:

1. **`figma_get_design_system_kit`** — one-call extraction of variables + styles + components. Use if available.
2. **`figma_get_variables`** + **`figma_get_styles`** — separate calls, combine the results.
3. **`figma_get_variable_defs`** + **`get_design_context`** — fallback when the other tools aren't available.

Extract every mode each file contains (light, dark, brand variations). If two files share the same mode name (e.g., both have a "light" mode), merge their tokens under that mode — do not create duplicate mode sections.

**Merge rule:** tokens from later files overwrite tokens from earlier files only if they share the exact same path. Otherwise append. The designer should specify which file is authoritative if there is a conflict.

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

### Format

The file is a **hybrid** that satisfies two standards simultaneously:

1. **YAML front matter** (between `---` fences) — machine-readable summary compatible with the [google-labs-code/design.md](https://github.com/google-labs-code/design.md) CLI (`design.md lint`, `diff`, `export`). Uses simple flat values: hex for colors, `Npx` for dimensions.
2. **DTCG markdown body** — full W3C DTCG token tree in JSON code blocks, consumed by `/ds-build`. Uses `$type`/`$value`/`$description` with `{path.to.token}` aliases.

The body section order follows the canonical [design.md](https://github.com/google-labs-code/design.md) sequence: Overview → Colors → Typography → Layout → Elevation & Depth → Shapes → Components → Do's and Don'ts → Themes → Gaps and Notes.

### Template

~~~markdown
---
name: {project}
version: alpha
description: {one-line description of the client's design system}
colors:
  primary: "{resolved hex of the primary brand color}"
  primaryHover: "{resolved hex}"
  surface: "{resolved hex of default background}"
  onSurface: "{resolved hex of default text}"
  border: "{resolved hex of default border}"
  error: "{resolved hex of error/critical color}"
  warning: "{resolved hex of warning color}"
  success: "{resolved hex of success color}"
typography:
  body:
    fontFamily: "{font family name}"
    fontSize: "{Npx}"
    fontWeight: {number}
    lineHeight: "{Npx}"
  heading:
    fontFamily: "{font family name}"
    fontSize: "{Npx}"
    fontWeight: {number}
    lineHeight: "{Npx}"
rounded:
  sm: "{Npx}"
  md: "{Npx}"
  lg: "{Npx}"
  full: "9999px"
spacing:
  xs: "{Npx}"
  sm: "{Npx}"
  md: "{Npx}"
  lg: "{Npx}"
  xl: "{Npx}"
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.surface}"
    rounded: "{rounded.md}"
    padding: "{spacing.sm} {spacing.md}"
---

# DESIGN.md

> Extracted from: **{Figma file name}**
> Source: `{Figma URL}`
> Extraction date: {YYYY-MM-DD}
> Modes: {list of modes}
> Token count: {N primitives + N semantic}

{One-paragraph description of the design system — brand personality, visual approach, target product context.}

---

## Overview

{2–4 sentences on the visual identity: primary color and what it conveys, typeface choice, spacing philosophy, elevation style. Write for a developer who hasn't seen the Figma file.}

---

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
      "100": { "$type": "color", "$value": "#f5f5f5" },
      "900": { "$type": "color", "$value": "#1a1c1e" }
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
      "inverse":  { "$type": "color", "$value": "{color.neutral.0}" }
    },
    "bg": {
      "surface":  { "$type": "color", "$value": "{color.neutral.0}" },
      "subtle":   { "$type": "color", "$value": "{color.neutral.100}" }
    },
    "border": {
      "default":  { "$type": "color", "$value": "{color.neutral.200}" }
    }
  }
}
```

---

## Typography

```json
{
  "typography": {
    "heading": {
      "h1": { "$type": "typography", "$value": { "fontFamily": "Inter", "fontSize": "32px", "fontWeight": 700, "lineHeight": "40px", "letterSpacing": "0" } },
      "h2": { "$type": "typography", "$value": { "fontFamily": "Inter", "fontSize": "24px", "fontWeight": 700, "lineHeight": "32px", "letterSpacing": "0" } }
    },
    "body": {
      "lg": { "$type": "typography", "$value": { "fontFamily": "Inter", "fontSize": "16px", "fontWeight": 400, "lineHeight": "24px", "letterSpacing": "0" } },
      "sm": { "$type": "typography", "$value": { "fontFamily": "Inter", "fontSize": "14px", "fontWeight": 400, "lineHeight": "20px", "letterSpacing": "0" } }
    },
    "label": {
      "md": { "$type": "typography", "$value": { "fontFamily": "Inter", "fontSize": "14px", "fontWeight": 600, "lineHeight": "20px", "letterSpacing": "0" } }
    }
  }
}
```

> Desktop vs mobile: list any scale differences here. If the file has mobile typography tokens, extract them under a `mobile` sub-key with the same tier names (`heading/h1/mobile`, etc.) so the desktop/mobile correlation rule holds.

---

## Layout

```json
{
  "layout": {
    "breakpoint": {
      "sm":  { "$type": "dimension", "$value": "640px" },
      "md":  { "$type": "dimension", "$value": "768px" },
      "lg":  { "$type": "dimension", "$value": "1024px" },
      "xl":  { "$type": "dimension", "$value": "1280px" }
    },
    "grid": {
      "columns": { "$type": "number", "$value": 12 },
      "gutter":  { "$type": "dimension", "$value": "24px" },
      "margin":  { "$type": "dimension", "$value": "24px" }
    },
    "spacing": {
      "none": { "$type": "dimension", "$value": "0px" },
      "xs":   { "$type": "dimension", "$value": "4px" },
      "sm":   { "$type": "dimension", "$value": "8px" },
      "md":   { "$type": "dimension", "$value": "16px" },
      "lg":   { "$type": "dimension", "$value": "24px" },
      "xl":   { "$type": "dimension", "$value": "32px" },
      "2xl":  { "$type": "dimension", "$value": "48px" },
      "3xl":  { "$type": "dimension", "$value": "64px" }
    }
  }
}
```

> If the Figma file has no layout tokens, note it here and use defaults from the library spec.

---

## Elevation & Depth

```json
{
  "shadow": {
    "sm":  { "$type": "shadow", "$value": { "color": "#0000001a", "offsetX": "0", "offsetY": "1px", "blur": "2px",  "spread": "0" } },
    "md":  { "$type": "shadow", "$value": { "color": "#0000001f", "offsetX": "0", "offsetY": "4px", "blur": "8px",  "spread": "0" } },
    "lg":  { "$type": "shadow", "$value": { "color": "#00000029", "offsetX": "0", "offsetY": "8px", "blur": "16px", "spread": "0" } },
    "ring.focused": { "$type": "shadow", "$value": { "color": "{color.interactive.primary}", "offsetX": "0", "offsetY": "0", "blur": "0", "spread": "2px" } }
  }
}
```

---

## Shapes

```json
{
  "radius": {
    "none": { "$type": "dimension", "$value": "0px" },
    "sm":   { "$type": "dimension", "$value": "4px" },
    "md":   { "$type": "dimension", "$value": "8px" },
    "lg":   { "$type": "dimension", "$value": "12px" },
    "xl":   { "$type": "dimension", "$value": "16px" },
    "full": { "$type": "dimension", "$value": "9999px" }
  }
}
```

---

## Components

> Flat component token summary — enough for the design.md CLI's `components:` front matter key. Full per-component slot mappings live in `token-map.json` (generated by `/ds-build`).

```json
{
  "component": {
    "button-primary": {
      "backgroundColor": { "$type": "color", "$value": "{color.interactive.primary}" },
      "textColor":       { "$type": "color", "$value": "{color.text.inverse}" },
      "rounded":         { "$type": "dimension", "$value": "{radius.md}" }
    },
    "button-outline": {
      "backgroundColor": "transparent",
      "textColor":       { "$type": "color", "$value": "{color.interactive.primary}" },
      "borderColor":     { "$type": "color", "$value": "{color.interactive.primary}" },
      "rounded":         { "$type": "dimension", "$value": "{radius.md}" }
    }
  }
}
```

---

## Do's and Don'ts

> Capture any explicit usage rules from the Figma file annotations, library docs, or client brief. If none are present, write "None documented." — do not invent rules.

- **Do:** {usage rule}
- **Don't:** {anti-pattern}

---

## Themes

For each non-default mode (`dark`, brand variants, etc.), emit semantic token overrides only — primitives stay constant.

```json
{
  "theme": {
    "dark": {
      "color": {
        "interactive": {
          "primary": { "$type": "color", "$value": "{color.brand.400}" }
        },
        "text": {
          "default": { "$type": "color", "$value": "{color.neutral.0}" }
        },
        "bg": {
          "surface": { "$type": "color", "$value": "{color.neutral.900}" }
        }
      }
    }
  }
}
```

---

## Gaps and Notes

Free text. Record anything that couldn't be extracted cleanly: missing tokens, unusual naming, composite variables that don't map to DTCG, stubs that need designer input.
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
