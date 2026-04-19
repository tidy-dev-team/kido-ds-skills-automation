# REQUIREMENTS.md — {Project or Job Name}

> Per-job rules for `/ds-build`. Either paste this file into `working/{job}/` and edit before running `/ds-build`, or let the skill interview you and write it automatically (Step 0).

---

## Scope

- **Project / client:** _{e.g., Acme Corp — Button library}_
- **Component(s) being built:** _{e.g., Button, Input, Checkbox}_
- **Figma build target:** _{Figma file URL + page name where the component set will be created}_
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

---

---

# Kido DS Quality Standards

> **These apply to every component built with `/ds-build` or `/ds-generate`.** They are not per-job settings — they are Kido's baseline acceptance criteria. The validator checks items marked ⚙️ automatically; items marked 👁️ require designer sign-off before handoff.

---

## ❗ Auto-Layout — STRICT

**Every frame that contains children must use auto-layout.** No exceptions.

- Apply at every level of the component tree: card → content area → row → icon+label.
- Absolute positioning is only permitted for decorative overlays, focus rings drawn as effects, or elements that genuinely float (tooltips, badges positioned relative to a parent).
- All spacing between children must be expressed as `gap` in the auto-layout panel, not as margins on individual nodes.
- Resizing mode must be explicitly set on every frame: `fixed`, `hug contents`, or `fill container` — never left undefined.

_Rationale: auto-layout is the only way components resize correctly in production. A component without it will break the moment a designer changes instance text or swaps an icon._

---

## 🏗️ Layer Naming & Structure

- All layers carry semantic names: `container`, `wrapper`, `header`, `body`, `footer`, `content`, `icon`, `label`, `caption`, `action`, `divider`, `badge`, etc. No "Frame 1", "Group 2", or "Rectangle 3".
- Component tree depth is intentional — not too deep (no unnecessary wrapper frames), not too flat (no "everything in one frame"). A good rule: each layer level should represent a real layout boundary.
- Layer names are lowercase and hyphenated for multi-word nodes: `icon-left`, `button-label`, `input-field`, `helper-text`.

---

## 🏷️ Component & Variant Naming

- **Component name** matches the developer's implementation name exactly (agreed with engineering before build).
- **Component lives above any frame/section** in the Figma layers panel — never nested inside a frame or section as a master. Only *instances* live inside frames.
- **Variant property vocabulary** is consistent across the system:

  | Axis | Use | Avoid |
  |------|-----|-------|
  | Size | `sm` / `md` / `lg` | `small`, `S`, `compact` |
  | Breakpoint | `desktop` / `mobile` | `large` / `small`, `web` |
  | Heading level | `h1`–`h6` | `title`, `headline` |
  | Body text | `body-lg` / `body-sm` | `paragraph`, `text` |
  | State | `default` / `hover` / `focused` / `disabled` / `error` | `normal`, `active`, `inactive` |
  | Appearance | match library convention (e.g. `filled` / `outline` / `subtle`) | custom synonyms |

---

## 🎨 Tokens & Styles (Variables)

All of the following must be bound to Figma variables or styles — no hardcoded values in published components:

- ⚙️ **Color fills & strokes** → color variables
- ⚙️ **Typography** (family, size, weight, line-height) → text styles or typography variables
- ⚙️ **Effects** (shadows, blurs) → effect styles or variables
- ⚙️ **Spacing & sizing** (padding, gap, corner radius, fixed widths/heights) → spacing/radius variables, or manually verified on 4px grid if variables don't exist yet

> If spacing variables exist in DESIGN.md, bind. If not, every spacing value must be a multiple of 4px.

---

## ⚙️ Component Properties

- **Text properties** — Every user-facing text node (`label`, `title`, `helper-text`, `placeholder`, etc.) has a component text property so the value can be edited from the instance panel without detaching.
- **Boolean properties** — Optional elements (icon slot, badge, loading state, helper text) are toggled via boolean component properties, not by manually showing/hiding layers.
- **Instance swap properties** — Every icon slot has an instance-swap property pointing to the correct icon library component. Default value is set.
- **Variant properties** — All visual states that differ in structure or color are modeled as variants, not as manual layer show/hide.

