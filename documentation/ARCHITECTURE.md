# Architecture

This document describes the architecture of the DS component automation system. As of this writing, all eight tracked issues (#1–6, #8, #9) have shipped — the system below reflects the current state of the repo, not a future plan.

> **Status legend:** ✅ shipped · 🟡 in progress · ⏳ planned (issue link)

---

## 1. Goals

Two outcomes drive every design decision in this repo.

### Reliability

Generation must not silently omit parts of a component. Today, a Button run can produce variants with no Label property, no icon toggles, or fixed-width layouts — and the designer only finds out by inspecting Figma. The system should plan what it intends to produce, produce it, then verify the result against the plan, and surface any gaps.

### Accumulated knowledge

Hard-won learnings — token relationships, state ownership rules, library conventions, project-specific overrides — should live in files that skills read on every run, not be re-derived by the model each time. Knowledge moves from the model's exploration into the repo.

---

## 2. The three layers

```
┌─────────────────────────────────────────────────────────────┐
│  DATA layer — what components ARE                            │
│  specs/*.spec.json                                           │
│    variants, tokens, anatomy, component_properties (0.2 ✅)  │
└─────────────────────────────────────────────────────────────┘
                          ▲
                          │ skills READ
                          │
┌─────────────────────────────────────────────────────────────┐
│  RULES layer — how to THINK about design                     │
│  DESIGN-SYSTEM.md (repo root, authoritative when present)    │
│    token decision tree, state ownership, modes-as-brands    │
└─────────────────────────────────────────────────────────────┘
                          ▲
                          │ skills READ via Step 0 hook
                          │
┌─────────────────────────────────────────────────────────────┐
│  SKILLS layer — what to DO                                   │
│  ds-generate, ds-build, ds-push, ds-storybook, ...           │
│    plan → execute → self-audit                              │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
                    Figma component set
```

Each layer has one job. They never reach across.

### Data layer — `specs/`

Compact JSON describing *what a component is*. Variants, anatomy, tokens, component properties. Not instructions, just facts. Skills read this; they don't memorize it.

- `specs/{component}.spec.json` — per-component spec
- `specs/{component}.spec.notes.md` — human-readable rationale
- `specs/_index.json` — lookup table (name, aliases, description)
- `specs/libraries/*.json` — UI library conventions (Chakra, Mantine, shadcn) for Workflow B

The current schema is `0.2` ✅ ([#8](https://github.com/tidy-dev-team/kido-ds-skills-automation/issues/8)) — adds structured `anatomy.component_properties` so component-property types (TEXT / BOOLEAN / INSTANCE_SWAP) are data, not prose. Skills consume the field directly; the 0.1 → 0.2 heuristic bridge added in #1 has been removed from `ds-generate`.

### Rules layer — `DESIGN-SYSTEM.md`

Project-level knowledge that applies *across* components. Lives in the repo root when present. Skills read it via a Step 0 hook and treat it as authoritative — when a skill default conflicts with the file, the file wins.

The Step 0 hook is ✅ wired into `ds-generate` and `ds-build` (#6). The CLAUDE.md critical rules section also makes it standing for any skill running in the repo.

Contents ✅ ([#5](https://github.com/tidy-dev-team/kido-ds-skills-automation/issues/5)):
- Token decision tree (`static` vs `interactive`; `fg` / `bg` / `border`; `brand` / `muted` / `base` / `secondary`)
- State Ownership Rule (which layer owns hover/pressed; what stays static; radio button as canonical reference)
- Modes-as-brands
- Bootstrap state-name mapping
- Disabled family rule
- Interactive state token mapping (stroke-primary vs fill-primary vs bg-zone)
- Per-project overrides section at the file's tail (variable IDs, active modes)

### Skills layer — `skills/`

Each skill is an instruction file. Skills follow the same shape regardless of workflow:

1. **Plan** — derive a checklist from the spec (data) and rules (DESIGN-SYSTEM.md)
2. **Execute** — build it in Figma via MCP
3. **Self-audit** — read the result back, tick the checklist, attempt one corrective pass, surface remaining gaps

Skills:
- `ds-guide` — guided wizard entry point that routes to the right skill (optional, opt-in)
- `ds-spec-authoring` — author/maintain Kido specs (DS team)
- `ds-extract-design` — extract client tokens to `tokens.json` + `DESIGN.md` (Workflow B prerequisite)
- `ds-generate` — Workflow A: client has no UI library
- `ds-build` — Workflow B: client has an existing UI library
- `ds-push` — sync polished Figma values back to code
- `ds-storybook` — generate CSF3 stories
- `ds-doc` — build canvas documentation pages (Component Breakdown / Mode / Usage Guidelines) for a polished component; see [`docs/adr/0001-ds-doc-three-page-architecture.md`](../docs/adr/0001-ds-doc-three-page-architecture.md). Workflow B v1.

`ds-generate` and `ds-build` are the two workflow orchestrators. `ds-push`, `ds-storybook`, and `ds-doc` are the shared post-polish pipeline (`ds-doc` is Workflow B only for v1). `ds-guide` is a routing layer above all of them — it never duplicates their work.

---

## 3. The two workflows

The system handles two starting conditions for any client engagement.

### Workflow A — `ds-generate`

Client has **no** existing UI library. They want Kido's structure styled with their brand.

```
Client input (Figma frame / screenshot / site URL / repo)
    │
    ▼ extract brand values
ds-generate
    │
    ▼ apply Kido spec + brand values
Kido-structured component set in Figma
```

### Workflow B — `ds-build`

Client **already ships** a UI library (Chakra, Mantine, shadcn, custom). Figma should mirror it.

```
Client library (GitHub / Storybook) + tokens.json + DESIGN.md + REQUIREMENTS.md
    │
    ▼
ds-build
    │
    ▼
Library-matched component set in Figma + validation report
```

### Shared post-polish

Both workflows converge into the same downstream pipeline:

```
DS specialist polishes in Figma
    │
    ▼
ds-push       → GitHub PR (CSS variables, Tailwind, source)
    │
    ▼
ds-storybook  → GitHub PR (CSF3 stories)
```

---

## 4. Per-project artifacts

Per-client work lives in `working/{project}/`. Gitignored. Local only.

```
working/{project}/
  tokens.json            ← DTCG token tree, source of truth (Workflow B)
  DESIGN.md              ← spec-compliant front matter + prose rationale (Workflow B)
  REQUIREMENTS.md        ← per-job rules (Workflow B)
  library-snapshot.json  ← resolved library structure (Workflow B)
  token-map.json         ← resolved tokens (both workflows)
  resolved-stubs.json    ← designer-provided stub values (Workflow A)
  validation-report.md   ← validator output (Workflow B)
  push-summary.json      ← ds-push diff + PR URL
  CHANGES.md             ← project-specific changes (template at skills/templates/CHANGES.template.md)
```

The `CHANGES.md` convention ✅ ([#6](https://github.com/tidy-dev-team/kido-ds-skills-automation/issues/6)) splits each project's notes into:
- **Generic improvements** — safe to merge to main
- **Project-specific** — must not travel to other projects

Template lives at `skills/templates/CHANGES.template.md`. This is how project quirks survive without polluting `main` — when the project finishes, the **Generic** section is lifted into the next PR; **Project-specific** stays in the working dir.

---

## 5. The plan-execute-audit loop

This is the contract every generation skill follows. Established in `ds-generate` ([#1](https://github.com/tidy-dev-team/kido-ds-skills-automation/issues/1), ✅) and extended into `ds-build`'s validator and binding pass ([#4](https://github.com/tidy-dev-team/kido-ds-skills-automation/issues/4), ✅).

### Step — Plan (hard gate)

Before any Figma code runs, the skill emits a markdown checklist derived from the spec:

- **Variant axes** — full cartesian product from `variants`
- **Token bindings** — every color/spacing/radius slot bound to a semantic variable
- **Component properties** — Label / icons / etc., with their Figma property type
- **State anatomy** — one line per `completeness.validation` entry
- **Identity** — component set name, variant format, description

Items remain unchecked. Generation is gated on this output.

### Step — Execute

Build in dependency order: base variants → state variants → size variants → inverse (if requested). Stub anything unresolved as a bright-pink `NEEDS_VALUE`. Never drop a required variant silently.

### Step — Self-audit

Read the produced component set back via Figma MCP (`figma_get_component_details`, `figma_get_design_context`, `get_screenshot`). Tick each plan item:

- `[x]` pass
- `[~]` partial (with note)
- `[ ]` fail (with note)

One corrective pass is allowed. Anything still failing is surfaced explicitly in the final report — never hidden.

---

## 6. Where each tracked issue lives in the architecture

| Issue | Layer | What it adds |
|---|---|---|
| [#1](https://github.com/tidy-dev-team/kido-ds-skills-automation/issues/1) ✅ | Skills | Plan-execute-audit loop in `ds-generate` |
| [#2](https://github.com/tidy-dev-team/kido-ds-skills-automation/issues/2) ✅ | Skills | Fix variant resize bug; regression check in self-audit |
| [#3](https://github.com/tidy-dev-team/kido-ds-skills-automation/issues/3) ✅ | Skills | Icon-missing protocol (placeholder, never silent failure) |
| [#4](https://github.com/tidy-dev-team/kido-ds-skills-automation/issues/4) ✅ | Skills | Variable-binding parity for `ds-build` |
| [#5](https://github.com/tidy-dev-team/kido-ds-skills-automation/issues/5) ✅ | Rules | Token decision tree + authoritative DESIGN-SYSTEM.md content |
| [#6](https://github.com/tidy-dev-team/kido-ds-skills-automation/issues/6) ✅ | Rules / per-project | Step 0 hook + `CHANGES.md` two-category split |
| [#8](https://github.com/tidy-dev-team/kido-ds-skills-automation/issues/8) ✅ | Data | Spec schema 0.2 — structured `anatomy.component_properties` |
| [#9](https://github.com/tidy-dev-team/kido-ds-skills-automation/issues/9) ✅ | Skills | New `ds-guide` wizard — guided UX entry point that routes to the right skill |

---

## 7. Invariants

These hold across the whole system. If a change breaks one of them, the change is wrong.

- **Specs are data, not instructions.** No prose-encoded behavior. Skills derive from structured fields.
- **Skills derive, not memorize.** No per-component logic embedded in skill files. The skill reads the spec and produces output.
- **Stubs are valid output.** Visible `NEEDS_VALUE` is better than an invented value. Designers polish in Figma after generation.
- **Plan before execute. Audit after execute.** No silent omissions. No silent successes.
- **Project quirks live in `working/{project}/` and `DESIGN-SYSTEM.md`.** They never modify the generic skills.
- **One canonical glossary.** When skills disagree with `LANGUAGE.md`, the glossary wins; update the skill.

---

## 8. Pointers

- `CLAUDE.md` — entry point for any agent working in this repo
- `LANGUAGE.md` — canonical glossary (terms, roles, paths, verbs)
- `README.md` — human-facing overview
- `CHANGELOG.md` — per-change log under `## [Unreleased]`
- `skills/templates/QUALITY_STANDARDS.md` — Kido DS baseline applied to every component
- `skills/templates/REQUIREMENTS.template.md` — starting point for per-job rules
