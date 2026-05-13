---
name: ds-guide
description: >
  Guided wizard entry point for the DS automation system. Run this when a designer
  doesn't know which skill to invoke. Asks up to two questions (Q1 picks an action
  bucket; Q2 disambiguates when needed) and routes to ds-generate, ds-build,
  ds-extract-design, ds-push, ds-storybook, ds-doc, or ds-spec-authoring.
  Triggers: "/ds-guide", "guide me", "I don't know where to start", "which skill do I use".
---

# DS Guide

A thin router. `ds-guide` is **one** way to enter the system — direct invocation of `/ds-generate`, `/ds-build`, etc. works exactly as before and is preferred when the user knows what they want. The wizard exists for first-time users.

This skill **routes**. It does not collect inputs and does not duplicate target-skill logic. Once the user picks an action, the wizard invokes the target skill via `Skill` with no arguments; the target skill runs its own intake.

---

## Flow

Two questions max. Q1 is always shown; Q2 only fires when Q1's choice covers two skills.

### Q1 — Action

`AskUserQuestion`. Header: `Action` · multiSelect: false. Four options, ordered by how often a designer reaches them (most common first):

| Option label | Description shown to user | Next step |
|---|---|---|
| Generate a Figma component set | Build a new component set in Figma from client inputs. I'll ask one more question to figure out whether the client has a UI library. | → Q2a |
| Push to repo, add Storybook stories, or build canvas docs | The component is built and polished in Figma. I'll ask which: push resolved token values to the client repo, generate a CSF3 stories file, or build the three canvas doc pages. | → Q2b |
| Extract design tokens from a Figma foundation file | Read a Kido or client foundation Figma file and emit `tokens.json` + `DESIGN.md` for a Workflow B project. Run once at the start of each project. | → `ds-extract-design` |
| Author or update a Kido DS component spec (DS team) | Maintained by the DS team only — a per-project designer rarely needs this. Authors `specs/{component}.spec.json` + notes for a Kido component. | → `ds-spec-authoring` |

### Q2a — only if Q1 was "Generate a Figma component set"

`AskUserQuestion`. Header: `Library` · multiSelect: false.

| Option label | Description shown to user | Routes to |
|---|---|---|
| Client has **NO** UI library | Workflow A → `ds-generate`. Styled site / Figma frame / brand guide / repo, but no installable component package. Kido structure + client brand values. | `ds-generate` |
| Client **HAS** a UI library | Workflow B → `ds-build`. Their codebase imports components from a package (Mantine, Chakra, shadcn, `@acme/ui`, etc.). Figma mirrors the library structure. | `ds-build` |

### Q2b — only if Q1 was the post-polish bucket

`AskUserQuestion`. Header: `Direction` · multiSelect: false.

| Option label | Description shown to user | Routes to |
|---|---|---|
| Push values to repo | `ds-push`. Sync resolved CSS variables, Tailwind config, and/or component source from the polished Figma component to the client repo as a PR. | `ds-push` |
| Add Storybook stories | `ds-storybook`. Generate a CSF3 `{component}.stories.tsx` file for a production-ready component. | `ds-storybook` |
| Build canvas doc pages | `ds-doc`. Build the three Figma doc pages (Component Breakdown / Mode / Usage Guidelines) on a `📄 Documentation` page in the target file. Workflow B only. | `ds-doc` |

### Other

`AskUserQuestion`'s auto-added **Other** option catches off-script free text. Map common phrases:

- **extract / tokens / DESIGN.md / tokens.json / foundation** → `ds-extract-design`
- **spec / Kido DS / spec-authoring** → `ds-spec-authoring`
- **generate / build / component set** → ask Q2a
- **push / sync / PR** → `ds-push`
- **stories / Storybook / CSF3** → `ds-storybook`
- **doc / docs / documentation / canvas docs / doc page / breakdown / usage guidelines** → `ds-doc`

If the free text is ambiguous, ask once more with the two closest options as a follow-up `AskUserQuestion`. If it doesn't match any of the seven routes, reply: "There's no skill for that yet." and stop.

---

## Routing

For each route, invoke the target skill with no arguments:

```
Skill(skill="ds-generate",         args="")
Skill(skill="ds-build",            args="")
Skill(skill="ds-push",             args="")
Skill(skill="ds-storybook",        args="")
Skill(skill="ds-doc",              args="")
Skill(skill="ds-extract-design",   args="")
Skill(skill="ds-spec-authoring",   args="")
```

Before invoking, send a one-line handoff: "Routing to `{skill}`. It'll ask what it needs." The target skill is responsible for collecting component name, project slug, URLs, and any prerequisite checks (`tokens.json`, `DESIGN.md`, `REQUIREMENTS.md`, etc.).

---

## What this skill does not do

- **Does not collect inputs upfront.** Component names, URLs, slugs, library references — the target skill asks for these. Coupling the wizard to target-skill arg shapes makes every skill rename ripple here.
- **Does not check prerequisites.** `ds-build` decides whether `tokens.json` / `DESIGN.md` are present. `ds-extract-design` decides whether the Figma URL is valid. The wizard does not pre-flight.
- **Does not maintain session state.** If the user has been talking about a component earlier in the conversation, the target skill picks that up — not the wizard.
- **Does not paraphrase target skills.** Don't tell the user what `ds-build` is going to do; let `ds-build` describe itself.

---

## When to use vs. direct invocation

- First-time user, doesn't know skill names → `/ds-guide`.
- Knows the skill → `/ds-{skill}` directly. The wizard saves zero time for an experienced user.
