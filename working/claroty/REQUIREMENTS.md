# REQUIREMENTS.md — Claroty DS

> Per-job rules for `/ds-build`. Written automatically by Step 0 interview on 2026-04-16.

---

## Scope

- **Project / client:** Claroty DS
- **Component(s) being built:** Button
- **Figma build target:** `https://www.figma.com/design/bkKqcvBkID99Tcp2mqMeky` — page: "dev tests", node: 17881:204722
- **Library:** Mantine ^7.0.0
- **Date started:** 2026-04-16

---

## Color Modes

- [x] both (light + dark)

> Generate variants for each mode using Figma variables bound to mode-aware tokens from DESIGN.md.

---

## Locales / RTL

- Primary locale: en
- Additional locales: none
- RTL required: no

---

## Naming Prefix

- Prefix for Figma node names: none
- Prefix for Figma variable names: none

---

## Accessibility Target

- [x] WCAG AAA
- Minimum touch target: 44×44 px
- Focus ring requirement: 2px spread, visible on all focused variants

> AAA requires stricter contrast: 7:1 for normal text, 4.5:1 for large text.

---

## Component-Specific Constraints

- Use 4px grid for all sizing and spacing values (padding, height, gap, radius must snap to multiples of 4px).

---

## Library Reference

- Library name / package: custom (Claroty DS)
- Figma Components file: `https://www.figma.com/design/bkKqcvBkID99Tcp2mqMeky/-Kido-%3C%3E-Claroty-DS--Components`
- Storybook URL: none provided

---

## DESIGN.md Reference

- Path: `working/claroty/DESIGN.md`
- Extracted from: `https://www.figma.com/design/92UFivxU2TyRTIe2U1Zjk0` + `https://www.figma.com/design/BHHyTAD7I1nszu4KxzNo4B`
- Extracted on: 2026-04-16

---

## Notes

- Multi-vertical DS: Industrial (purple primary, default) and Healthcare (indigo primary). Build Industrial variant set first.
- Secondary accent: Industrial = yellow (#FCDB6C), Healthcare = orange (#FA934E).
