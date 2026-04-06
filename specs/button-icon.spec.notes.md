# Button Icon — Spec Notes

## What This Component Is

Icon-only buttons for actions where space is limited. No label — icon communicates the action.
Kido description: "Compact buttons that display only an icon, often used for common actions where space is limited."

## Key Differences from Button

- 4 sizes (xs, s, m, l) vs 3 (s, m, l) — adds xs at 24×24px
- Square (equal width and height) vs label-driven width
- No label, no gap
- Padding is symmetric on all four sides
- `aria-label` is required since there is no visible text
- WCAG contrast requirement is 3:1 (non-text / icons) vs 4.5:1 for text
- Token set is identical to regular button except no typography tokens and adds `system/border/static/03` and `radius/semantic/small-controls`

## Verified Values (from design context)

- **size=m:** 40×40px, padding=8px, icon=24px — confirmed directly
- **Border radius:** 8px (`radius/semantic/large-controls`) — confirmed for size=m

## Unverified Values — Check in Figma

- **size=s:** 32×32px. Padding and icon size not confirmed. Estimated: 6px padding + 20px icon. Verify.
- **size=l:** 48×48px. Padding and icon size not confirmed. Estimated: 12px padding + 24px icon. Verify.
- **xs radius:** `radius/semantic/small-controls: 6` is in the variable defs. Unclear if xs uses 6px instead of 8px. Verify.
- **Spinner size for xs/s:** May differ from the 20px spinner used in m/l. Estimated: xs=16px, s=20px.

## Token Differences from Button

Present in ButtonIcon, absent in Button:
- `system/border/static/03: #e2e8f0` — likely used for outlined idle border (button uses `system/border/interactive/03-idle` which has the same value — may just be a naming difference)
- `radius/semantic/small-controls: 6` — available but usage unclear

Absent in ButtonIcon (present in Button):
- `label/m`, `label/l` — no typography (no label)
- `system/border/interactive/03-idle` — replaced by `system/border/static/03` (same value `#e2e8f0`)

## Token Derivation for Client Brand Colors

Same patterns as Button apply — only `system/bg/primary` needs to change for brand customization:

| Token | Derivation |
|-------|-----------|
| `system/border/interactive/hover` | same as primary |
| `system/fg/hover` | same as primary |
| `system/border/interactive/focused` | primary at 50% opacity |
| `components/button/outlined/bg-hover` | primary at 20% opacity |
| `components/button/outlined/bg-pressed` | primary at 38% opacity |

## History

- v1.0.0 (2026-04-06): Initial spec extracted from Kido DS, node 107:322.
