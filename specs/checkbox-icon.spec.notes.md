# CheckboxIcon Spec Notes

## Design Decisions

### Icon-Only Sub-Component
`CheckboxIcon` (Figma: `checkbox-1`) is the visual icon portion of a checkbox — the box/check mark itself, without a label. It is a sub-component expected to be composed inside a full `Checkbox` component that adds a label, spacing, and adequate touch area.

### Material Symbols Icons (Not Custom SVG)
The three selection states use Google Material Symbols Outlined icons baked as image assets in Figma. Each state gets a differently colored version of the same icon. In code implementation, these are better served as tinted SVG or icon font with CSS `color`/`fill` applied, rather than separate image assets per state.

Icons:
- **unchecked:** `check_box_outline_blank` (FILL@1; wght@400; GRAD@0; opsz@24)
- **checked:** `check_box`
- **indeterminate:** `indeterminate_check_box`

### Color Is Applied to the Icon, Not the Container
There is no separate background or border layer on the root container for idle/hover/pressed states. The icon SVG itself carries the fill color (including the box border for unchecked, and the filled background for checked/indeterminate). Only in `focused` state is a border added to the container element.

### Hover/Active via Overlay System
State color shifts for checked and indeterminate use overlay tokens on top of the base icon color:
- hover: `system/overlays/overlay-hover-on-color` (#0000001f) darkens the icon
- pressed: `system/fg/active` (#4f39f6) replaces primary — a more saturated/darker purple
- unchecked hover/pressed: overlay on gray icon

### Inverse Not Present
No inverse axis. This component is always on light backgrounds. If a dark-background variant is needed, it must be added to Kido DS first.

### Different Border Radius per Size
Focus ring radius follows Kido's semantic radius scale:
- size=s → `radius/semantic/tiny-elements` = 2px
- size=m → `radius/semantic/small-elements` = 4px

The icon itself has no visible border radius on its background (the radius is part of the icon SVG rendering).

### Token Derivation for Client Brands
When mapping client brand to CheckboxIcon:
- `system/fg/primary` → client brand primary (the checkbox fill color when checked)
- `system/fg/active` → darker shade of brand primary (pressed state)
- `system/fg/03` → neutral/muted gray (unchecked border — typically not brand-specific)
- `system/border/interactive/focused` → brand primary at ~50% opacity

## Known Gaps & Deviations

| Issue | Severity | Note |
|-------|----------|------|
| Both sizes below WCAG 44px touch target | Low | Expected — this is a sub-component; parent provides touch area |
| No inverse variants | Medium | If needed, must be added to DS first |
| Icons are image-baked in Figma | Info | In code, implement as tinted SVG/icon font for maintainability |

## Version History

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | 2026-04-09 | Initial spec extracted from Figma node 2545:11819 |
