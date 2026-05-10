# Design System Token Logic

> **Living document.** This file reflects the current state of the DS token system.
> Update it whenever the system is restructured, a new collection is added, or a rule changes.
> Skills read this file at the start of every run — stale rules produce wrong output.

---

## Purpose

This file teaches Claude the token logic of this design system so that every skill run —
`ds-generate`, `ds-build`, `ds-push` — applies tokens correctly without guessing.

**When a skill finds this file at the repo root, it must read it before touching Figma.**
The rules here override any generic defaults baked into the skill files.

For per-project specifics (variable IDs, brand names, custom modes), see the **Project-specific overrides** section at the bottom of this file or `working/{project}/DESIGN-SYSTEM.md` if a project-scoped override exists.

---

## Collections Overview

| Collection | Owns | Figma field |
|---|---|---|
| **Semantic colors** | All color tokens — fg, bg, border across static and interactive states | `fills`, `strokes` |
| **Corner radius** | All border-radius values — sharp through pill | `cornerRadius` |
| **Spacing** | All spacing/gap values | `paddingX`, `paddingY`, `itemSpacing` |
| **Typography** | Font family tokens | `fontFamily` |
| **_Global** | Primitive color palette (raw hex values — not for direct component use) | — |

**Rule: never hardcode a hex value or a raw number for radius/spacing. Always bind to the appropriate collection variable.**
A hardcoded value breaks brand-switching across modes.

---

## Semantic Colors — Core Logic

### Top-level categories

Every semantic color token starts with one of three prefixes:

| Prefix | Used for | Always applied to Figma field |
|---|---|---|
| `fg/` | Text nodes, icon fills, foreground elements (e.g. focus rings built as fills) | `fills` |
| `bg/` | Container backgrounds — pages, buttons, cards, any element that holds other elements | `fills` |
| `border/` | Strokes — outlined buttons, input borders, separators, dividers | `strokes` |

**Exception — focus rings:** focus rings can use either `fg/` (when built as a fill/effect) or `border/` (when built as a stroke). The token category follows the implementation choice.

---

### `static` vs `interactive`

Every `fg/`, `bg/`, and `border/` token belongs to one of two sub-categories:

**`static`**
- Used for the idle/resting state of any element — interactive or not
- A page background uses `bg/static/page`
- A primary button in its idle state uses `bg/static/brand`
- There is no `interactive/*/idle` — idle always lives in `static`

**`interactive`**
- Used only when the user is actively doing something: hover, pressed/selected, focused, disabled
- Never used for idle states
- Decision rule: **start with `static`, escalate to `interactive` only when state changes**

---

### `interactive` sub-categories: `brand`, `muted`, `base`, `secondary`

| Sub-category | Represents | Example use |
|---|---|---|
| `brand` | Primary brand color at full strength — highest hierarchy interactive element | Primary button hover/pressed, form focus ring |
| `muted` | Same brand color at reduced intensity — brand-adjacent but softer | Subtle selected states, lower-emphasis brand interactions |
| `base` | Neutral grey — no brand association, mode-independent | Table row hover, sidebar item selected, system UI chrome |
| `secondary` | Secondary brand color (distinct from primary) — exists for `bg` only | Secondary button hover/selected background |

**Key distinction — `muted` is not a separate color.**
`muted` is the primary brand color at a lighter tint, not a different color.
`secondary` is a genuinely different brand color (a second, distinct brand hue when the project defines one).

**`secondary` under `interactive` exists for `bg` only** — there is no `fg/interactive/secondary` because text on secondary backgrounds uses `fg/static/on-brand` (white), which doesn't change by state.

---

### Disabled states

- Disabled tokens always resolve to **neutral grey** — never brand, never secondary
- The rule for which disabled token to use: **stay in the same family as the component's other states**

```
Primary button example:
  idle     → bg/static/brand
  hover    → bg/interactive/brand/hover
  pressed  → bg/interactive/brand/selected
  disabled → bg/interactive/brand/disabled   ← same family (brand), resolves to grey
```

```
Neutral/base element example:
  idle     → bg/static/base
  hover    → bg/interactive/base/hover
  disabled → bg/interactive/base/disabled    ← same family (base), resolves to grey
```

The family determines which disabled token to pick — not the resulting colour.

---

### How to read a token name

```
bg / interactive / brand / hover
│       │           │        └── state: hover | selected | disabled | focus
│       │           └── sub-category: brand | muted | base | secondary
│       └── group: static | interactive
└── layer: fg | bg | border
```

---

## Modes = Brands

The Semantic colors collection has **multiple modes — each mode is a separate brand**. The default project ships with one mode; additional brands are added as new modes on the same collection (not new collections).

**Every component node must be bound to a variable.**
A hardcoded hex value won't respond to a mode switch. With multiple brands planned, every unbound value requires a manual fix per brand — binding once handles all of them.

When generating components:
1. Bind all color slots to Semantic colors variables
2. Bind all radius slots to Corner radius variables
3. Bind all spacing slots to Spacing variables
4. Hardcoded values are only acceptable for values that genuinely have no variable equivalent (e.g. overlay opacities — `#0000001f` for hover, `#00000033` for pressed)

The list of active modes for the current project lives in **Project-specific overrides** at the bottom of this file.

---

## Component Token Assignment — Decision Tree

When assigning a token to a component layer, answer these questions in order:

```
1. What kind of layer is it?
   → Text or icon fill            → use fg/
   → Container background         → use bg/
   → Stroke/outline               → use border/

2. What state is this variant?
   → Idle                         → use static/
   → Hover / pressed / focused / disabled → use interactive/

3. What's the component's hierarchy?
   → Primary CTA / highest hierarchy      → brand
   → Same brand color, lower emphasis     → muted
   → Neutral / no brand association       → base
   → Secondary brand color                → secondary (bg only)

4. Is it disabled?
   → Yes → pick the disabled token from the same family as the non-disabled states
            (it will always resolve to grey — that's correct)
```

