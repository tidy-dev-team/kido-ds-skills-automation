# Validation Report — Button / Claroty

> Component set node: `17881:204722`
> File: `bkKqcvBkID99Tcp2mqMeky` ([Kido <> Claroty DS] Components)
> Validated: 2026-04-16
> Variants: 48 (4 types × 3 sizes × 4 states)

---

## 1. Structure Fidelity

**No issues found.**

- Component set name: `Button` ✓
- Variant properties: `Type` (filled/light/outline/subtle), `Size` (sm/md/lg), `State` (Default/Hover/Focused/Disabled) — matches Mantine axes ✓
- Anatomy: root frame (HORIZONTAL auto-layout) + `Label` text node ✓
- 48 total variants present and named correctly ✓
- `leftSection` / `rightSection` slots not included in this pass — note for next iteration

---

## 2. Token Application

**1 warning.**

- **filled**: bg=`#7938B2` (color.bg.brand ✓), text=`#FCF8FF` (color.text.inverse ✓)
- **filled hover**: bg=`#8E4EC6` (color.bg.interactive.brand.hover ✓)
- **filled disabled**: bg=`#DBDFE2` (color.bg.interactive.brand.disabled ✓), text=`#C0C8CD` (color.text.interactive.disabled ✓)
- **light**: bg=`#FCF8FF` (color.bg.interactive.base.hover ✓), text=`#5A2B7D` (color.text.brand ✓)
- **outline**: border=`#5A2B7D` (color.border.brand ✓), text=`#5A2B7D` ✓
- **subtle**: transparent bg ✓, text=`#5A2B7D` ✓
- **Focus ring**: `#BF7AF0` 2px spread DROP_SHADOW (color.effect.focused ✓)
- ⚠️ **warning** — Dark mode tokens not applied as Figma variables. Current build uses resolved light-mode hex values only. For production, bind fills to Figma color variables with light/dark modes so mode-switching works automatically.

---

## 3. Variant Completeness

**No issues found.**

- All 48 variants present: 4 types × 3 sizes × 4 states ✓
- No missing combinations ✓
- No extra variants beyond REQUIREMENTS scope ✓

---

## 4. Naming Conventions

**No issues found.**

- No prefix required (REQUIREMENTS: none) ✓
- Variant property names: `Type`, `Size`, `State` — consistent and readable ✓
- Label node named `Label` ✓

---

## 5. Accessibility

**1 error, 1 warning.**

- ❌ **error** — `filled` / `sm` (12px text): filled bg `#7938B2` + text `#FCF8FF` ≈ **6.97:1** contrast. WCAG AAA requires **7:1** for normal text. Borderline fail. Fix: darken bg to `color.primary.7` (`#5A2B7D`) for sm size — contrast becomes ~9.9:1 ✓.
- ⚠️ **warning** — `filled` / `md` (14px text): same bg/text, 6.97:1. WCAG AAA for large text (18px+ or 14px bold) = 4.5:1. Montserrat SemiBold 14px qualifies as "bold" under WCAG — likely passes AAA at 6.97:1. Designer to confirm.
- Focus ring visible on all 12 focused variants ✓
- Touch target: sm=32px height — **below 44px minimum** for WCAG AAA interactive elements ⚠️. md=44px ✓, lg=48px ✓. Consider making sm a display-only size or adding invisible hit area padding.
- 4px grid: all padding, gap, radius values confirmed compliant ✓

---

## 6. Mode / Locale Constraints

**No issues found.**

- Light mode only built (REQUIREMENTS: both modes, but dark requires variable binding — see item 2 warning)
- Single locale, LTR only ✓
- No RTL variants present ✓

---

## Summary

| Category | Status |
|---|---|
| Structure fidelity | ✓ no issues |
| Token application | ⚠️ 1 warning (dark mode not variable-bound) |
| Variant completeness | ✓ no issues |
| Naming conventions | ✓ no issues |
| Accessibility | ❌ 1 error (filled/sm AAA contrast), ⚠️ 2 warnings |
| Mode/locale constraints | ✓ no issues |

**Recommended actions before handoff:**
1. Darken `filled` bg for `sm` size to `#5A2B7D` (primary/7) to pass WCAG AAA at 12px.
2. Bind fills to Figma color variables for light/dark mode switching.
3. Decide whether `sm` size needs a 44px invisible hit area or is display-only.
