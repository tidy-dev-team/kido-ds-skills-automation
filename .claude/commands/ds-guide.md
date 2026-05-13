---
description: Guided wizard entry point for the DS automation system. Asks up to two questions (Q1 picks an action bucket: Generate, Post-polish, Extract tokens, or Author Kido spec; Q2 disambiguates when needed — A vs B for Generate; push / stories / docs for Post-polish) and routes to ds-generate, ds-build, ds-extract-design, ds-push, ds-storybook, ds-doc, or ds-spec-authoring. Pure router — the target skill collects its own inputs. Direct invocation of any individual skill remains supported. Arguments: none.
---

Read `skills/ds-guide.md` and follow its instructions.

ARGUMENTS: $ARGUMENTS
