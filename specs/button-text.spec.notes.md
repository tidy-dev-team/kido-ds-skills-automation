# ButtonText Spec Notes

## Design Decisions

### No Container
ButtonText has no background, border, or border-radius. It is a pure text row with optional leading/trailing icons. This makes it visually lighter than ghost-type Button but also means it has no hover background overlay — the state change is communicated by text color alone.

### Only 2 Sizes (s, m)
Unlike Button (s/m/l), ButtonText only has size=s (12px) and size=m (16px). There is no size=l variant in the Figma component set.

### No Focused or Loading States
The Figma component set defines only: idle, hover, pressed, disabled. There is no `focused` state (unlike Button which has a focus ring) and no `loading` state. If a focus state is needed for accessibility, it must be implemented in code without a Figma reference — recommend using an underline or browser default outline.

### Typography Tokens: body/ not label/
ButtonText uses `body/s-medium` and `body/l-medium` tokens (lineHeight=1.4), while the regular Button uses `label/m` and `label/l` (lineHeight=1). The looser line height means the component has a more relaxed vertical rhythm and its height is derived from typography, not a fixed box.

### pressed/active Color Matches idle
The `fg-active` token for both regular and inverse is the same as `fg-idle`. This means pressed state does not change text color — only an overlay (if added in code) would distinguish it visually. This appears intentional: text buttons communicate press via interaction affordance, not a color shift.

### Token Derivation for Client Brands
When mapping a client brand to ButtonText:
- `components/button/text/fg-idle` → client brand secondary/muted tone (subdued text link color)
- `components/button/text/fg-hover` → brand darkest foreground (near-black for light mode)
- `components/button/text/fg-active` → same as idle (no change on press)
- Inverse tokens mirror: idle→muted-light, hover→near-white

## Known Gaps & Deviations

| Issue | Severity | Note |
|-------|----------|------|
| No focused state in Figma | Medium | Must add focus ring in code for a11y |
| size=s (~17px) below WCAG 44px touch target | Low | Same deviation as Button size=s; expected |
| size=m (~22px) below WCAG 44px touch target | Low | Text buttons are typically inline; note in consumer docs |

## Version History

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | 2026-04-09 | Initial spec extracted from Figma node 109:1057 |
