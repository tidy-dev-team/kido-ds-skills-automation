---
name: claroty
version: alpha
description: Claroty DS — dual-vertical (Industrial + Healthcare) design system with light/dark modes, built on Montserrat and a purple/indigo brand palette.
colors:
  primary: "#793AAF"
  primaryHover: "#8E4EC6"
  surface: "#FFFFFF"
  onSurface: "#1A1523"
  border: "#DCDBDD"
  error: "NEEDS_VALUE"
  warning: "NEEDS_VALUE"
  success: "NEEDS_VALUE"
typography:
  body:
    fontFamily: "Montserrat"
    fontSize: "14px"
    fontWeight: 400
    lineHeight: "20px"
  heading:
    fontFamily: "Montserrat"
    fontSize: "24px"
    fontWeight: 700
    lineHeight: "32px"
rounded:
  sm: "4px"
  md: "8px"
  lg: "12px"
  full: "9999px"
spacing:
  xs: "12px"
  sm: "16px"
  md: "20px"
  lg: "24px"
  xl: "32px"
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.surface}"
    rounded: "{rounded.md}"
    padding: "{spacing.xs} {spacing.sm}"
---

# DESIGN.md

> Extracted from: **Kido ↔ Claroty DS — All Foundations** + **Colors — Layer 3 — Kido ↔ Claroty DS — Common Usages**
> Source: `https://www.figma.com/design/BHHyTAD7I1nszu4KxzNo4B/-Kido-%3C%3E-Claroty-DS--All-Foundations` + `https://www.figma.com/design/vhaKrP4eXlIvOO4YA0Tuuv/Colors---Layer-3----Kido-%3C%3E-Claroty-DS---Common-Usages`
> Extraction date: 2026-04-22
> Modes: Industrial Light (default), Industrial Dark, Healthcare Light, Healthcare Dark
> Token count: ~120 dimension/typography tokens (foundations) + 71 semantic color variables × 4 modes

Claroty is an industrial cybersecurity and healthcare IoT security platform. Its design system supports two distinct product verticals — **Industrial** (purple brand, `#793AAF`) and **Healthcare** (indigo brand, `#3451B2`) — each with light and dark variants, yielding four fully-specified theme modes. The foundations are extracted across two Figma files: all dimension/typography/shadow primitives live in the All Foundations file; semantic color variables live in the Colors Layer 3 file as aliases resolved across the primitive library chain.

---

## Overview

Claroty DS uses **Montserrat** as its sole typeface, conveying technical precision and institutional trust. The Industrial vertical anchors around a deep purple (`#793AAF`) that signals authority in OT security; the Healthcare vertical shifts to an accessible indigo (`#3451B2`) appropriate for clinical environments. Spacing follows a step-based scale (0 → 64px in 13 steps), radius is conservative (0–16px + full pill), and elevation is expressed through subtle layered box-shadows using the foreground shadow color rather than pure black. The system supports a full dark mode for both verticals with semantically-named surface/text/border tokens rather than raw primitives, making theming a clean override set.

---

## Colors

### Primitives

```json
{
  "color": {
    "basic": {
      "white": { "$type": "color", "$value": "#FFFFFF", "$description": "Pure white — used across all four modes" },
      "black": { "$type": "color", "$value": "#000000", "$description": "Pure black" }
    }
  }
}
```

> Note: The Layer 3 Colors file contains only semantic (alias) variables. Primitive palettes (Layer 1/2) live in external library files and are resolved at alias-walk time. The hex values below are the fully-resolved leaf values for Industrial Light (default). See the Themes section for per-mode overrides.

### Semantic — Industrial Light (default)

