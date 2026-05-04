# LANGUAGE.md — Design System Skills

The shared vocabulary for this project. Every skill, command, spec, and doc must use these terms exactly as defined here. If a concept is missing, add it here first, then update the skills — not the other way around.

When two terms might mean the same thing, this file picks one. The other is a **deprecated alias** and should be replaced on sight.

---

## Workflows

### Workflow A (`/ds-generate`)
The pipeline for clients **without an existing UI library**. Uses Kido's component structure styled with the client's brand values. Trigger: client supplies a Figma frame, screenshot, live site URL, repo, or description — but no shippable component library.

### Workflow B (`/ds-build`)
The pipeline for clients **with an existing UI library** (Chakra, Mantine, shadcn, custom). Uses the library's structure styled with tokens from `DESIGN.md`. Trigger: client ships a UI library in production and wants Figma to mirror it.

### Shared post-polish
The stages after either workflow finishes generation: **polish** (manual, in Figma) → **`/ds-push`** (PR with token values) → **`/ds-storybook`** (PR with stories). Identical for both workflows.

> Always pair the workflow letter with the slash command on first reference: "Workflow A (`/ds-generate`)". Avoid bare "the generate workflow" or "the build path."

---

## Roles

### Designer
The human running skills end-to-end on a client project. Owns `/ds-extract-design`, `/ds-generate`, `/ds-build`. May also run `/ds-push` and `/ds-storybook`.

### DS specialist
The human who **polishes** generated component sets in Figma (fills stubs, refines visuals) and is the primary owner of `/ds-push`. May overlap with Designer on small projects.

### DS team
The Kido design system maintainers. The **only** role that runs `/ds-spec-authoring`. Not a per-project role.

### Developer
Engineer on the client side. May run `/ds-storybook` if the DS specialist hands off. Not a workflow actor otherwise.

### Client
The end company whose brand the work is being done for. **Not** a workflow actor — never instruct or address the client. When you mean "the client's React package," say **UI library** instead.

### Validator (subagent)
Automated subagent run at the end of `/ds-build`. Emits the **validation report** as a soft advisory. Never a human role.

> "Designer" and "DS specialist" are distinct. If a step says "designer," a DS specialist may also do it; if a step says "DS specialist," a designer should not unless explicitly authorized.

---

## Glossary

### Component set
A Figma node containing every variant combination for a single component. Default Workflow A output is 54 variants (compact); full output is 108.
**Deprecated aliases:** "variant set", "component group", "set of variants".

### Variant
A single combination of property values within a component set. Not the same as a "component" or a "version."

### Spec
A compact JSON description of a Kido DS component, stored as `specs/{component}.spec.json` with notes in `specs/{component}.spec.notes.md`. The authoritative structure source for Workflow A.
**Aliases acceptable on first reference:** "Kido spec", "component spec". Never just "the JSON."

### Foundation file
A Figma file containing the **design tokens** (variables + styles) for a project. Read by `/ds-extract-design`. May be a Kido foundation, a client foundation, or both.
**Deprecated aliases:** "tokens file", "foundation export".

### Design tokens
The named, semantic values (colors, typography, spacing, radius, shadow) that drive component styling. Always **DTCG-formatted** when serialized.

### `DESIGN.md`
The per-project artifact written by `/ds-extract-design` to `working/{project}/DESIGN.md`. Markdown body with embedded DTCG JSON blocks. The single source of token truth for Workflow B.
Always uppercase, always `.md`. Never "design.md", "tokens.md", "foundation.md".

### `REQUIREMENTS.md`
The per-project per-job rules file at `working/{project}/REQUIREMENTS.md`. Authored by interview at the start of `/ds-build`. Captures modes, locales, prefix, a11y level, scope overrides.
**Deprecated aliases:** "job rules file", "the requirements doc".

### Quality Standards (Kido DS)
The baseline Figma hygiene rules (auto-layout, naming, tokens & styles usage, components, icons, typography, responsiveness, nesting, descriptions, pre-handoff checklist). Currently embedded in `REQUIREMENTS.template.md`; should be referenced as **"Quality Standards"** distinct from per-job requirements.

### UI library
The client's shipped React component package (Chakra, Mantine, shadcn, custom). Distinct from:
- **Library mapping** — the static convention file at `specs/libraries/{library}.json`.
- **Library snapshot** — the resolved runtime file at `working/{project}/library-snapshot.json`.

Never use bare "library" — always disambiguate which of the three.

