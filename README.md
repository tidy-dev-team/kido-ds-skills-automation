# DS Component Automation

> Turn partial client inputs into production-ready Figma component sets, and keep code and Figma in sync automatically — using Kido DS expertise as the foundation.
> Built by Tidy Dev. Powered by Claude + Figma MCP.

---

## The Big Idea

Designers do the same work over and over for every new client: taking a handful of reference designs and expanding them into a full production-grade component library in Figma. 54 variants per component. 12 components per project. Every project.

This repo turns that expansion into a skill system. Claude reads Kido's design-system expertise (encoded in specs and skill files), reads whatever the client has provided (Figma, site, repo, library), and builds the components directly in Figma.

There are **two ways a client shows up** — and each gets its own workflow.

---

## The Two Workflows

### A · Client has no UI library yet → **`/ds-generate`**

The client has brand direction (a logo, a primary color, a landing page, maybe a button design) but no production component library. You want to give them Kido's battle-tested component structure, styled with their brand values.

```
Client Figma frame / screenshot / site / repo
                    ↓
           Extract brand values
            (color, radius, font)
                    ↓
        Map to Kido spec token slots
                    ↓
         Build in Figma — 54 variants
      (Kido structure + client brand)
```

### B · Client already has a UI library → **`/ds-build`**

The client ships a React library (Chakra, Mantine, shadcn/ui, or their own) and you want the Figma components to mirror exactly what their engineers use — their variant names, their size scale, their prop axes. Meanwhile, Kido's design tokens go on top so the styling stays consistent with Kido's opinions.

```
 Client library (GitHub / Storybook)  →  library structure
                                           +
          Kido foundation Figma       →  tokens.json (DTCG) + DESIGN.md (rationale)
                                           +
              Designer + PM           →  REQUIREMENTS.md (job rules)
                                           ↓
                                      Build in Figma
                                           ↓
                                  Validator (soft-advisory)
                                           ↓
                              Library-matched component set
```

### Shared post-polish — **`/ds-push`** + **`/ds-storybook`**

Once the DS specialist polishes the generated component in Figma, two skills close the loop:

- **`/ds-push`** — reads resolved Figma values, surgically updates CSS variables / Tailwind config / component source in the client repo, opens a PR.
- **`/ds-storybook`** — generates a CSF3 story file from the component's Figma axes and the codebase's prop interface, opens a PR.

Both are workflow-agnostic — they don't care whether the component came from Workflow A or B.

---

## The Seven Skills at a Glance

| Skill | Who | When | Output |
|---|---|---|---|
| `/ds-guide` | Anyone | Unsure where to start | Routes to the right skill |
| `/ds-spec-authoring` | DS team | New Kido component, one-time | `specs/{component}.spec.json` |
| `/ds-extract-design` | Designer | Start of Workflow B project | `working/{project}/tokens.json` + `DESIGN.md` |
| `/ds-generate` | Designer | Per client project, Workflow A | Figma component set |
| `/ds-build` | Designer | Per component, Workflow B | Figma component set + validation report |
| `/ds-push` | DS specialist | After Figma polish | GitHub PR (token sync) |
| `/ds-storybook` | DS specialist / dev | After tokens pushed | GitHub PR (CSF3 stories) |

`/ds-guide` is the optional guided entry point — it asks a short series of questions and invokes the right skill for you. Direct invocation of any individual skill works exactly the same as before.

Each skill has its own file in `skills/` — **those files are authoritative**. CLAUDE.md and this README are overviews.

For shared vocabulary (terms, roles, artifact paths, verbs, skill boundaries), see [`LANGUAGE.md`](./LANGUAGE.md). The glossary wins when skills disagree.

For token logic (decision tree, State Ownership Rule, Bootstrap mapping, modes-as-brands), see [`DESIGN-SYSTEM.md`](./DESIGN-SYSTEM.md). Every skill reads it at Step 0; the file overrides skill defaults when they conflict.

For the layered architecture (data → rules → skills) and where each issue fits, see [`documentation/ARCHITECTURE.md`](./documentation/ARCHITECTURE.md).

---

## Quick Start — Workflow A (no client library)