---

## 🔣 Icons, Illustrations & Logos

- **Flattened vector** — Every icon inside a component is a flattened SVG (no nested groups, no masks, no booleans). The vector layer is named `ic`.
- **Base color** — Icon paths use `#000000` as the fill. Color is applied via the parent frame's fill tint or a bound color variable — never colored directly on the path.
- **Foundation link** — Icons, illustrations, and logos are component instances sourced from the project's foundation file or dedicated icon library — never pasted as detached vectors. This ensures any upstream update propagates automatically.

---

## 🔤 Typography: Desktop / Mobile Correlation

- A text element that is `h5` in the desktop layout must remain `h5` in mobile — only the scale/size token changes, never the semantic hierarchy level.
- Desktop and mobile style names share the same tier prefix: `heading/h5/desktop` and `heading/h5/mobile`.
- If a component has a single breakpoint variant, the text style tier must still be documented so mobile can be built consistently later.

---

## 📐 Responsiveness & Min / Max

- Every component has a defined **min-width** and, where applicable, **max-width** (and min/max height).
- All frames use explicit resizing behavior — `fill container`, `hug contents`, or `fixed`. Never undefined.
- Components must visually survive being dragged narrower and wider in Figma before sign-off (👁️ designer drag-test).
- Whether a component is responsive or fixed-width must be stated explicitly in REQUIREMENTS (under Component-Specific Constraints).

---

## 🧩 Nesting & Composition

- Nested components (e.g., Button inside a Card) are **instances**, never detached copies.
- Properties of nested instances that designers will commonly want to change (e.g., button variant, icon choice) are **exposed** in the parent component's property panel via instance-swap or nested-override.
- **Not too heavy** — more than ~5 distinct nested component types is a signal to split the component into sub-components (atoms → molecules).
- **Not too flat** — a component with all styling baked into a single frame and no sub-components is a signal that atoms are missing from the system.
- **Preferred components** — Where only certain variants are valid inside a slot (e.g., only icon-only button variants go in a toolbar), set them as "preferred" in the Figma component panel so designers see the right options first.

---

## 📝 Description & Discoverability

Every published component must have a Figma description in this format:

```
Also known as: {alias1}, {alias2}, {alias3}
{component name typed on Hebrew keyboard layout — for Hebrew-locale search}
{one-line purpose description, optional}
```

**Example for "Button":**
```
Also known as: CTA, action button, primary action
נואאםמ
A clickable element that triggers an action or navigation.
```

- The "also known as" line covers common synonyms and informal names used by PMs, developers, and non-designers.
- The Hebrew-keyboard misprint (e.g., `נואאםמ` for "Button") ensures Hebrew-locale designers can find the component when they accidentally type with the wrong keyboard layout active.
- To generate the misprint: type the component name on an English keyboard while the Hebrew layout is active, or use a keyboard mapping table.

---

## ✅ Pre-Handoff Checklist

Before marking a component ready for handoff, confirm:

| # | Check | How |
|---|-------|-----|
| 1 | Auto-layout on every frame | ⚙️ validator |
| 2 | All layers named semantically | 👁️ layer panel review |
| 3 | Component name matches dev | 👁️ align with engineering |
| 4 | All colors/typography/effects/spacing bound to variables | ⚙️ validator |
| 5 | All text nodes have component text properties | ⚙️ validator |
| 6 | Icon layers named `ic`, flattened, base color `#000000` | 👁️ icon audit |
| 7 | Icons/illustrations linked to foundation (instances, not copies) | 👁️ layer panel review |
| 8 | Desktop/mobile typography hierarchy consistent | 👁️ side-by-side review |
| 9 | Min/max width and height defined | 👁️ drag-test in Figma |
| 10 | Nested components are instances with exposed properties | ⚙️ validator |
| 11 | Preferred variants set where applicable | 👁️ component panel check |
| 12 | Description written: aliases + Hebrew misprint | 👁️ Figma description field |
