---
name: ds-doc
description: >
  Generate component documentation as markdown in the chat — variant descriptions
  and a Usage Guidelines section — from a polished Figma component-set URL. Reads
  component data via the official Figma MCP only; never edits the canvas. Synthesises
  Usage Guidelines from `specs/{component}.guidelines.json` (recommended=true entries)
  when available, otherwise searches the web for industry best practices. Returns
  formatted markdown in the chat window — no Figma writes, no files written.
  Triggers on phrases like "document this component", "write docs for X",
  "generate variant descriptions", or when a designer has finished polish and asks
  for component documentation prose.
---

# DS Doc

**Polished Figma component → markdown documentation in the chat.**

Two deliverables:

1. **Variant descriptions** — one short paragraph per variant explaining purpose + "When to use" guidance.
2. **Usage Guidelines** — When-to-use / When-not-to-use / General / Behavior / Content / Accessibility prose.

The skill is read-only on Figma and writes nothing to disk. All output goes to the chat as formatted markdown.

---

## Intake

The skill takes a single argument: the polished Figma component-set URL.

Extract `fileKey` and `nodeId` from the URL (`figma.com/design/:fileKey/...?node-id=:nodeId`; convert `-` to `:` in nodeId).

Infer the **component name** from the Figma node's name (e.g., `Button`, `Banner Contained`). If the inferred name doesn't match any entry in `specs/_index.json`, ask the designer once to confirm the spec slug — don't guess across plausible matches.

If no URL is provided, ask for it. Don't pick a default.

---

## Constraints (hard rules)

- **Never build anything on canvas.** No `use_figma` calls that mutate. Read-only Figma access.
- **Use MCP only to get component data.** `get_design_context`, `get_metadata`, `get_screenshot`, `get_variable_defs` are the allowed Figma tools.
- **Search the web without asking** when guidelines.json is absent or thin. Use `WebSearch` / `WebFetch` directly — no clarifying question.
- **Return results as formatted markdown in the chat.** No files written, no working/ artifacts.

---

## Step 1 — Pull component data from Figma

Call the official `claude.ai Figma` MCP:

- `get_design_context` on the URL → variant axes, variant values, layer structure, resolved tokens.
- `get_metadata` if needed to enumerate variant nodes.
- `get_screenshot` only if a visual reference is useful for distinguishing variants (e.g., to verify variant naming maps to visual emphasis).

From the returned data, build:

- The **list of variant axes** (e.g., `variant`, `size`, `state`).
- For each axis, the **list of variant values** (e.g., `variant ∈ {primary, secondary, ghost, danger}`).
- The component's **anatomy summary** (layer names, icon slots, content slots) — used to ground the prose.

The "variants" the user wants described are the values of the primary semantic axis — usually `variant`, sometimes `kind`, `emphasis`, or `type`. Pick the axis that carries emphasis/role distinctions (not `size`, `state`, or boolean toggles). If ambiguous, describe both candidate axes and let the designer pick — but make a recommendation first.

---

## Step 2 — Load Kido spec context (best-effort)

Try to load:

- `specs/_index.json` — to resolve the component slug.
- `specs/{component}.spec.json` — for anatomy, sizing, and any structured variant metadata.
- `specs/{component}.spec.notes.md` — for design decisions and Kido-voice rationale.
- `specs/{component}.guidelines.json` — primary source for Usage Guidelines prose.

Missing files are fine — the skill degrades to "polished Figma + web research" without them. Don't fail.

---

## Step 3 — Write variant descriptions

For each value of the primary variant axis, write **one short description** (~3-6 sentences) covering:

1. **What the variant is** — one sentence on its visual emphasis and role in the hierarchy.
2. **When to use it** — 2-4 bullet points describing the right context.
3. *(Optional — only when the constraint is sharp and well-known)* **When not to use it** — one line.

Use industry-standard hierarchy logic (Primary = most important action, single-per-screen; Secondary = important non-dominant; Tertiary / Ghost = low-emphasis, supporting actions; Danger / Destructive = irreversible / negative consequences; Link = inline navigation, not commitment). Adapt to whatever variants the component actually exposes.

For non-button components, derive equivalent emphasis logic from the variant's name and visual treatment (e.g., Banner: Info / Success / Warning / Error — each tied to a message severity; Badge: Solid / Outlined / Subtle — each tied to a prominence level).

**Voice:** confident, prescriptive, in Kido tone — sentences not fragments, active voice. Match the example style from the user's request (Primary / Secondary buttons).

