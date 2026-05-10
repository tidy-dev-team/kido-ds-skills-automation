# CHANGES — {project}

> Per-project change log. Lives in `working/{project}/CHANGES.md`. Gitignored along with the rest of `working/`.
>
> Two categories. Every entry must be one or the other — no mixing.

---

## How to use this file

During a session, when you make a change to skills, specs, or templates that grew out of working on this project, decide which category it belongs to **before** writing it down:

- **Generic improvements** — the change applies to *any* future project. Things like a new self-audit check, a token decision rule, a fixed bug. These are candidates for the next PR to `main`.
- **Project-specific** — the change only makes sense inside this project. A custom token mapping, a one-off override, an exception note. These must **not** travel to `main`.

When the project finishes (or at any milestone), the **Generic improvements** section is what gets lifted into a PR against `main` (with a corresponding `CHANGELOG.md` entry). The **Project-specific** section stays here.

If a change feels like both — split it. Generic part goes up; project-specific note stays.

---

## Category 1 — Generic improvements (safe to merge to main)

> Add entries with: file touched, one-line summary, optional rationale.
> Format: `- {date} — \`path/to/file\` — what changed (why)`

_(none yet)_

---

## Category 2 — Project-specific (must not travel)

> Add entries with: scope (which file/component), one-line summary, why it's project-specific.
> Format: `- {date} — {scope} — what changed (why project-specific)`

_(none yet)_
