# REQUIREMENTS.md — {Project or Job Name}

> Per-job rules for `/ds-build`. Either paste this file into `working/{job}/` and edit before running `/ds-build`, or let the skill interview you and write it automatically (Step 0).

---

## Scope

- **Project / client:** _{e.g., Acme Corp — Button library}_
- **Component(s) being built:** _{e.g., Button, Input, Checkbox}_
- **Date started:** _{YYYY-MM-DD}_

---

## Color Modes

Which modes must the components support?

- [ ] light
- [ ] dark
- [ ] both
- [ ] other: _{name}_

> If "both", generate variants for each mode using Figma variables bound to mode-aware tokens from DESIGN.md.
> If a single mode, skip the other to keep the variant count down.

---

## Locales / RTL

- Primary locale: _{e.g., en}_
- Additional locales: _{e.g., he, ar}_
- RTL required: _{yes / no}_

> For each RTL locale, `/ds-build` will generate mirrored variants where the library's behavior requires them (iconography, label alignment, directional affordances).

---

## Naming Prefix

- Prefix for Figma node names: _{e.g., `pz_` — or "none"}_
- Prefix for Figma variable names (if different): _{optional}_

> Applied to every node name and every bound variable the skill creates.

---

## Accessibility Target

- [ ] WCAG AA (default)
- [ ] WCAG AAA
- Minimum touch target: _{default: 44×44 px}_
- Focus ring requirement: _{default: 2px, visible on all focused variants}_

> The validator checks contrast ratios and touch targets against the selected target. AA is the default — AAA requires stricter contrast (7:1 for normal text, 4.5:1 for large text).

---

## Component-Specific Constraints

Free-text list of anything else the designer or PM knows about this job. Examples:

- "Loading state shows a progress bar, not a spinner."
- "Only `primary` and `secondary` variants — skip `ghost` and `link`."
- "All icons must come from the Lucide set, not the library default."
- "Text labels must never wrap; truncate with ellipsis."
- "Disabled state uses 40% opacity, not a separate color token."

_{List your constraints here, one per line.}_

---

## Library Reference

- Library name / package: _{e.g., @chakra-ui/react, @mantine/core, custom}_
- GitHub repo: _{URL, optional}_
- Storybook URL: _{URL, optional}_
- Version constraint: _{e.g., ^2.8.0, optional}_

> At least one of repo or Storybook is required for `/ds-build` to resolve the component structure. Version is helpful when the library has breaking changes across majors.

---

## DESIGN.md Reference

- Path: _{working/{project}-{date}/DESIGN.md}_
- Extracted from: _{Figma URL}_
- Extracted on: _{YYYY-MM-DD}_

> If DESIGN.md doesn't exist yet, run `/ds-extract-design` first.

---

## Notes

_{Anything else. Context for the designer or for Claude. Decisions made with the client. Known gaps.}_