**Grounding:** if `specs/{component}.spec.notes.md` has design decisions relevant to a variant (e.g., "Ghost idle text uses primary fg color — intentional, ghost buttons are interactive not neutral"), weave that into the description as the rationale. Don't paste the design-decision verbatim — synthesise.

---

## Step 4 — Write Usage Guidelines

Produce six subsections (omit any that come up empty after research):

1. **When to use** — bulleted Do list
2. **When not to use** — bulleted Don't list
3. **General guidelines** — bulleted Do list (broader principles)
4. **Behavior** — Do/Don't paired rules covering interaction & states
5. **Content** — Do/Don't paired rules covering labels, copy, length
6. **Accessibility** — bulleted Do list (a11y rules)

### Source order

For each subsection:

1. **First**, check `specs/{component}.guidelines.json`. Filter to `recommended: true` entries. Map by `type` → section:
   - `Do` → When-to-use / General / Behavior-Do / Content-Do (decide by content)
   - `Not` → When-not-to-use / Behavior-Don't / Content-Don't
   - `Accessibility` → Accessibility
   - `Best` → General
   - `Description` → skip (not rendered)

   For Behavior and Content Do/Don't pairs, group `Do` + `Not` entries that address the same topic — pair them under a shared rule heading.

2. **If guidelines.json is missing or has fewer than ~3 recommended entries for a section**, search the web for industry best practices. Prefer authoritative design-system sources: Material, Carbon, Atlassian, Primer, Spectrum, Polaris, Wise, Orbit. Synthesise — don't paste.

3. **Never invent a rule** without either a guidelines.json entry or a web source backing it.

### Synthesis rules

- **Rewrite in Kido voice.** Active, prescriptive, second-person where natural. Strip vendor-specific terms (don't say "Carbon Button"; say "the button").
- **No source attribution in the rendered prose.** If multiple sources agree, that's signal but not output.
- **Concise.** Bullets are one sentence each. Do/Don't pair descriptions are two sentences max (rule + reason).
- **Behavior vs Content split:** Behavior = states, interactions, icon usage, grouping. Content = labels, copy, length, casing.

---

## Step 5 — Output

Return one markdown document in the chat with this structure:

```markdown
# {Component Name} — Documentation

> Source: [Figma]({polished URL})
> Generated: {today's date}

## Variants

### {Variant Name 1}

{Description paragraph.}

**When to use:**
- {bullet}
- {bullet}

### {Variant Name 2}

…

## Usage Guidelines

### When to use
- {bullet}
- {bullet}

### When not to use
- {bullet}
- {bullet}

### General guidelines
- {bullet}
- {bullet}

### Behavior

**Do:** {rule heading}
{One-sentence rationale.}

**Don't:** {paired rule}
{One-sentence rationale.}

…

### Content

**Do:** {rule heading}
{Rationale.}

**Don't:** {rule heading}
{Rationale.}

…

### Accessibility
- {bullet}
- {bullet}
```

Trailing notes (after the document body):

- If the secondary variant axis is non-trivial (e.g., a `size` axis with 3+ values), append a one-paragraph **Sizes** note describing when to reach for each.
- If web research was used for any section, append a short **Sources consulted** list (URLs only — for the designer's reference, not part of the doc itself).

---

## Failure Modes

| Condition | Behavior |
|---|---|
| URL doesn't resolve / Figma access fails | Stop. Tell designer to verify the URL or auth (`/mcp` → `claude.ai Figma`). |
| Component name doesn't match any spec | Ask designer once to confirm the slug; if they can't, proceed with Figma-only data + web research. |
| guidelines.json absent | Use web research silently — don't mention the missing file. |
| Single-variant component (no semantic axis) | Skip the Variants section; produce only Usage Guidelines. |
| Polished component has obvious gaps (mislabeled variants, missing states) | Note the gap inline in the Variants section ("Figma exposes `tertiary` but no description was generated — variant appears identical to `secondary`; verify in canvas") and continue. |

---

## Critical Rules

**No canvas writes — ever.** This skill is read-only on Figma. If a future request asks for canvas docs, route to a different skill (the older canvas-rendering version of `/ds-doc` is retired).

**No files written.** All output is chat markdown. The designer copies what they want into spec notes, Figma rich-text, Notion, or wherever.

**Web search is unattended.** When guidelines.json is missing or thin, search and synthesise without asking. The designer expects best-practice prose, not a clarifying question.

**Synthesise, don't paste.** Competitor prose gets rewritten in Kido voice. Sources inform the writing, not the rendered output.

**Variants axis ≠ size/state axis.** Always describe the *semantic emphasis* axis. Sizes and states get at most a one-paragraph note at the end.