---

## Bootstrap Alignment

This DS follows Bootstrap's conventions where applicable. When in doubt, Bootstrap's logic is the reference.

**State name mapping — three names for the same thing:**

| This DS (token name) | Bootstrap term | UX term |
|---|---|---|
| `selected` | `active` | pressed |
| `hover` | `hover` | hover |
| `disabled` | `disabled` | disabled |
| `focus` | `focus` | focused |

Always use `selected` in token names (not `pressed`, not `active`). Map to Bootstrap's `active` when reading Bootstrap docs.

**Bootstrap equivalents for token sub-categories:**

| This DS | Bootstrap equivalent |
|---|---|
| `brand` | `primary` / emphasis variant |
| `muted` | `subtle` variant |
| `base` | neutral / default |
| `secondary` | `secondary` |

---

## State Ownership Rule

The strict rule — *hover/pressed → always use `interactive/`* — has a justified exception for multi-layer components. This is consistent with Bootstrap's own component behavior.

**The principle:** within a component, one layer owns the state signal. That layer uses `interactive/` tokens. Other layers that don't need to independently communicate state use `static/` tokens and remain visually stable.

Bootstrap example: a primary button on hover — the background changes (`btn-primary` darkens) but the label text stays white. The text does not get its own hover color. The background owns the hover state.

**Which layer owns state:**

| Component has… | Owns state | Other layers |
|---|---|---|
| A `bg-zone` or hover background | `bg/` layer | Use `static/` |
| A focus ring | `border/` layer | Use `static/` |
| No bg, no zone, no ring | Icon or label layer | Use `interactive/` |

**When to break the strict rule (use `static/` on non-idle states):**
- Another layer in the same component is already using an `interactive/` token for that state
- Adding a color change to the icon/label would compete with or contradict that signal

**When NOT to break it:**
- The element is the only visual feedback mechanism for that state (e.g. a text link — no background, no zone, color is the only signal)
- The design calls for deliberate double-reinforcement — e.g. a pressed state where both the background AND the icon darken together for a stronger action signal (Bootstrap calls this the `active` state — it IS intentional reinforcement)

**Radio button — canonical reference:**
- Hover: `bg-zone` uses `bg/interactive/muted/hover` → owns hover. Ring/dot stay `fg/static/brand` — static is correct.
- Pressed: `bg-zone` uses `bg/interactive/muted/selected` AND ring/dot use `fg/interactive/brand/selected` → intentional double reinforcement for a decisive action.
- Focus: focus ring uses `border/interactive/brand/focus` → owns focused. Ring/dot stay `fg/static/brand`.

---

## Interactive State Token Mapping (Hover / Pressed)

Never hardcode hover or pressed colors. Determine which layer owns the state signal, then pick the right token.

### Step 1 — Identify the primary signal layer

| Element type | Primary signal layer |
|---|---|
| Unchecked checkbox, radio ring (unselected), outlined/ghost button border | **Stroke** |
| Checked/indeterminate checkbox, contained button background | **Fill** |
| Ghost/outlined button hover background zone | **Dedicated bg layer** |

### Step 2 — Apply the correct token

**Stroke-primary elements** (e.g. unchecked checkbox):
- hover → change stroke to `border/interactive/brand/hover` — **no fill added**
- pressed → same as hover — **no fill added**
- The box/ring has no background; adding a fill would create an opaque colored box

**Fill-primary elements** (e.g. checked checkbox, contained button):
- hover → change fill directly to `bg/interactive/brand/hover`
- pressed → change fill directly to `bg/interactive/brand/selected`
- No overlay layer needed — the fill owns the state

**Ghost/outlined button bg zone** (a dedicated hover background, not the label or border):
- hover → apply `bg/interactive/muted/hover` as fill
- pressed → apply `bg/interactive/muted/selected` as fill

### Why `bg/interactive/muted` ≠ a transparent overlay

`bg/interactive/muted/hover` and `bg/interactive/muted/selected` resolve to **fully opaque brand tints** (e.g. a light brand-200 / brand-300 step). They are NOT semi-transparent. Never apply them as a fill layer on an element that has no background in its idle state — the result will be an opaque colored box.

Use `bg/interactive/muted` only on elements that carry a dedicated background layer (button hover zones, chip hover zones).

---

## Project-specific overrides

> Per-project specifics (variable IDs, brand names, custom modes) live below.
> Generic rules above this line apply to every project. Overrides below apply only to the current project — replace this section when starting a new client.

### Modes (brands) active in this project

| Mode | Brand | Primary color |
|---|---|---|
| _(none yet)_ | — | — |

Add one row per active mode in the Semantic colors collection.

### Variable IDs

| Token | Variable ID (or alias) |
|---|---|
| `border/interactive/brand/hover` | _(fill in once collection is wired)_ |
| `bg/interactive/brand/hover` | _(fill in once collection is wired)_ |
| `bg/interactive/brand/selected` | _(fill in once collection is wired)_ |
| `bg/interactive/muted/hover` | _(fill in once collection is wired)_ |
| `bg/interactive/muted/selected` | _(fill in once collection is wired)_ |

Only fall back to hardcoded opacities if these variables genuinely do not exist in the project's Semantic colors collection.

---

## Known Gaps & WIP Notes

- The token system is not yet complete or fully optimized
- Not all components have full token coverage
- This file must be updated whenever the token system is restructured or a new brand/mode is added