```
1. Designer opens Claude Code in this repo.
2. /ds-generate "Generate this button: [Figma URL / screenshot / site / repo]"
3. Claude identifies the component, extracts brand values, maps to Kido tokens,
   and builds 54 variants directly in the designer's Figma file.
4. DS specialist polishes in Figma (fills pink stubs, refines visuals).
5. DS specialist runs /ds-push "Push the polished button: [Figma URL] [GitHub repo URL]"
   → GitHub PR opens with updated CSS vars / Tailwind config.
6. DS specialist runs /ds-storybook "Add stories: [GitHub repo URL]"
   → GitHub PR with {component}.stories.tsx.
```

Typical end-to-end: 30 minutes from client input to open PRs.

## Quick Start — Workflow B (client has a library)

```
1. Extract design foundation once per project:
   /ds-extract-design "Extract tokens from: [Kido foundation Figma URL]"
   → working/{project}/tokens.json   (DTCG, source of truth)
     working/{project}/DESIGN.md     (front matter + prose)

2. Build each component:
   /ds-build "Build a Button for [library GitHub/Storybook URL]"

   • Claude reads/creates working/{project}/REQUIREMENTS.md
     (pastes if provided, interviews designer if not — modes, locales, prefix, a11y)
   • Claude reads the client library (Storybook stories.json or gh api repo source)
   • Claude maps tokens.json values onto the library's prop/variant axes
   • Claude builds the component in Figma matching the library's structure
   • Validator subagent runs a 9-category checklist → validation-report.md
   • Designer reviews; decides whether to fix issues or accept

3. Post-polish: same /ds-push and /ds-storybook as Workflow A.
```

---

## Inputs Accepted by Each Workflow

**`/ds-generate`** takes any of:

| Input | How Claude reads it |
|---|---|
| Figma URL | `get_design_context` via Figma MCP |
| Screenshot | Visual analysis |
| Description | Text extraction |
| Live site URL | `WebFetch` — CSS custom properties |
| GitHub repo URL | `gh api` — `index.css`, `tailwind.config.ts`, component source |

**`/ds-build`** takes a component name + one or more of:

| Input | How Claude reads it |
|---|---|
| Storybook URL | `WebFetch` `{url}/stories.json` or `/index.json` |
| GitHub repo URL | `gh api` — component source + `package.json` |
| `tokens.json` (from `/ds-extract-design`) | Local file read — token values |
| `DESIGN.md` (from `/ds-extract-design`) | Local file read — rationale, do's/don'ts |
| REQUIREMENTS.md | Local file read, or interview Step 0 |

---

## File Layout

```
specs/
  _index.json                          ← Kido component lookup
  {component}.spec.json                ← Kido spec (compact)
  {component}.spec.notes.md            ← rationale + history
  libraries/
    _index.json                        ← library mapping lookup
    chakra.json | mantine.json | …     ← UI library conventions

skills/
  ds-spec-authoring.md
  ds-generate.md
  ds-extract-design.md
  ds-build.md
  ds-push.md
  ds-storybook.md
  templates/
    REQUIREMENTS.template.md           ← per-job rules template
    QUALITY_STANDARDS.md               ← Kido DS baseline (applies to every component)

working/                               ← gitignored per-session artifacts
  {component}-{YYYY-MM-DD}/            ← Workflow A (per generation session)
    token-map.json                     ← /ds-generate
    resolved-stubs.json                ← /ds-generate
    push-summary.json                  ← /ds-push (Workflow A)
  {project}/                           ← Workflow B (per project, shared across components)
    tokens.json                        ← /ds-extract-design (DTCG, source of truth)
    DESIGN.md                          ← /ds-extract-design (front matter + prose)
    REQUIREMENTS.md                    ← /ds-build
    library-snapshot.json              ← /ds-build
    token-map.json                     ← /ds-build
    validation-report.md               ← /ds-build (validator)
    push-summary.json                  ← /ds-push (Workflow B)

.claude/
  commands/                            ← /slash-command wiring
    ds-*.md

CLAUDE.md                              ← agent instructions (auto-loaded)
LANGUAGE.md                            ← canonical glossary (terms, roles, paths, verbs)
README.md                              ← this file
```

---

## Core Concepts

