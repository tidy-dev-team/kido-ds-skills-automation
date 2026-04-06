# Button — Spec Notes

## Design Decisions

- **Size S (32px)** is below the WCAG 44px touch target minimum. Known Kido decision — flag to clients using size=s in mobile contexts.
- **Ghost idle text** uses `system/fg/primary` (#615fff), not `system/fg/01` (#0f172b, dark text). Intentional — ghost buttons are interactive, not neutral.
- **Focus ring** is a border change on the Container layer (width increases to 2px, color = `system/border/interactive/focused`), not an external offset ring.
- **Size M and L share the same typography** (`label/l`, 16px). Only size S steps down to `label/m` (14px).
- **Loading state reduces paddingY** to accommodate the spinner visually: s→8px, m→10px, l→14px.
- **Inverse variants** use entirely separate token sets — not derivable from inverse=no by opacity alone.

## Token Derivation for Client Brand Colors

When a client's primary color replaces `system/bg/primary`, the following tokens should be re-derived:

| Token | Derivation |
|-------|-----------|
| `system/border/interactive/hover` | same as primary |
| `system/fg/hover` | same as primary |
| `system/border/interactive/focused` | primary at 50% opacity |
| `components/button/outlined/bg-hover` | primary at 20% opacity |
| `components/button/outlined/bg-pressed` | primary at 38% opacity |

All other tokens are Kido system values and do not change with brand color.

## Known Gaps

- **Inverse variants** are almost always stubbed in client projects — client dark-background designs are rarely provided upfront. Expected behavior.
- **PaddingY** values are derived from `(height - font-size) / 2` and should be verified directly in Figma if pixel-perfect precision is needed.
- **Spinner asset** for loading state references the Kido Spinner component. Ensure it is available in the client's Figma file before generation.

## History

- v1.0.0 (2026-04-05): Initial spec extracted from Kido DS, node 98:899.