```json
{
  "color": {
    "brand": {
      "default":    { "$type": "color", "$value": "#793AAF", "$description": "Primary brand — Industrial vertical" },
      "secondary":  { "$type": "color", "$value": "#E4C64E", "$description": "Secondary brand accent — Industrial" },
      "disabled":   { "$type": "color", "$value": "#D3B4ED", "$description": "Brand color at disabled state" }
    },
    "background": {
      "page":            { "$type": "color", "$value": "#F8FAFF", "$description": "Page-level background" },
      "card":            { "$type": "color", "$value": "#FFFFFF",  "$description": "Card / panel surface" },
      "popup":           { "$type": "color", "$value": "#FFFFFF",  "$description": "Modal / popover surface" },
      "box":             { "$type": "color", "$value": "#FFFFFF",  "$description": "Box / inset surface" },
      "header-dialog":   { "$type": "color", "$value": "#F4F2F4", "$description": "Dialog header background" },
      "disabled":        { "$type": "color", "$value": "#F4F2F4", "$description": "Disabled element background" },
      "brand-dark":      { "$type": "color", "$value": "#3A1E48", "$description": "Dark brand splash background (nav, hero)" },
      "primary-active":  { "$type": "color", "$value": "#2B0E44", "$description": "Active/selected brand background" }
    },
    "hover": {
      "default":     { "$type": "color", "$value": "#F9F1FE", "$description": "Row / item hover background" },
      "on-primary":  { "$type": "color", "$value": "#8E4EC6", "$description": "Hover on primary-filled element" }
    },
    "text": {
      "1":        { "$type": "color", "$value": "#1A1523", "$description": "Primary text" },
      "2":        { "$type": "color", "$value": "#6F6E77", "$description": "Secondary / muted text" },
      "3":        { "$type": "color", "$value": "#C8C7CB", "$description": "Placeholder / disabled text" },
      "inverse":  { "$type": "color", "$value": "#F7ECFC", "$description": "Text on dark/brand backgrounds" },
      "disabled": { "$type": "color", "$value": "#C8C7CB", "$description": "Disabled text" },
      "brand":    { "$type": "color", "$value": "#793AAF", "$description": "Brand-colored text" },
      "link":     { "$type": "color", "$value": "#8E4EC6", "$description": "Hyperlink text color" }
    },
    "icon": {
      "default":  { "$type": "color", "$value": "#86848D", "$description": "Default icon fill" },
      "hover":    { "$type": "color", "$value": "#793AAF", "$description": "Icon on hover" },
      "disabled": { "$type": "color", "$value": "#C8C7CB", "$description": "Disabled icon fill" }
    },
    "border": {
      "default": { "$type": "color", "$value": "#DCDBDD", "$description": "Default border (hairline)" },
      "strong":  { "$type": "color", "$value": "#C8C7CB", "$description": "Strong border (divider)" },
      "3":       { "$type": "color", "$value": "#908E96", "$description": "Emphasis border (active, focus frame)" },
      "brand":   { "$type": "color", "$value": "#793AAF", "$description": "Brand-colored border" }
    },
    "shadow": {
      "color": { "$type": "color", "$value": "#E9E8EA", "$description": "Shadow tint — used in all elevation tokens" }
    },
    "scrollbar": {
      "default": { "$type": "color", "$value": "#6F6E77", "$description": "Scrollbar thumb" }
    },
    "gradient": {
      "brand-50": { "$type": "color", "$value": "#793AAF80", "$description": "Brand at 50% opacity — gradient start" },
      "brand-0":  { "$type": "color", "$value": "#793AAF00", "$description": "Brand at 0% opacity — gradient end" }
    },
    "level": {
      "critical": {
        "text":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "main":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary":            { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary-background": { "$type": "color", "$value": "NEEDS_VALUE" },
        "background":           { "$type": "color", "$value": "NEEDS_VALUE" }
      },
      "high": {
        "text":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "main":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary":            { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary-background": { "$type": "color", "$value": "NEEDS_VALUE" },
        "background":           { "$type": "color", "$value": "NEEDS_VALUE" }
      },
      "medium": {
        "text":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "main":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary":            { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary-background": { "$type": "color", "$value": "NEEDS_VALUE" },
        "background":           { "$type": "color", "$value": "NEEDS_VALUE" }
      },
      "low": {
        "text":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "main":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary":            { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary-background": { "$type": "color", "$value": "NEEDS_VALUE" },
        "background":           { "$type": "color", "$value": "NEEDS_VALUE" }
      },
      "very-low": {
        "text":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "main":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary":            { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary-background": { "$type": "color", "$value": "NEEDS_VALUE" },
        "background":           { "$type": "color", "$value": "NEEDS_VALUE" }
      },
      "gray": {
        "text":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "main":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary":            { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary-background": { "$type": "color", "$value": "NEEDS_VALUE" },
        "background":           { "$type": "color", "$value": "NEEDS_VALUE" }
      },
      "info": {
        "text":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "main":                 { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary":            { "$type": "color", "$value": "NEEDS_VALUE" },
        "secondary-background": { "$type": "color", "$value": "NEEDS_VALUE" },
        "background":           { "$type": "color", "$value": "NEEDS_VALUE" }
      }
    }
  }
}
```

