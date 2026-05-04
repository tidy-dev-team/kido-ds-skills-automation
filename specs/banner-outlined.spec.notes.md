# BannerOutlined — spec notes

## What it is
A medium-emphasis notification banner / message bar. Same anatomy as `banner-contained`, but the surface treatment is "outlined": a 1px severity-colored border around either a white background (`bgColor=default`) or a light severity-tinted background (`bgColor=feedback`). Text is dark (`system/fg/01`) regardless of severity — color semantics are carried by the border (and optional fill), not by the text.

## Variant model
**Two variant axes:**
- `severity` — `neutral | success | info | warning | danger`
- `bgColor` — `default` (white) | `feedback` (severity-subtle tint)

5 × 2 = **10 variants** by default.

Optional sub-elements (icon, description, action button, dismiss button) remain **component properties** (boolean / instance / text overrides), not variants — same as `banner-contained`. This keeps the matrix at 10 instead of 10 × 2 × 2 × 2 × 2 = 160.

## Token derivation pattern
Outlined banners pair the `system/feedback/{severity}/*` family across border + (optional) fill, with neutral foreground:

| Slot                          | Token                                                     |
|-------------------------------|-----------------------------------------------------------|
| Container border              | `system/feedback/{severity}/medium`                       |
| Container bg (`default`)      | `system/bg/03` (white)                                    |
| Container bg (`feedback`)     | `system/feedback/{severity}/subtle`                       |
| Title text color              | `system/fg/01` (#0f172b)                                  |
| Description text color        | `system/fg/01`                                            |
| Action label / dismiss icon   | `system/fg/01`                                            |

Mapping a client brand:
- Map each severity to the closest brand status color → derive `medium` (saturated border) and `subtle` (light tint) variants.
- Foreground stays neutral dark — do not tint text by severity in the outlined variant. If the client's `fg/01` is a true black or near-black neutral, that is the right value.
- If the client only ships a primary brand color, keep neutral/success/info/warning/danger as Kido defaults; only `info` should be tuned toward the brand hue.

## Anatomy (visual)
```
[Container — 1px {severity}/medium border, bg=white|{severity}/subtle, padding 16, gap 16, sharp corners]
  [left content — gap 8]
    [Slot — icon, optional via `content`]
    [texts — gap 2]
      [Text       — body/l-medium, fg/01]
      [Description — body/l-regular, fg/01, optional via `description1`]
  [buttons-container — flex 1, justify-end, gap 4]
    [button       — action label, optional via `action`, ghost, radius 8, 32px tall]
    [button-icon  — close, optional via `dismiss`, radius 8, padding 8]
```

## Outlined vs. contained — when to use which
- **Contained** — high-emphasis, full-bleed page/section banners; severity dominates the color field.
- **Outlined (`bgColor=default`)** — lowest-emphasis inline alerts; sits on white surfaces without overpowering surrounding content.
- **Outlined (`bgColor=feedback`)** — medium-emphasis; reads as a severity-tinted card. Common default for in-page messaging.

## Known deviations / things to flag
- **Touch targets:** Same as `banner-contained` — inline action button (32px) and dismiss icon button (~32px) are below the WCAG 44×44 recommendation.
- **Color-only signaling risk:** With dark neutral text, severity is communicated by the border color (and, optionally, the tinted fill). Always keep the leading icon slot populated, or pair with text that names the severity, to satisfy WCAG 1.4.1 (Use of Color).
- **Container is sharp (radius 0):** Intentional, mirrors `banner-contained`. If a client wants rounded outlined banners, treat as a token override (`radius/semantic/sharp` → another radius), not a structural change.
- **Misprint in Figma description:** The Figma component description contains a Hebrew-keyboard-misprint placeholder ("נשממקר…") — ignore.

## Generation defaults
Generate **10 variants** by default — 5 severities × 2 bg colors. All component-property booleans default to `true` (icon, description, action, dismiss all visible) so the canonical variant shows the full anatomy. Designers will toggle properties off in instances as needed.

## Version history
- **1.0.0** — 2026-04-30 — Initial spec extracted from node `2873:51786`.