### Token map
The component-level mapping of visual slots to semantic tokens. Stored as `working/{project-or-component}/token-map.json`. Written by `/ds-generate` and `/ds-build`; read by `/ds-push`.
The internal `slot_mapping` key is a sub-concept ("slot → semantic token"), not a synonym.

### Slot
A named visual surface within a component (e.g., `background`, `label`, `icon`, `border`). Slots map to semantic tokens. Not the same as a "layer" or "node."

### Stub
A visible bright-pink (`#FF0066`) placeholder in Figma for an unresolved value. The data-layer marker is `NEEDS_VALUE` (in `token-map.json` and specs). A stub is **valid output** — never an error. Designers fill stubs during **polish**.
**Deprecated aliases:** "placeholder", "pink fill", "missing value marker."

### Polish
The manual Figma refinement step performed by the **DS specialist** between generation and `/ds-push`. Includes filling stubs, refining visuals, adjusting layout. Not a skill — a human activity.
**Deprecated aliases:** "refinement", "review and polish", "DS review."

### Validation report
The soft-advisory checklist emitted by the validator subagent at the end of `/ds-build`. Stored as `working/{project}/validation-report.md`. Errors **never block** completion.
Validator currently has **9 categories** (see `ds-build.md` Step 4). Older docs reference "6 categories" — that count is outdated.

### Working directory
Local, gitignored session storage:
- **Workflow A:** `working/{component}-{YYYY-MM-DD}/` (one per generation session)
- **Workflow B:** `working/{project}/` (one per project, shared across components)

`{project}` is a short slug the designer names at session start (e.g., `acme`, `payzo`). Never abbreviated to `{job}`.

### Compact generation / full generation
Workflow A output sizing. **Compact** (default) = 54 variants, `inverse=no` only. **Full** = 108 variants, includes inverse for dark backgrounds. Always say "compact" or "full" — not "small/large" or "minimal/complete."

### Brand values
The client-supplied visual primitives extracted in Workflow A: brand colors, radius, font, semantic accents. Distinct from **design tokens** (which are the named, structured representation).

### Inline validation
The lightweight in-skill checks performed by `/ds-generate` (e.g., font availability, token coverage). Distinct from the **validator subagent** that runs in `/ds-build`.

---

## Artifacts (canonical paths)

| Artifact | Path | Owner skill |
|---|---|---|
| Kido spec | `specs/{component}.spec.json` | `/ds-spec-authoring` |
| Spec notes | `specs/{component}.spec.notes.md` | `/ds-spec-authoring` |
| Spec index | `specs/_index.json` | `/ds-spec-authoring` |
| Library mapping | `specs/libraries/{library}.json` | (static; maintained by DS team) |
| Library mapping index | `specs/libraries/_index.json` | (static) |
| `DESIGN.md` | `working/{project}/DESIGN.md` | `/ds-extract-design` |
| `REQUIREMENTS.md` | `working/{project}/REQUIREMENTS.md` | `/ds-build` |
| Library snapshot | `working/{project}/library-snapshot.json` | `/ds-build` |
| Token map (B) | `working/{project}/token-map.json` | `/ds-build` |
| Token map (A) | `working/{component}-{YYYY-MM-DD}/token-map.json` | `/ds-generate` |
| Resolved stubs | `working/{component}-{YYYY-MM-DD}/resolved-stubs.json` | `/ds-generate` |
| Validation report | `working/{project}/validation-report.md` | `/ds-build` (validator) |
| Push summary | `working/{component-or-project}/push-summary.json` | `/ds-push` |
| Storybook output (local) | `working/{component-or-project}/storybook.tsx` | `/ds-storybook` |
| Stories file (in repo) | `{component}.stories.tsx` | `/ds-storybook` |

All `working/` files are gitignored. All `specs/` files are committed.

### PR branch naming
- `/ds-push`: `ds/push-{component}-{YYYY-MM-DD}`
- `/ds-storybook`: `ds/storybook-{component}-{YYYY-MM-DD}`

Always `ds/{action}-{component}-{date}`. Avoid `ds-sync-…`, `ds-push-…`, or other variants.

---

## Verbs (canonical usage)

Each verb has one job. Don't substitute.

