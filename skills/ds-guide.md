---
name: ds-guide
description: >
  Guided wizard entry point for the DS automation system. Run this when a designer
  doesn't know which skill to invoke. Asks one question and routes to ds-generate,
  ds-build, ds-extract-design, ds-push, ds-storybook, or ds-spec-authoring.
  Triggers: "/ds-guide", "guide me", "I don't know where to start", "which skill do I use".
---

# DS Guide

A thin router. `ds-guide` is **one** way to enter the system — direct invocation of `/ds-generate`, `/ds-build`, etc. works exactly as before and is preferred when the user knows what they want. The wizard exists for first-time users.

This skill **routes**. It does not collect inputs and does not duplicate target-skill logic. Once the user picks an action, the wizard invokes the target skill via `Skill` with no arguments; the target skill runs its own intake.

---

## Flow

Ask one question with `AskUserQuestion`. Header: `Action` · multiSelect: false.

| Option label | Description shown to user | Routes to |
|---|---|---|
| Generate — client has **NO** UI library | Use this if the client has a styled site, Figma frame, or brand guide but no installable component package. | `ds-generate` |
| Generate — client **HAS** a UI library | Use this if their codebase imports components from a package (Mantine, Chakra, shadcn, `@acme/ui`, etc.). | `ds-build` |
| Push polished Figma back to code | Sync resolved values from a polished Figma component to the client repo as a PR. | `ds-push` |
| Add Storybook stories | Generate a CSF3 stories file for a production-ready component. | `ds-storybook` |

The auto-added **Other** option catches the two rarer routes:

- Free text mentioning **extract**, **tokens**, **DESIGN.md**, or a Figma foundation file → `ds-extract-design`
- Free text mentioning **spec**, **Kido DS**, or **spec-authoring** → `ds-spec-authoring`

If the free text is ambiguous, ask once more with the two leftover options as a follow-up `AskUserQuestion`. If it doesn't match any of the six routes, reply: "There's no skill for that yet." and stop.

---

## Routing

For each route, invoke the target skill with no arguments:

```
Skill(skill="ds-generate",         args="")
Skill(skill="ds-build",            args="")
Skill(skill="ds-push",             args="")
Skill(skill="ds-storybook",        args="")
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