**Spec** (Kido's)
A compact JSON file describing a Kido DS component: variant axes, tokens, sizing, accessibility. Used by `ds-generate` as the structure source. Authored once by the DS team via `/ds-spec-authoring`.

**`tokens.json`** (per-project)
A W3C DTCG token tree — colors (primitives + semantic), typography, spacing, rounded, shadow, components, themes. Extracted by `/ds-extract-design` from a Figma foundation file at the start of each Workflow B project. Source of token truth, consumed by `/ds-build`. Lives in `working/` (not committed).

**`DESIGN.md`** (per-project)
The human/agent-readable companion to `tokens.json`. Front matter is a flat token summary conforming to the [google-labs-code/design.md](https://github.com/google-labs-code/design.md) spec (lints cleanly with `npx @google/design.md lint`). Prose body covers palette philosophy, typography intent, do's and don'ts, gaps. Lives in `working/` next to `tokens.json` (not committed). The two files always travel together.

**REQUIREMENTS.md** (per-job)
A Markdown file describing job-specific rules — color modes required, locales, naming prefixes, accessibility level, any component-specific constraints. Either pasted in by the designer or generated via a short interview at the start of `/ds-build`.

**Library mapping** (`specs/libraries/*.json`)
A small committed reference describing a UI library's variant prop names, size scale, and compound component conventions. Loaded by `/ds-build` when a library is detected from a `package.json`. Currently shipped: Chakra, Mantine, shadcn.

**Stubs (`#FF0066`)**
When Claude can't resolve a value from the input, it places a bright pink fill in Figma with a label describing what's needed. The DS specialist fills these in during polish. Stubs are expected and valid — they never block generation.

**Validator (Workflow B only)**
A subagent inside `/ds-build` that reviews the generated component against a 9-category checklist: structure fidelity, token application, variant completeness, naming conventions, accessibility, mode/locale constraints, auto-layout, component properties, and component description. Writes `validation-report.md`. Soft advisory — doesn't block completion.

---

## Design Principles

**Two workflows, one post-polish.** `ds-generate` and `ds-build` split the front half based on client context. `ds-push` and `ds-storybook` are shared and unaware of which path produced the component.

**Skills are authoritative.** Skill files in `skills/` hold the real instructions. CLAUDE.md and README are overviews that may lag.

**Never fabricate.** Missing values become stubs (pink fills) or `NEEDS_VALUE` entries. A visible gap is better than an invented value.

**Preserve the client's font.** Extract it; find closest available match in Figma; never silently default to Inter.

**Kido rules in Workflow A.** Client brand on Kido structure.
**Library rules in Workflow B.** Kido tokens on library structure.

**Compact generation by default.** Workflow A generates `inverse=no` only unless dark-background support is explicitly requested.

**Validator is advisory, not a gate.** It catches issues but the designer decides what to do about them.

**Specs are data.** Token names are self-documenting. Claude reasons about derivation patterns; the spec doesn't need to spell them out.

**Closed loop.** Figma → code via `/ds-push`. Code → Figma via `/ds-generate` or `/ds-build`. No manual translation either direction.

---

## Requirements

| Requirement | Purpose |
|---|---|
| Claude Code | Runs the workflows; auto-loads CLAUDE.md |
| Figma MCP connected | Reads and writes Figma for all skills |
| `gh` CLI authenticated (`gh auth login`) | Reading client repos, opening PRs |
| Optional — Storybook running for client | Used by `/ds-build` when client provides a Storybook URL |

---

## Limitations

- **No Kido spec = no `/ds-generate`.** Novel components need `/ds-spec-authoring` first (one-time cost).
- **Unknown library = fallback to Kido spec in `/ds-build`.** Claude notes the deviation in `token-map.json`. Ship a `specs/libraries/{name}.json` mapping to upgrade fidelity.
- **Inverse variants often stubbed** in Workflow A — clients rarely provide dark designs upfront.
- **Screenshot input extracts less precisely than Figma URL** in Workflow A.
- **Validator is advisory.** It catches common errors; it does not guarantee correctness. Human review is still required.
- **Kido drift** — run `/ds-spec-authoring check [component]` if Kido DS has changed. Stale specs produce off-brand output.

---

*The goal: generating the tenth client button takes five minutes. Keeping code and Figma in sync takes five more.*