| Verb | Means | Used by |
|---|---|---|
| **extract** | Read raw values from a source (Figma, CSS, repo, screenshot). | `/ds-extract-design`, `/ds-generate` (Step 2), `/ds-spec-authoring`, `/ds-push` (Step 1) |
| **identify** | Determine which Kido component matches a target. **Not** "classify." | `/ds-generate` Step 1, `/ds-spec-authoring` Step 1 |
| **map** | Bind a slot to a semantic token, or a Figma variant axis to a React prop. | All workflows |
| **resolve** | Follow alias chains or look up references to produce a concrete value. A **resolved value** is final. | `/ds-build` (library, tokens), `/ds-push` (Figma values) |
| **generate** | Produce a Figma component set in Workflow A; produce a CSF3 stories file. | `/ds-generate`, `/ds-storybook` |
| **build** | Produce a Figma component set in Workflow B. **Never used for Workflow A.** | `/ds-build` |
| **polish** | Manual Figma refinement by the DS specialist. **Never** "review", "refine", "clean up." | post-generation, pre-push |
| **validate** | Run checks against generated output. Qualify as **inline validation** (in-skill) or **validator subagent** (`/ds-build` Step 4). | `/ds-build`, `/ds-generate` |
| **review** | Always qualify: **validator review**, **designer review**, **spec review**. Never bare "review." | various |
| **push** | Open a GitHub PR carrying changes. Both `/ds-push` and `/ds-storybook` push. | `/ds-push`, `/ds-storybook` |
| **sync** | The conceptual Figma↔code loop. **Not** a synonym for push. | conceptual only |
| **author** | Create a Kido spec. Reserved for `/ds-spec-authoring`. | `/ds-spec-authoring` |
| **interview** | The Q&A that produces `REQUIREMENTS.md`. | `/ds-build` Step 0 |
| **discover** | Find files in a target repo via `gh api`. **Not** for Figma reads (use *extract*). | `/ds-push`, `/ds-storybook` |
| **stub** | Mark a value as `NEEDS_VALUE`, rendered as `#FF0066` in Figma. Both verb and noun. | all generation |
| **classify** | **Banned.** `/ds-generate` explicitly does not classify frames as component types. Use *identify*. | — |

---

## Skill boundaries (separation rules)

Each skill owns one stage. If two skills appear to do the same thing, one of them is wrong.

| Skill | Owns | Never does |
|---|---|---|
| `/ds-spec-authoring` | Authoring/updating Kido DS specs (`specs/*.json`). | Per-project work. Reading client foundations. |
| `/ds-extract-design` | Reading **foundation files** → `DESIGN.md`. | Reading specific components. Building component sets. |
| `/ds-generate` | Workflow A end-to-end: identify → extract brand values → generate Figma component set. | Reading from `DESIGN.md`. Building from a UI library. Pushing to GitHub. |
| `/ds-build` | Workflow B end-to-end: interview → resolve library → map tokens from `DESIGN.md` → build Figma component set → validator. | Extracting tokens (delegated to `/ds-extract-design`). Authoring specs. |
| `/ds-push` | Reading polished Figma values → opening a PR that updates **token values** (CSS vars, Tailwind config, component class swaps). | Adding new files. Changing component structure. Generating stories. |
| `/ds-storybook` | Reading Figma variant axes → opening a PR that adds `{component}.stories.tsx`. | Updating tokens. Editing existing files except stories config. |

### Known overlap watchpoints

These are points where boundaries are easy to blur — call them out explicitly when working in either skill:

1. **`/ds-build` falling back to a Kido spec** when no library mapping resolves. At that moment `/ds-build` looks like `/ds-generate`. Boundary remains: `/ds-build` consumes `DESIGN.md`; `/ds-generate` consumes brand values directly. The presence of `DESIGN.md` is the dividing line.
2. **Token extraction** appears in four skills (`/ds-extract-design`, `/ds-generate`, `/ds-spec-authoring`, `/ds-push`). Each reads from a different source for a different purpose — `/ds-extract-design` from foundations, `/ds-generate` from brand inputs, `/ds-spec-authoring` from a Kido component, `/ds-push` from a polished component. The verb `extract` is shared; the source and target are not.
3. **Both `/ds-push` and `/ds-storybook`** open GitHub PRs. Use the canonical branch naming above; never share branches.
4. **"No matching spec" handoff.** Both `/ds-generate` and `/ds-build` may prompt the user to run `/ds-spec-authoring` first. The wording should match across both: *"No Kido spec exists for {component}. Author one with `/ds-spec-authoring`, or proceed with the closest match?"*

---

## Maintenance

When a skill introduces a new term, add it here in the same PR. When two skills disagree, this file wins — update the skill, not the glossary. Deprecated aliases stay in the glossary (with the strikethrough or "Deprecated aliases" line) so old transcripts and PRs remain interpretable.
