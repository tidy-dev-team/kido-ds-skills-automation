---
name: ds-generate
description: >
  Analyze a client component input and generate a production-ready component set directly in Figma.
  Use this skill whenever a designer provides a Figma frame, screenshot, description, live site URL,
  or code repository for a component they want to generate. This is the primary designer-facing skill
  — it handles classification, brand value extraction, token mapping, and Figma execution in one pass.
  Triggers on phrases like "generate this button", "build this component", "here's my design",
  "create variants for this", "here's the repo", "here's the site", "use this codebase",
  or when a designer shares a Figma URL, image, live site link, or GitHub repo.
---

# DS Generate

**One command. Frame in, component set out.**

Identify target component → extract brand values from frame → map tokens → execute in Figma.
No intermediate files. No approval gates. Stubs for anything unresolved.

---

## What the Input Frame Is

The frame the designer provides is **brand context** — a source of visual values (colors, radius, font).
It does not need to be the component being generated. A badge, a card, a header — any frame carrying the client's visual style is valid input.

**The target component comes from the request text**, not from the frame.
- "generate this button: [frame]" → generate a Button using brand values extracted from the frame
- "generate this toggle: [frame]" → generate a Toggle using brand values extracted from the frame

The designer rarely provides a fully interactive multi-state component. A single idle state, or any frame showing the brand primary color, is enough to generate the full set.

A live site URL or code repository works the same way: it supplies the brand values (colors, radius, font). The component to generate still comes from the request text.

---

## Step 0 — Read Project-Level Rules (always)

Before Step 1, check the repo root for **`DESIGN-SYSTEM.md`**. If present, read it.

Its contents are **authoritative** — they override any default behavior in this skill (token decision tree, state ownership, modes-as-brands, bootstrap state names, etc.). When this skill's defaults conflict with `DESIGN-SYSTEM.md`, the file wins.

If absent, proceed with this skill's defaults. Never invent rules to fill the gap — surface a `NEEDS_VALUE` stub or a `NEEDS_SPEC` plan item instead.

This step is silent — no output unless the file is present and changed your behavior in a way the designer should know about.

---

## Step 1 — Identify Target Component

Read the request text to determine what component to generate.

Look up the component in `specs/_index.json` — match by name, alias, or description.

- **Spec found:** load it, proceed silently
- **No spec:** follow the "No Matching Spec" section below

Do not try to classify the frame as a component type. The frame provides values, not identity.

---

## Step 2 — Extract Brand Values

Use the appropriate method based on what the designer provided:

- **Figma URL** → `get_design_context`
- **Screenshot / description** → visual analysis
- **Live site URL** → `WebFetch` the URL; extract CSS custom properties and computed styles from the returned markup/CSS
- **GitHub repo** → `gh api repos/{owner}/{repo}/contents/{file} | base64 -d`
  Priority files:
  1. `src/index.css` or `globals.css` — CSS custom properties (`--primary`, `--radius`, etc.)
  2. `tailwind.config.ts/js` — `theme.extend.colors`, `borderRadius`, custom keyframes
  3. The component source file — Tailwind class names (`rounded-*`, `font-*`, `tracking-*`, `uppercase`)

  **CSS/Tailwind → token slot mapping:**
  | Source value | Token slot |
  |---|---|
  | `--primary` / `theme.colors.primary` | `system/bg/primary` |
  | `--radius` / `rounded-{size}` | `border_radius` |
  | `font-{family}` class | `font_family` |
  | `font-black` / `font-bold` | typography weight |
  | `tracking-{size}` | `letterSpacing` |
  | `uppercase` | `textCase: UPPER` |

  **HSL → hex:** CSS often uses `hsl(H, S%, L%)` — convert to hex before applying to tokens.

  **Font resolution (always run this — do not assume Inter):**
  1. Extract the font family from the source (`font-family` CSS, `fontFamily` in Tailwind, `font-{family}` class on the component)
  2. Call `listAvailableFontsAsync()` in Figma and check if the exact family is available
  3. If exact match exists → use it
  4. If not → find closest by category:
     - Monospace (`font-mono`, `Courier`, `Menlo`, `Consolas`) → prefer **Geist Mono**, then JetBrains Mono, IBM Plex Mono
     - Geometric sans → prefer **Plus Jakarta Sans**, then DM Sans, Nunito
     - Neutral sans (system-ui, -apple-system) → prefer **Inter**
     - Serif → prefer **Playfair Display**, then Lora
  5. Match the weight: `font-black`=Black, `font-bold`=Bold, `font-semibold`=SemiBold, `font-medium`=Medium, `font-normal`=Regular
  6. If no weight variant exists for the chosen family, use the heaviest available
  7. Load the resolved font with `loadFontAsync` before applying
  8. Note the substitution in the session summary

