# BannerContained — spec notes

## What it is
A high-emphasis notification banner / message bar that displays a status message with optional icon, description, action button, and dismiss control. The "contained" variant uses a fully filled severity-colored background (no border, no rounded corners).

## Variant model
**One variant axis: `severity`** with five values: `neutral`, `success`, `info`, `warning`, `danger`.

Everything else (icon, description, action button, dismiss button) is a **component property** (boolean toggle or instance/text override), not a variant. This keeps the variant matrix small (5 instead of 5 × 2 × 2 × 2 × 2 = 80).

This is the same pattern Kido uses in other "feedback" components — the visual category is the only orthogonal style axis; presence/absence of optional sub-elements is a property.

## Token derivation pattern
Banner colors are paired across the `system/feedback/{severity}/*` family:

| Slot                     | Token                                  |
|--------------------------|----------------------------------------|
| Container background     | `system/feedback/{severity}/strong`    |
| Title text color         | `system/feedback/{severity}/subtle`    |
| Description text color   | `system/feedback/{severity}/subtle`    |
| Action label / icon      | `system/fg/inverse` (white)            |

Mapping a client brand:
- If the client provides their own status colors, map each severity to the closest brand status color → derive `subtle` (light tint) and `strong` (saturated) variants from it.
- If the client only provides a primary brand color, keep neutral/success/info/warning/danger as Kido defaults but tune `info` toward the brand hue.

## Anatomy (visual)
```
[Container — severity strong bg, padding 16, gap 16, sharp corners]
  [left content — gap 8]
    [Slot — icon, optional via `content`]
    [texts — gap 2]
      [Text       — body/l-medium]
      [Description — body/l-regular, optional via `description1`]
  [buttons-container — flex 1, justify-end, gap 4]
    [button       — action label, optional via `action`, radius 8, 32px tall]
    [button-icon  — close, optional via `dismiss`, radius 8, padding 8]
```

## Known deviations / things to flag
- **Touch targets:** Both the inline action button (32px) and the dismiss icon button (~32px) are below the WCAG 44×44 recommendation. Acceptable in a banner context where the primary affordance is the message itself.
- **No `inverse` axis:** Because the container is always severity-colored and the text uses `subtle` tints, an `inverse` axis is not needed. If a client requests light banners, that should be a separate `banner-outlined` or `banner-soft` component, not an `inverse` flag here.
- **Container is sharp (radius 0):** Intentional. The contained banner is meant to span full-bleed sections (page top, section dividers). If a client wants rounded banners, treat that as a token override (`radius/semantic/sharp` → another radius), not a structural change.
- **Misprint in Figma description:** The Figma component description contains a Hebrew-keyboard-misprint placeholder ("נשממקר…") — ignore; not a real misprint to surface.

## Generation defaults
Generate **5 variants** by default — one per severity. All component-property booleans default to `true` (icon, description, action, dismiss all visible) so the canonical variant shows the full anatomy. Designers will toggle properties off in instances as needed.

## Version history
- **1.0.0** — 2026-04-30 — Initial spec extracted from node `2803:91656`.