---

## Typography

```json
{
  "typography": {
    "scale": {
      "desktop": {
        "xs":  { "$type": "dimension", "$value": "10px" },
        "sm":  { "$type": "dimension", "$value": "12px" },
        "md":  { "$type": "dimension", "$value": "14px" },
        "lg":  { "$type": "dimension", "$value": "16px" },
        "xl":  { "$type": "dimension", "$value": "18px" },
        "2xl": { "$type": "dimension", "$value": "20px" },
        "3xl": { "$type": "dimension", "$value": "24px" },
        "4xl": { "$type": "dimension", "$value": "32px" },
        "5xl": { "$type": "dimension", "$value": "96px" }
      },
      "mobile": {
        "xs":  { "$type": "dimension", "$value": "12px" },
        "sm":  { "$type": "dimension", "$value": "14px" },
        "md":  { "$type": "dimension", "$value": "16px" },
        "lg":  { "$type": "dimension", "$value": "18px" },
        "xl":  { "$type": "dimension", "$value": "20px" },
        "2xl": { "$type": "dimension", "$value": "24px" }
      }
    },
    "lineHeight": {
      "desktop": {
        "xs":  { "$type": "dimension", "$value": "14px" },
        "sm":  { "$type": "dimension", "$value": "16px" },
        "md":  { "$type": "dimension", "$value": "20px" },
        "lg":  { "$type": "dimension", "$value": "24px" },
        "xl":  { "$type": "dimension", "$value": "28px" },
        "2xl": { "$type": "dimension", "$value": "28px" },
        "3xl": { "$type": "dimension", "$value": "32px" },
        "4xl": { "$type": "dimension", "$value": "96px" }
      },
      "mobile": {
        "xs":  { "$type": "dimension", "$value": "16px" },
        "sm":  { "$type": "dimension", "$value": "20px" },
        "md":  { "$type": "dimension", "$value": "24px" },
        "lg":  { "$type": "dimension", "$value": "28px" },
        "xl":  { "$type": "dimension", "$value": "32px" }
      }
    },
    "display": {
      "lg": {
        "$type": "typography",
        "$value": { "fontFamily": "Montserrat", "fontSize": "96px", "fontWeight": 700, "lineHeight": "96px", "letterSpacing": "0" },
        "$description": "Largest display — splash / hero"
      },
      "md": {
        "$type": "typography",
        "$value": { "fontFamily": "Montserrat", "fontSize": "32px", "fontWeight": 700, "lineHeight": "40px", "letterSpacing": "0" }
      }
    },
    "heading": {
      "desktop": {
        "h1": { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "24px", "fontWeight": 700, "lineHeight": "32px", "letterSpacing": "0" } },
        "h2": { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "20px", "fontWeight": 700, "lineHeight": "28px", "letterSpacing": "0" } },
        "h3": { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "18px", "fontWeight": 600, "lineHeight": "28px", "letterSpacing": "0" } }
      },
      "mobile": {
        "h1": { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "20px", "fontWeight": 700, "lineHeight": "28px", "letterSpacing": "0" } },
        "h2": { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "18px", "fontWeight": 700, "lineHeight": "24px", "letterSpacing": "0" } },
        "h3": { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "16px", "fontWeight": 600, "lineHeight": "24px", "letterSpacing": "0" } }
      }
    },
    "body": {
      "lg": {
        "regular":  { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "16px", "fontWeight": 400, "lineHeight": "24px", "letterSpacing": "0" } },
        "medium":   { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "16px", "fontWeight": 500, "lineHeight": "24px", "letterSpacing": "0" } },
        "semibold": { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "16px", "fontWeight": 600, "lineHeight": "24px", "letterSpacing": "0" } },
        "bold":     { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "16px", "fontWeight": 700, "lineHeight": "24px", "letterSpacing": "0" } },
        "italic":   { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "16px", "fontWeight": 400, "lineHeight": "24px", "letterSpacing": "0" } }
      },
      "md": {
        "regular":  { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "14px", "fontWeight": 400, "lineHeight": "20px", "letterSpacing": "0" } },
        "medium":   { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "14px", "fontWeight": 500, "lineHeight": "20px", "letterSpacing": "0" } },
        "semibold": { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "14px", "fontWeight": 600, "lineHeight": "20px", "letterSpacing": "0" } },
        "bold":     { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "14px", "fontWeight": 700, "lineHeight": "20px", "letterSpacing": "0" } },
        "italic":   { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "14px", "fontWeight": 400, "lineHeight": "20px", "letterSpacing": "0" } }
      },
      "sm": {
        "regular":  { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "12px", "fontWeight": 400, "lineHeight": "16px", "letterSpacing": "0" } },
        "medium":   { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "12px", "fontWeight": 500, "lineHeight": "16px", "letterSpacing": "0" } },
        "semibold": { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "12px", "fontWeight": 600, "lineHeight": "16px", "letterSpacing": "0" } },
        "bold":     { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "12px", "fontWeight": 700, "lineHeight": "16px", "letterSpacing": "0" } },
        "italic":   { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "12px", "fontWeight": 400, "lineHeight": "16px", "letterSpacing": "0" } }
      }
    },
    "caption": {
      "regular":  { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "10px", "fontWeight": 400, "lineHeight": "14px", "letterSpacing": "0" } },
      "semibold": { "$type": "typography", "$value": { "fontFamily": "Montserrat", "fontSize": "10px", "fontWeight": 600, "lineHeight": "14px", "letterSpacing": "0" } }
    }
  }
}
```