**What to extract:**
- Primary brand color — dominant interactive fill
- Border radius — if visibly different from spec standard
- Font family — always extract; never default to Inter unless the client explicitly uses Inter
- Font weight, text transform, letter spacing — if specified in the source
- Any additional color values explicitly present in the input

**Map extracted values to spec token slots:**

The spec's `tokens` object contains Kido's default values. Override with client values:

```
client primary = #E53E3E
→ replace system/bg/primary: "#E53E3E"
→ derive components/button/outlined/bg-hover: "#E53E3E33"  (primary at 20% opacity)
→ derive components/button/outlined/bg-pressed: "#E53E3E61"  (primary at 38% opacity)
→ derive system/border/interactive/focused: "#E53E3E80"  (primary at 50% opacity)
→ derive system/border/interactive/hover: "#E53E3E"  (same as primary)
```

All other token values come from the spec unchanged — they are Kido system values, not brand-specific.

**Deviations:** if the client's radius or font differs from spec standard, use the client value silently. Note it in the session summary.

**Missing values:** stub as `NEEDS_VALUE`. Do not ask. Generation proceeds.

---

## Step 3 — Pre-flight Checklist & Execution Plan

**Hard gate.** Before any Figma code runs, emit a written execution plan derived from the loaded spec. Generation must not start until this plan has been output as a markdown checklist with every item unchecked. Step 5 (Self-Audit) ticks against it.

### Derivation rules

The plan is derived from the spec — not memorized. For any component:

| Section | Source in spec |
|---|---|
| **Variant axes** | `variants` — list each axis × its values. If `inverse.generate_by_default` is `false` and the designer didn't request inverse, exclude `inverse=yes` and use `completeness.default_set`; otherwise use `completeness.full_set`. |
| **Token bindings** | `tokens` — every color slot must be bound to a semantic variable (not a raw hex); every spacing/radius/typography slot must be bound to a variable. One checklist line per token group is enough. |
| **Component properties** | `anatomy.component_properties` (when present — see schema upgrade in #8). When absent, infer heuristically: layers named `Label`, `Title`, `Description` → **TEXT**; layers matching `Icon*` / `*Icon` → a **BOOLEAN** visibility property + an **INSTANCE_SWAP** swap property, paired. Any layer flagged in `anatomy.notes` as "component property, not variant axis" must appear here. If a layer is ambiguous, emit `[ ] (NEEDS_SPEC: clarify property type for {layer})` and continue. |
| **State anatomy** | one line per entry in `completeness.validation` that describes a state's visual contract (focus border, disabled tokens, loading swap, etc.). |
| **Sizing modes** | derived from `anatomy.layout`. If `layout` says "fixed height" → counter axis FIXED. If it says "hug" or implies content-driven width → primary axis AUTO. Emit one checklist line: e.g. `[ ] All variants: primaryAxisSizingMode = AUTO, counterAxisSizingMode = FIXED`. |
| **Icon slots** | derived from `anatomy.layers` and component_properties — any layer matching `Icon*` / `*Icon` / `Spinner` is an icon slot. If any exist, search the target Figma file for an icon library (component set named `icons`, `icon`, `icon-library`, or any set whose components are 16/20/24px square instances). Emit two lines: `[ ] Icon library: <found at {path}> | <not found — using placeholders>` and one `[ ] {Slot} populated` line per icon slot. See "Icon Slots" in Step 4. |
| **Identity** | component set name = `naming.component_set`; variant property format = `naming.variant_format`; component description written (one or two sentences derived from the spec or from `_index.json` description). |

### Output shape (illustration only — your component's plan is derived from its spec)

```
## Execution plan — <Component> (<N> variants)

### Variant axes
- [ ] <axis>: <v1>, <v2>, ...
  ...

### Token bindings
- [ ] All color slots bound to semantic variables
- [ ] All spacing/radius/typography slots bound to variables

### Component properties
- [ ] <Layer> → <TYPE> property
  ...

### State anatomy
- [ ] <state>: <validation contract from spec>
  ...

### Identity
- [ ] Component set name = "<naming.component_set>"
- [ ] Variant format = "<naming.variant_format>"
- [ ] Component description written
```

Items remain unchecked through Step 4. They get ticked off in Step 5 after reading the produced component set back from Figma.

If the spec is missing data needed to derive an item, surface the gap as a `[ ] (NEEDS_SPEC: ...)` line and proceed — never silently drop the item.

---

## Step 4 — Generate in Figma

Use Figma MCP tools to build the component set directly.

### Generation scope

**Default (always):** `inverse=no` variants only.
For Button: 3 types × 3 sizes × 6 states = **54 variants**.

**Full (only if designer explicitly requests dark-background support):**
Add `inverse=yes` variants. For Button: 108 variants total.

### Execution order

Build in dependency order — base before derived:

1. **Base variants** (`type × size`, `state=idle`, `inverse=no`) — these define the visual foundation
2. **State variants** — derive hover, pressed, focused, disabled, loading from each base
3. **Size variants** — if only one size was provided, scale others using spec sizing tokens
4. **Inverse variants** — only if requested; stub if inverse brand colors are unknown

### Token application per state

Token names are self-documenting — use them to determine what applies where.
Key patterns to know:

- `system/bg/primary` → contained background (idle)
- `system/overlays/overlay-hover-on-color` → overlay on contained hover
- `system/overlays/overlay-active-on-color` → overlay on contained pressed
- `components/button/outlined/bg-hover` → outlined background on hover
- `components/button/outlined/bg-pressed` → outlined background on pressed
- `system/border/interactive/focused` → focus border color (2px, all types)
- `system/bg/disabled` + `system/fg/disabled` → disabled state (all types)
- Loading state: hide Label and icons, show Spinner (20px), apply `paddingY_loading` from spec sizing

### Auto-layout sizing — pitfalls

Two operations silently override sizing modes. After either, **explicitly restore the intended modes** before moving to the next variant.

| Operation | Side effect | Required follow-up |
|---|---|---|
| Assigning `layoutMode` | Both sizing modes reset to `'AUTO'` (hug) | If a fixed dimension is required, set `primaryAxisSizingMode` / `counterAxisSizingMode = 'FIXED'` after assignment. |
| Calling `resize(w, h)` | Both sizing modes reset to `'FIXED'` | If the variant is meant to hug content on an axis, restore that axis to `'AUTO'` after `resize()`. |

For Button (and any component with `layout: "horizontal, ..., fixed height"` in its spec):
- counter axis (height) = `'FIXED'` — height comes from sizing tokens, never hugs
- primary axis (width) = `'AUTO'` — width hugs the label + icons

Correct pattern:

```js
comp.layoutMode = 'HORIZONTAL';            // resets both → AUTO
comp.resize(sc.mw, sc.h);                  // resets both → FIXED
comp.counterAxisSizingMode = 'FIXED';      // height stays FIXED
comp.primaryAxisSizingMode = 'AUTO';       // width restored to HUG
```

The order matters: do `resize()` first, then restore the modes. Never leave either mode unset after one of these operations.

### Stubs

Any variant with a `NEEDS_VALUE` token gets a stub:
- Fill: `#FF0066` (bright pink — easy to spot)
- Label text: describes what's needed, e.g. "NEEDS_VALUE: inverse primary color"

Never drop a required variant silently. Either generate it or stub it.

### Icon Slots

When the spec's anatomy declares icon slots (e.g. `Icon L`, `Icon R`, `Icon`, `Spinner`), follow this protocol — **never draw icons inline with vector primitives**.

**1. Search for an icon library.** Look in the target Figma file for:
- A component set named `icons`, `icon`, `icon-library`, `Icons`, or `Material Symbols Outlined`
- Any published library whose components are square instances at standard sizes (16 / 20 / 24 / 32 px)
- Components matching the icon names referenced in the spec (e.g. for CheckboxIcon: `check_box`, `check_box_outline_blank`, `indeterminate_check_box`)

If multiple candidates exist, prefer the one whose name or path most closely matches the spec's referenced icons.

**2a. Library found.** Use real icon instances — wire them to the slot's INSTANCE_SWAP property. Set the `default_component` to the most representative icon for the variant (e.g. arrow_forward for `Icon R`, the spec's specific icon for CheckboxIcon's checked state).

**2b. Library absent or doesn't cover the needed icons.** Insert a **placeholder frame** for every icon slot:
- Frame size: matches the spec's icon size (default 24 × 24, or per spec)
- Fill: `#FF0066` (bright pink — same convention as `NEEDS_VALUE` stubs)
- Centered text label: the icon's intended name in lowercase, e.g. `icon`, `arrow_forward`, `check_box`
- Make the placeholder a component instance so it can be swapped to a real icon later via INSTANCE_SWAP

The placeholder is the icon-slot equivalent of `NEEDS_VALUE` — visible, swappable, and surfaced in the report. Never substitute a hand-drawn shape, an emoji, or a reused glyph from another component.

**3. Surface the gap.** If any placeholder was used, add a line to the Step 6 report:

```
Icons not found — N placeholder(s) inserted: <Icon L>, <Icon R>, ...
Wire real icons via INSTANCE_SWAP during polish, or run `/ds-icon-bootstrap` to import a default icon set.
```

(`ds-icon-bootstrap` is not yet a skill — the message points the designer at the future bootstrap path; for now they import icons manually.)

### Component set assembly

Use the spec's `naming` field exactly:
- Component set name: `button`
- Variant property format: `size={size}, type={type}, state={state}, inverse={inverse}`
- Component properties (not variant axes): Label (text), Icon L (boolean), Icon R (boolean)

Arrange variants: grouped by type, then size. Group spacing 40px, variant spacing 16px.

---

## Step 5 — Self-Audit

After generation, read the produced component set back from Figma and tick the Step 3 checklist item by item. Never skip this step. Never silently omit a failure.

### Read-back tools

- `figma_get_component_details` — variant set, component properties (name + type + default), variant property format
- `figma_get_design_context` — token bindings on the produced component
- `get_screenshot` — visual confirmation of state anatomy

### Audit procedure

For each item in the execution plan:
- **Pass** → mark `[x]`
- **Fail** → mark `[ ]` and add a one-line note
- **Partial** → mark `[~]` with notes

If any item fails, attempt **one** corrective pass (re-run the missing piece — e.g. add the missing component property, rebind the missing token, fix the variant format). After the corrective pass, re-audit. If items still fail, surface them explicitly in Step 6 — do not silently omit.

### What to verify per section

- **Variant axes** — full cartesian product produced; no missing combinations.
- **Token bindings** — every visible fill / stroke / spacing on the produced component reads back as a variable reference, not a raw value (except declared `NEEDS_VALUE` stubs).
- **Component properties** — each property in the plan exists on the component set with the correct *type* (TEXT / BOOLEAN / INSTANCE_SWAP) and bound to the correct layer.
- **State anatomy** — each item in `completeness.validation` holds (e.g. heights per size, focus border present, disabled tokens applied, loading swap correct).
- **Sizing modes** — for every variant, the auto-layout sizing modes match the spec's `layout` intent. For a `"horizontal, ..., fixed height"` component (e.g. Button): `primaryAxisSizingMode === 'AUTO'` and `counterAxisSizingMode === 'FIXED'` on every variant. A variant that ends up width-fixed at the minimum is the canonical resize() regression — fail and restore. (See "Auto-layout sizing — pitfalls" in Step 4.)
- **Icon slots** — every layer declared as an icon slot is populated with either (a) a real icon instance from the resolved library or (b) a `#FF0066` placeholder frame whose label matches the intended icon name. No icon slot is empty; no slot contains a hand-drawn shape. Count placeholders for the Step 6 report.
- **Identity** — component set name and variant property format match `naming` exactly; component description is set on the component set.

Output the audited checklist before the Step 6 report. The designer sees both the original plan and the resulting state.

---

## Step 6 — Report and Offer Feedback Capture

Give the designer a brief summary:

```
Button generated.

54 variants created (inverse=no set).
  • 1 built from your design
  • 53 from Kido rules

Radius: 12px (your design) vs 8px (Kido standard) — using yours.

No stubs. Ready to review in Figma.
```

Or if there are stubs:

```
Button generated.

54 variants created. 6 stubs (bright pink) — secondary color not in input.
Find them in Figma and fill in the brand secondary color.
```

If icon placeholders were used, surface them as a separate line:

```
Icons not found in this file — 2 placeholder(s) inserted: Icon L, Icon R.
Wire real icons via INSTANCE_SWAP during polish, or import an icon set first
(e.g. Material Symbols Outlined) and re-run the icon-swap step.
```

The icon line appears in addition to the regular stub count, never merged with it — they're different gaps with different fixes.

**Offer feedback capture** if any stubs were resolved during the session:

"If you fill in any stubs, I can record the values for the next project — just share them when you're done."

If the designer provides resolved values, save to `working/{component}-{date}/resolved-stubs.json`.

---

## No Matching Spec

Try in order:

1. **Closest spec** — rank by shared variant axes and similar anatomy. Proceed with it, note differences.
   Tell the designer: "No exact spec for [X]. Using [Y] as the closest match."

2. **Compose from two specs** — if the component clearly combines two patterns (e.g. Tag = Button structure + Badge sizing).

3. **Ask** — only if neither works: "I don't have a spec for this. Should I use [Y] as a base, or author a new spec first with `/ds-spec-authoring`?"

---

## Working Directory

Save session artifacts to `working/{component}-{YYYY-MM-DD}/`:
- `token-map.json` — the resolved token values used for this generation
- `resolved-stubs.json` — any stub values provided by the designer (optional)

These are local only, not committed to the repo.
