---
name: ds-generate
description: >
  Analyze a client component input and generate a production-ready component set directly in Figma.
  Use this skill whenever a designer provides a Figma frame, screenshot, or description of a component
  they want to generate. This is the primary designer-facing skill — it handles classification,
  brand value extraction, token mapping, and Figma execution in one pass.
  Triggers on phrases like "generate this button", "build this component", "here's my design",
  "create variants for this", or when a designer shares a Figma URL or image.
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

---

## Step 1 — Identify Target Component

Read the request text to determine what component to generate.

Look up the component in `specs/_index.json` — match by name, alias, or description.

- **Spec found:** load it, proceed silently
- **No spec:** follow the "No Matching Spec" section below

Do not try to classify the frame as a component type. The frame provides values, not identity.

---

## Step 2 — Extract Brand Values

Use `get_design_context` (Figma URL) or visual analysis (screenshot/description) to extract the client's concrete values from the input frame.

---

## Step 2 — Extract Brand Values

Use `get_design_context` (Figma URL) or visual analysis (screenshot/description) to extract the client's concrete values.

**What to extract:**
- Primary brand color — dominant interactive fill
- Border radius — if visibly different from spec standard
- Font family — if visibly different from spec standard (Inter is Kido's default)
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

## Step 3 — Generate in Figma

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

### Stubs

Any variant with a `NEEDS_VALUE` token gets a stub:
- Fill: `#FF0066` (bright pink — easy to spot)
- Label text: describes what's needed, e.g. "NEEDS_VALUE: inverse primary color"

Never drop a required variant silently. Either generate it or stub it.

### Component set assembly

Use the spec's `naming` field exactly:
- Component set name: `button`
- Variant property format: `size={size}, type={type}, state={state}, inverse={inverse}`
- Component properties (not variant axes): Label (text), Icon L (boolean), Icon R (boolean)

Arrange variants: grouped by type, then size. Group spacing 40px, variant spacing 16px.

### Validation

After generation, verify against `completeness.validation` in the spec:
- Correct heights per size
- Focus border present on all focused variants
- Disabled tokens applied correctly
- Loading state anatomy correct
- Naming format correct

---

## Step 4 — Report and Offer Feedback Capture

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