> Desktop vs mobile: the font size scale differs (desktop xs=10px → 5xl=96px; mobile xs=12px → 2xl=24px). Both share the same weight/style variants. Exact px values for heading h1–h3 are derived from scale tokens; verify against the Figma text style panel if precise values are needed (see Gaps).

---

## Layout

```json
{
  "layout": {
    "spacing": {
      "none": { "$type": "dimension", "$value": "0px" },
      "4xs":  { "$type": "dimension", "$value": "2px" },
      "3xs":  { "$type": "dimension", "$value": "4px" },
      "2xs":  { "$type": "dimension", "$value": "8px" },
      "xs":   { "$type": "dimension", "$value": "12px" },
      "sm":   { "$type": "dimension", "$value": "16px" },
      "md":   { "$type": "dimension", "$value": "20px" },
      "lg":   { "$type": "dimension", "$value": "24px" },
      "xl":   { "$type": "dimension", "$value": "32px" },
      "2xl":  { "$type": "dimension", "$value": "40px" },
      "3xl":  { "$type": "dimension", "$value": "48px" },
      "4xl":  { "$type": "dimension", "$value": "64px" }
    }
  }
}
```

> No explicit grid/breakpoint tokens were found in the extracted foundations. Use Claroty engineering defaults or extract from a layout-spec Figma file if one exists.

---

## Elevation & Depth

```json
{
  "shadow": {
    "sm": {
      "$type": "shadow",
      "$value": { "color": "{color.shadow.color}", "offsetX": "0", "offsetY": "2px", "blur": "12px", "spread": "0" },
      "$description": "Subtle elevation — cards, dropdowns"
    },
    "md": {
      "$type": "shadow",
      "$value": { "color": "{color.shadow.color}", "offsetX": "0", "offsetY": "4px", "blur": "24px", "spread": "0" },
      "$description": "Medium elevation — popovers, side panels"
    },
    "lg": {
      "$type": "shadow",
      "$value": { "color": "{color.shadow.color}", "offsetX": "0", "offsetY": "8px", "blur": "36px", "spread": "0" },
      "$description": "High elevation — modals, full-screen overlays"
    },
    "ring-focused": {
      "$type": "shadow",
      "$value": { "color": "{color.brand.default}", "offsetX": "0", "offsetY": "0", "blur": "0", "spread": "2px" },
      "$description": "Focus ring — keyboard navigation indicator"
    },
    "ring-blinking": {
      "$type": "shadow",
      "$value": { "color": "{color.brand.default}", "offsetX": "0", "offsetY": "0", "blur": "6px", "spread": "0" },
      "$description": "Blinking ring — active/alert state indicator"
    }
  }
}
```

> Shadow color uses `color.shadow.color` (Industrial Light: `#E9E8EA`). In dark modes it resolves to a dark neutral — intentional, using the foreground palette color not pure black. Figma effect style names: `shadow/sm`, `shadow/md`, `shadow/lg`, `ring-indicator/focused`, `ring-indicator/blinking`.

---

## Shapes

