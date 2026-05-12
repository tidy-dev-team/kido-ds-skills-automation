---
name: ds-extract-design
description: >
  Extract design tokens from a Figma foundation file and emit two sibling files in
  working/{project}/: DESIGN.md (front matter conforming to the google-labs-code/design.md spec
  plus a prose body of rationale) and tokens.json (full W3C DTCG token tree consumed by
  /ds-build). The split aligns with the design.md spec, which treats the body as prose.
  Top-level keys are identical across both files so a third-party tool reading only DESIGN.md
  still gets a usable summary.
  Use this at the start of a Workflow B project (before /ds-build) to produce the token reference
  that will be mapped onto the client's UI library structure.
  Triggers on phrases like "extract tokens from this Figma file", "create DESIGN.md from Figma",
  "read the foundation file", "extract design decisions", or when a designer shares a Kido or
  client foundation Figma URL and says they need it in a token format.
---

# DS Extract Design

**Figma foundation file → DESIGN.md (rationale) + tokens.json (DTCG tokens).**

Two sibling files in `working/{project}/`. `tokens.json` is the machine source of truth consumed by `/ds-build`; `DESIGN.md` is the human-readable companion with a spec-compliant token summary. Not committed. Fresh extraction per project.

---

## When to Use This

Run this once per project, before the first `/ds-build` call.

- A new client project is starting and the client uses an existing UI library (Workflow B).
- You need to capture the visual foundations (Kido's or the client's) as machine-readable tokens.
- The foundation Figma file has changed and you need to re-extract.

Do **not** run this for Workflow A projects (`/ds-generate`) — that flow extracts brand values on-the-fly from client input.

---

## Inputs

- **Figma foundation file URL(s)** — one or more Figma file URLs to extract from. Foundations are often split across multiple files (e.g., colors in one file, spacing/typography/radius in another, icons in a third). Pass all of them — they will be extracted in sequence and merged into a single `tokens.json` (with a matching `DESIGN.md` summary).
- **Project name** (required) — a short slug used as the working directory name (e.g., `acme`, `payzo`, `quirky`). All `/ds-build` runs for this project will reference the same directory, so the name should be recognizable and reusable.

**If the designer doesn't provide a project name, ask for one.** Don't derive it silently — it's the key that ties all Workflow B artifacts together for this project.

**Multi-file foundations are the norm, not the exception.** If the designer provides only one URL but the project has separate icon or color files, ask: "Is this the only foundation file, or are there others (e.g. a separate colors file or icon library)?"

---

## Output

- `working/{project}/tokens.json` — full W3C DTCG token tree. Source of truth, consumed by `/ds-build`.
- `working/{project}/DESIGN.md` — front matter (spec-compliant token summary) + prose body (rationale, palette philosophy, do's/don'ts). Human/agent reference; lints cleanly with `npx @google/design.md lint`.

Both files share identical top-level keys (`colors`, `typography`, `spacing`, `rounded`, `components`). DESIGN.md front matter is a flat extract of the semantic tokens in `tokens.json`.

---

## Step 1 — Read the Foundation File(s)

Foundations are frequently split across multiple Figma files. Extract each file separately, then merge the results before writing DESIGN.md and tokens.json.

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

## Step 3 — Write DESIGN.md + tokens.json

This step emits **two sibling files** in `working/{project}/`:

| File | What it is | Who reads it |
|---|---|---|
| `DESIGN.md` | Front matter (token summary, spec-compliant) + prose body (rationale). | Humans; agents looking up *why* a value exists; `npx @google/design.md lint`. |
| `tokens.json` | Full W3C DTCG token tree — primitives, semantic, modes, shadows, components. | `/ds-build` (programmatic consumer). Source of token truth. |