```json
{
  "radius": {
    "none": { "$type": "dimension", "$value": "0px" },
    "xs":   { "$type": "dimension", "$value": "2px" },
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

> Flat component token summary for the design.md CLI `components:` front matter key. Full per-component slot mappings live in `token-map.json` (generated by `/ds-build`).

```json
{
  "component": {
    "button-primary": {
      "backgroundColor": { "$type": "color", "$value": "{color.brand.default}" },
      "textColor":       { "$type": "color", "$value": "{color.basic.white}" },
      "hoverBackground": { "$type": "color", "$value": "{color.hover.on-primary}" },
      "rounded":         { "$type": "dimension", "$value": "{radius.md}" },
      "paddingV":        { "$type": "dimension", "$value": "{layout.spacing.xs}" },
      "paddingH":        { "$type": "dimension", "$value": "{layout.spacing.sm}" }
    },
    "button-outline": {
      "backgroundColor": { "$type": "color", "$value": "transparent" },
      "textColor":       { "$type": "color", "$value": "{color.brand.default}" },
      "borderColor":     { "$type": "color", "$value": "{color.border.brand}" },
      "hoverBackground": { "$type": "color", "$value": "{color.hover.default}" },
      "rounded":         { "$type": "dimension", "$value": "{radius.md}" }
    },
    "button-ghost": {
      "backgroundColor": { "$type": "color", "$value": "transparent" },
      "textColor":       { "$type": "color", "$value": "{color.brand.default}" },
      "hoverBackground": { "$type": "color", "$value": "{color.hover.default}" },
      "rounded":         { "$type": "dimension", "$value": "{radius.md}" }
    },
    "input": {
      "backgroundColor":  { "$type": "color", "$value": "{color.background.card}" },
      "borderColor":      { "$type": "color", "$value": "{color.border.default}" },
      "focusBorderColor": { "$type": "color", "$value": "{color.brand.default}" },
      "textColor":        { "$type": "color", "$value": "{color.text.1}" },
      "placeholderColor": { "$type": "color", "$value": "{color.text.3}" },
      "rounded":          { "$type": "dimension", "$value": "{radius.sm}" }
    },
    "card": {
      "backgroundColor": { "$type": "color", "$value": "{color.background.card}" },
      "borderColor":     { "$type": "color", "$value": "{color.border.default}" },
      "shadow":          { "$type": "shadow", "$value": "{shadow.sm}" },
      "rounded":         { "$type": "dimension", "$value": "{radius.md}" }
    }
  }
}
```

---

## Do's and Don'ts

- **Do:** Use `color.brand.default` (not a hardcoded hex) for all brand-colored elements — the token resolves differently for Industrial vs Healthcare verticals.
- **Do:** Apply `ring-focused` shadow on all interactive elements for keyboard accessibility.
- **Do:** Use `color.background.brand-dark` (`#3A1E48` Industrial / `#1C274F` Healthcare) for dark branded headers and navigation — not a generic neutral dark.
- **Don't:** Use the `Internal design colors` collection tokens in production — those are DS tooling colors only.
- **Don't:** Use `Graph Colors` or `Label Colors` tokens for UI chrome — they are scoped to data visualizations and tag/badge taxonomy respectively.
- **Don't:** Flatten the shadow color to `rgba(0,0,0,0.1)` — Claroty shadows intentionally use the foreground palette color (`color.shadow.color`) to match the mode.

---

## Themes

> Semantic token overrides only. Primitives (radius, spacing, typography scale) stay constant across all modes.

### industrial-dark

```json
{
  "theme": {
    "industrial-dark": {
      "color": {
        "brand": {
          "default":   { "$type": "color", "$value": "#BF7AF0" },
          "secondary": { "$type": "color", "$value": "#E3C345" },
          "disabled":  { "$type": "color", "$value": "#5F2D84" }
        },
        "background": {
          "page":           { "$type": "color", "$value": "#1C1C1F" },
          "card":           { "$type": "color", "$value": "#28282C" },
          "popup":          { "$type": "color", "$value": "#232326" },
          "box":            { "$type": "color", "$value": "#504F57" },
          "header-dialog":  { "$type": "color", "$value": "#232326" },
          "disabled":       { "$type": "color", "$value": "#232326" },
          "brand-dark":     { "$type": "color", "$value": "#3A1E48" },
          "primary-active": { "$type": "color", "$value": "#F7ECFC" }
        },
        "hover": {
          "default":    { "$type": "color", "$value": "#301A3A" },
          "on-primary": { "$type": "color", "$value": "#8E4EC6" }
        },
        "text": {
          "1":        { "$type": "color", "$value": "#EDEDEF" },
          "2":        { "$type": "color", "$value": "#A09FA6" },
          "3":        { "$type": "color", "$value": "#504F57" },
          "inverse":  { "$type": "color", "$value": "#F7ECFC" },
          "disabled": { "$type": "color", "$value": "#4C5155" },
          "brand":    { "$type": "color", "$value": "#793AAF" },
          "link":     { "$type": "color", "$value": "#8E4EC6" }
        },
        "icon": {
          "default":  { "$type": "color", "$value": "#7E7D86" },
          "hover":    { "$type": "color", "$value": "#793AAF" },
          "disabled": { "$type": "color", "$value": "#504F57" }
        },
        "border": {
          "default": { "$type": "color", "$value": "#3E3E44" },
          "strong":  { "$type": "color", "$value": "#504F57" },
          "3":       { "$type": "color", "$value": "#706F78" },
          "brand":   { "$type": "color", "$value": "#793AAF" }
        },
        "shadow": {
          "color": { "$type": "color", "$value": "#1A1523" }
        },
        "scrollbar": {
          "default": { "$type": "color", "$value": "#A09FA6" }
        },
        "gradient": {
          "brand-50": { "$type": "color", "$value": "#BF7AF080" },
          "brand-0":  { "$type": "color", "$value": "#BF7AF000" }
        }
      }
    }
  }
}
```

### healthcare-light

```json
{
  "theme": {
    "healthcare-light": {
      "color": {
        "brand": {
          "default":   { "$type": "color", "$value": "#3451B2" },
          "secondary": { "$type": "color", "$value": "#FA934E" },
          "disabled":  { "$type": "color", "$value": "#AEC0F5" }
        },
        "background": {
          "page":           { "$type": "color", "$value": "#F8FAFF" },
          "card":           { "$type": "color", "$value": "#FFFFFF" },
          "popup":          { "$type": "color", "$value": "#FFFFFF" },
          "box":            { "$type": "color", "$value": "#FFFFFF" },
          "header-dialog":  { "$type": "color", "$value": "#F1F3F5" },
          "disabled":       { "$type": "color", "$value": "#F1F3F5" },
          "brand-dark":     { "$type": "color", "$value": "#1C274F" },
          "primary-active": { "$type": "color", "$value": "#101D46" }
        },
        "hover": {
          "default":    { "$type": "color", "$value": "#F0F4FF" },
          "on-primary": { "$type": "color", "$value": "#3E63DD" }
        },
        "text": {
          "1":        { "$type": "color", "$value": "#11181C" },
          "2":        { "$type": "color", "$value": "#687076" },
          "3":        { "$type": "color", "$value": "#C1C8CD" },
          "inverse":  { "$type": "color", "$value": "#EEF1FD" },
          "disabled": { "$type": "color", "$value": "#C1C8CD" },
          "brand":    { "$type": "color", "$value": "#793AAF" },
          "link":     { "$type": "color", "$value": "#3E63DD" }
        },
        "icon": {
          "default":  { "$type": "color", "$value": "#7E868C" },
          "hover":    { "$type": "color", "$value": "#793AAF" },
          "disabled": { "$type": "color", "$value": "#C1C8CD" }
        },
        "border": {
          "default": { "$type": "color", "$value": "#D7DBDF" },
          "strong":  { "$type": "color", "$value": "#C1C8CD" },
          "3":       { "$type": "color", "$value": "#889096" },
          "brand":   { "$type": "color", "$value": "#793AAF" }
        },
        "shadow": {
          "color": { "$type": "color", "$value": "#E6E8EB" }
        },
        "scrollbar": {
          "default": { "$type": "color", "$value": "#687076" }
        },
        "gradient": {
          "brand-50": { "$type": "color", "$value": "#3451B280" },
          "brand-0":  { "$type": "color", "$value": "#3451B200" }
        }
      }
    }
  }
}
```

### healthcare-dark