This split aligns with the [google-labs-code/design.md](https://github.com/google-labs-code/design.md) spec, which treats the markdown body as **prose rationale** ("Tokens give agents exact values. Prose tells them *why* those values exist and how to apply them."). The richer DTCG tree lives in a sibling file because the spec's front matter is too flat to express modes, primitives/semantic split, or nested namespaces.

**Cross-file consistency:** front matter keys and tokens.json top-level keys are identical (`colors`, `typography`, `spacing`, `rounded`, `components`). Front matter values are a flat extract of semantic tokens from `tokens.json` — a third-party tool that only reads the front matter still gets a usable summary; `/ds-build` reads `tokens.json` for the full picture.

The DESIGN.md body section order follows the canonical [design.md](https://github.com/google-labs-code/design.md) sequence: Overview → Colors → Typography → Layout → Elevation & Depth → Shapes → Components → Do's and Don'ts → Themes → Gaps and Notes. Each section is prose; the equivalent token data lives under the matching top-level key in `tokens.json`.

### DESIGN.md template

The body is **prose only**. No JSON blocks. Each section describes the *intent* of that area of the token tree; concrete values live in `tokens.json`.

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
> Companion file: `tokens.json` (full DTCG tree)

{One-paragraph description of the design system — brand personality, visual approach, target product context.}

---

## Overview

{2–4 sentences on the visual identity: primary color and what it conveys, typeface choice, spacing philosophy, elevation style. Write for a developer who hasn't seen the Figma file.}

---

## Colors

{Prose: how the palette is organized (primitives + semantic layering), the brand color's role, neutral ladder structure, any mode-specific behavior. Name 2–3 representative semantic tokens by their token path (e.g. `colors.interactive.primary`, `colors.text.muted`). Refer the reader to `tokens.json` for the full set.}

---

## Typography

{Prose: type family choices (heading vs body vs label), the size scale ladder, weight system. Note desktop/mobile differences if any. Refer to `tokens.json` for the full set.}

---

## Layout

{Prose: spacing scale philosophy (e.g. 4-base, 8-base, custom), grid system (columns/gutter), breakpoints. If the Figma file has no layout tokens, say so here.}

---

## Elevation & Depth

{Prose: shadow language — count of elevation steps, whether the system uses a single shadow or layered, focus-ring conventions.}

---

## Shapes

{Prose: radius philosophy (sharp, rounded, pill). Note any component-class-specific radii (small controls vs cards). Refer to `tokens.json` for the scale.}

---

## Components

{Prose: which components have explicit token mappings in `tokens.json` and what variants are covered (e.g. "button has `primary` and `outline` variants; secondary uses the `bg/subtle` token"). Do **not** list every property — that's what `tokens.json` is for.}

**Allowed component properties in `tokens.json` (design.md spec):** `backgroundColor`, `textColor`, `typography`, `rounded`, `padding`, `size`, `height`, `width`.

**Kido extension properties:** `borderColor`, `borderWidth`. Used by outlined/ghost variants. Linted as warnings by `@google/design.md`; fully accepted by `/ds-build`.

---

## Do's and Don'ts

> Capture any explicit usage rules from the Figma file annotations, library docs, or client brief. If none are present, write "None documented." — do not invent rules.

- **Do:** {usage rule}
- **Don't:** {anti-pattern}

---

## Themes

{Prose: name each non-default mode (e.g. `dark`, brand variants) and describe in 1–2 sentences how it differs from the default. Concrete token overrides live in `tokens.json` under `theme.{mode}`. Do not duplicate them here.}

---

## Gaps and Notes

Free text. Record anything that couldn't be extracted cleanly: missing tokens, unusual naming, composite variables that don't map to DTCG, stubs that need designer input.
~~~

### tokens.json template

This is the **machine source of truth** consumed by `/ds-build`. Pure DTCG tree, one top-level object per token category, identical top-level keys to the DESIGN.md front matter (`colors`, `typography`, `spacing`, `rounded`, `components`) plus Kido extensions (`breakpoint`, `grid`, `shadow`, `theme`).

~~~json
{
  "$schema": "https://design-tokens.github.io/community-group/format/",
  "$description": "Generated by /ds-extract-design. See sibling DESIGN.md for rationale.",

  "colors": {
    "brand": {
      "500": { "$type": "color", "$value": "#615fff", "$description": "Base brand color" },
      "600": { "$type": "color", "$value": "#4f39f6", "$description": "Brand one step darker" }
    },
    "neutral": {
      "0":   { "$type": "color", "$value": "#ffffff" },
      "100": { "$type": "color", "$value": "#f5f5f5" },
      "900": { "$type": "color", "$value": "#1a1c1e" }
    },
    "interactive": {
      "primary":      { "$type": "color", "$value": "{colors.brand.500}" },
      "primaryHover": { "$type": "color", "$value": "{colors.brand.600}" }
    },
    "text": {
      "default":  { "$type": "color", "$value": "{colors.neutral.900}" },
      "muted":    { "$type": "color", "$value": "{colors.neutral.600}" },
      "inverse":  { "$type": "color", "$value": "{colors.neutral.0}" }
    },
    "bg": {
      "surface":  { "$type": "color", "$value": "{colors.neutral.0}" },
      "subtle":   { "$type": "color", "$value": "{colors.neutral.100}" }
    },
    "border": {
      "default":  { "$type": "color", "$value": "{colors.neutral.200}" }
    }
  },

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
  },

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

  "shadow": {
    "sm":  { "$type": "shadow", "$value": { "color": "#0000001a", "offsetX": "0", "offsetY": "1px", "blur": "2px",  "spread": "0" } },
    "md":  { "$type": "shadow", "$value": { "color": "#0000001f", "offsetX": "0", "offsetY": "4px", "blur": "8px",  "spread": "0" } },
    "lg":  { "$type": "shadow", "$value": { "color": "#00000029", "offsetX": "0", "offsetY": "8px", "blur": "16px", "spread": "0" } },
    "ring.focused": { "$type": "shadow", "$value": { "color": "{colors.interactive.primary}", "offsetX": "0", "offsetY": "0", "blur": "0", "spread": "2px" } }
  },

  "rounded": {
    "none": { "$type": "dimension", "$value": "0px" },
    "sm":   { "$type": "dimension", "$value": "4px" },
    "md":   { "$type": "dimension", "$value": "8px" },
    "lg":   { "$type": "dimension", "$value": "12px" },
    "xl":   { "$type": "dimension", "$value": "16px" },
    "full": { "$type": "dimension", "$value": "9999px" }
  },

  "components": {
    "button-primary": {
      "backgroundColor": { "$type": "color", "$value": "{colors.interactive.primary}" },
      "textColor":       { "$type": "color", "$value": "{colors.text.inverse}" },
      "rounded":         { "$type": "dimension", "$value": "{rounded.md}" }
    },
    "button-outline": {
      "backgroundColor": "transparent",
      "textColor":       { "$type": "color", "$value": "{colors.interactive.primary}" },
      "borderColor":     { "$type": "color", "$value": "{colors.interactive.primary}" },
      "borderWidth":     { "$type": "dimension", "$value": "1px" },
      "rounded":         { "$type": "dimension", "$value": "{rounded.md}" }
    }
  },

  "theme": {
    "dark": {
      "colors": {
        "interactive": {
          "primary": { "$type": "color", "$value": "{colors.brand.400}" }
        },
        "text": {
          "default": { "$type": "color", "$value": "{colors.neutral.0}" }
        },
        "bg": {
          "surface": { "$type": "color", "$value": "{colors.neutral.900}" }
        }
      }
    }
  }
}
~~~

**Mode overrides** under `theme.{mode}` use the same top-level keys as the root tree (`colors`, `typography`, etc.) and emit **semantic overrides only** — primitives are constant across modes. Missing a mode breaks multi-theme component generation.

---

## Step 4 — Report

```
Tokens extracted.

Source:  {Figma file name}
Outputs: working/{project}/tokens.json   (DTCG, source of truth)
         working/{project}/DESIGN.md     (front matter + prose)
Modes:   light, dark
Tokens:  247 (184 primitive, 63 semantic)

Ready to run /ds-build.
```

---

## Critical Rules

**Never invent token values.** If a referenced variable is missing, note it in the "Gaps and Notes" section of DESIGN.md — do not fabricate a value in `tokens.json`.

**Preserve the client's naming when possible.** If the Figma uses `color/main/primary`, keep that path in `tokens.json`. Do not rename to Kido conventions unless explicitly requested.

**Resolve aliases in both directions.** DTCG allows `"$value": "{colors.brand.500}"` references. Preserve these in `tokens.json` — don't flatten everything to raw values. `/ds-build` can follow them.

**Capture all modes.** A file with `light` and `dark` modes needs both represented under `theme.{mode}` in `tokens.json`. Missing a mode breaks multi-theme component generation.

**Both files are per-project.** Overwrite the previous files if they exist. Do not append. The two files always travel together — never emit one without the other.

**Front matter mirrors tokens.json.** The flat values in DESIGN.md front matter must match the corresponding semantic tokens in `tokens.json`. If `tokens.json` has `colors.interactive.primary = #615fff`, the front matter `colors.primary` must be `"#615fff"`. Drift between the two is a bug.