```json
{
  "theme": {
    "healthcare-dark": {
      "color": {
        "brand": {
          "default":   { "$type": "color", "$value": "#849DFF" },
          "secondary": { "$type": "color", "$value": "#FF802B" },
          "disabled":  { "$type": "color", "$value": "#273E89" }
        },
        "background": {
          "page":           { "$type": "color", "$value": "#1A1D1E" },
          "card":           { "$type": "color", "$value": "#26292B" },
          "popup":          { "$type": "color", "$value": "#202425" },
          "box":            { "$type": "color", "$value": "#4C5155" },
          "header-dialog":  { "$type": "color", "$value": "#202425" },
          "disabled":       { "$type": "color", "$value": "#202425" },
          "brand-dark":     { "$type": "color", "$value": "#1C274F" },
          "primary-active": { "$type": "color", "$value": "#EEF1FD" }
        },
        "hover": {
          "default":    { "$type": "color", "$value": "#192140" },
          "on-primary": { "$type": "color", "$value": "#3E63DD" }
        },
        "text": {
          "1":        { "$type": "color", "$value": "#ECEDEE" },
          "2":        { "$type": "color", "$value": "#9BA1A6" },
          "3":        { "$type": "color", "$value": "#4C5155" },
          "inverse":  { "$type": "color", "$value": "#EEF1FD" },
          "disabled": { "$type": "color", "$value": "#4C5155" },
          "brand":    { "$type": "color", "$value": "#793AAF" },
          "link":     { "$type": "color", "$value": "#3E63DD" }
        },
        "icon": {
          "default":  { "$type": "color", "$value": "#787F85" },
          "hover":    { "$type": "color", "$value": "#793AAF" },
          "disabled": { "$type": "color", "$value": "#4C5155" }
        },
        "border": {
          "default": { "$type": "color", "$value": "#3A3F42" },
          "strong":  { "$type": "color", "$value": "#4C5155" },
          "3":       { "$type": "color", "$value": "#697177" },
          "brand":   { "$type": "color", "$value": "#793AAF" }
        },
        "shadow": {
          "color": { "$type": "color", "$value": "#11181C" }
        },
        "scrollbar": {
          "default": { "$type": "color", "$value": "#9BA1A6" }
        },
        "gradient": {
          "brand-50": { "$type": "color", "$value": "#849DFF80" },
          "brand-0":  { "$type": "color", "$value": "#849DFF00" }
        }
      }
    }
  }
}
```

---

## Gaps and Notes

**Level colors not resolved** — The `Vertical` collection contains 7 severity-level groups (Critical, High, Medium, Low, Very Low, Gray, Info), each with 5 variants (text, main, secondary, secondary-background, background), totalling 35 color variables per mode (140 values across 4 modes). These are cross-file aliases resolving to Level Colors primitive libraries that were not available in the current session. All are marked `NEEDS_VALUE`. Run a targeted alias-resolution pass against the `Level Colors` subcollection in a follow-up session. These are required for Badge, Alert, and vulnerability table components.

**Front matter error/warning/success fields** — Marked `NEEDS_VALUE` for the same reason; they map to `level.critical.main`, `level.medium.main`, and `level.info.main` respectively once resolved.

**Label Colors collection** (44 variables) — Provides tag/badge background and text colors for Claroty's taxonomy labels (asset categories, protocol labels, etc.). Not included in this extraction. Relevant for Badge and Tag components — extract separately when those components are in scope.

**Graph Colors collection** (40 variables) — Data-visualization palette (bar chart series, line chart series, heatmap scales). Not included. Relevant for chart/dashboard components only.

**Internal design colors collection** (21 variables) — DS tooling and annotation colors. Not for production use; excluded intentionally.

**Typography exact px values** — The Figma text styles (`desktop/headline/h1`, `desktop/body/md`, etc.) were identified by name but exact CSS values were not inspected in this session. The typography tokens above are derived from the font-size scale tokens (xs=10px → 5xl=96px). Verify exact px/weight values against the Figma text style panel before finalising component specs, particularly for heading h1–h3 tiers.

**Grid and breakpoints** — No layout grid or breakpoint tokens were found in the extracted variable collections. Use Claroty engineering defaults or extract from a separate layout-spec Figma file if one exists.

**`color.text.brand` is fixed at `#793AAF`** across all 4 modes — this appears intentional in the Figma file (brand text always uses the Industrial purple, regardless of vertical or mode). Verify with the Claroty DS team before overriding per-vertical.

**`color.border.brand` is fixed at `#793AAF`** across all 4 modes — same observation as above. Likely intentional for cross-vertical consistency in brand-accented UI chrome.
