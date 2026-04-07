---
name: ds-push
description: >
  Read a polished Figma component set and push updated token values back to the connected
  GitHub repository. Use this skill after a DS specialist has reviewed and polished a
  generated component in Figma and the code needs to reflect those final values.
  Triggers on phrases like "push to github", "sync to code", "update the repo from Figma",
  "close the loop", "the component is ready, push it", or when a designer shares a polished
  Figma component URL and a GitHub repo URL.
---

# DS Push

**Figma in, code out.**

Read the polished component set → extract resolved token values → update repo files → open PR.
The designer reviews Figma. Claude handles the diff and the push.

---

## What This Skill Does (and Does Not Do)

**Does:** Extract resolved token values from a polished Figma component set and surgically update
the target repo's CSS variables, Tailwind config, and component source to match.

**Does not:** Rewrite component logic, restructure files, or update code that wasn't touched
by the Figma polish pass. Only the values change — the structure stays.

---

## Inputs Required

- **Figma URL** — the polished component set node (e.g. `?node-id=2009:110`)
- **GitHub repo URL** — the target codebase (e.g. `https://github.com/owner/repo`)
- **Session token-map.json** (optional) — from `working/{component}-{date}/token-map.json`.
  If present, used to diff and push only changed values. If absent, push all extracted values.

---

## Step 1 — Read the Polished Component Set

Use `get_design_context` on the Figma component set node to extract resolved token values.

**Target variants to read:**

| Variant | What to extract |
|---|---|
| `type=contained, size=m, state=idle` | fill → primary color; text fill → fg on primary; cornerRadius → border_radius; fontName → font_family + weight; letterSpacing, textCase |
| `type=contained, size=m, state=hover` | fill → hover bg color |
| `type=contained, size=m, state=pressed` | fill → pressed bg color |
| `type=contained, size=m, state=focused` | stroke → focus ring color |
| `type=contained, size=m, state=disabled` | fill → bg-disabled |
| `type=outlined, size=m, state=idle` | stroke → border-idle color |

Read all fills as hex. Convert RGBA opacity to token notation (e.g. `rgba(21,193,93,0.2)` → `#15C15D33`).

If `get_design_context` returns colors as RGB 0–1 floats, convert: `Math.round(val * 255).toString(16).padStart(2,'0')`.

---

## Step 2 — Discover Repo Structure

Use `gh api` to locate the relevant files in the target repo.

```bash
# List src/ contents
gh api repos/{owner}/{repo}/contents/src --jq '.[].name'
```

**Files to find and update (in priority order):**

1. **CSS variables file** — `src/index.css`, `src/globals.css`, `src/app.css`, or `app/globals.css`
   Contains `--primary`, `--radius`, `--neon-glow`, etc. as HSL or hex values.

2. **Tailwind config** — `tailwind.config.ts` or `tailwind.config.js`
   May define colors in `theme.extend.colors` or custom keyframes/animations.

3. **Component source** — `src/components/ui/button.tsx` (or equivalent)
   Update only if structural token values changed (border-radius class, border-width class, font class).

---

## Step 3 — Map Figma Values to Code

Convert extracted Figma token values to the format used in each file.

### CSS variable file

Figma fills are hex. CSS variables in Tailwind projects are typically HSL (no `hsl()` wrapper):
```css
--primary: 145 80% 42%;   /* NOT #15C15D, NOT hsl(145 80% 42%) */
```

Convert hex → HSL before writing. Standard formula:
```
hex #15C15D → rgb(21, 193, 93) → hsl(145, 80%, 42%)
Write as: --primary: 145 80% 42%;
```

HSL conversion steps:
1. Normalize RGB to 0–1
2. max = Math.max(r,g,b), min = Math.min(r,g,b), delta = max - min
3. L = (max + min) / 2
4. S = delta === 0 ? 0 : delta / (1 - Math.abs(2L - 1))
5. H: depends on which channel is max (standard formula)
6. Output as `H S% L%` (space-separated, percent signs on S and L)

### Tailwind config

Only update `theme.extend.colors` values if custom color tokens are defined there (e.g. `neon.glow`, `glitch.1`). Do not touch keyframes unless animation timings or colors changed.

### Component source

Update only Tailwind utility classes that map to changed token slots:

| Changed token | Class to update |
|---|---|
| border_radius (0 → 4px) | `rounded-none` → `rounded` |
| border_radius (0 → 8px) | `rounded-none` → `rounded-md` |
| border_width (2px → 1px) | `border-2` → `border` |
| font_weight (900 → 500) | `font-black` → `font-medium` |
| text transform removed | remove `uppercase` |

If the border_radius, font, and border_width are unchanged — skip this file entirely.

---

## Step 4 — Push to GitHub

Create a branch, push each changed file, open a PR.

```bash
# 1. Get main branch SHA
MAIN_SHA=$(gh api repos/{owner}/{repo}/git/ref/heads/main --jq '.object.sha')

# 2. Create a new branch
BRANCH="ds-sync-$(date +%Y-%m-%d)"
gh api repos/{owner}/{repo}/git/refs \
  -X POST \
  -f ref="refs/heads/${BRANCH}" \
  -f sha="${MAIN_SHA}"

# 3. For each file to update — get current SHA, then PUT new content
FILE_SHA=$(gh api repos/{owner}/{repo}/contents/{path} --jq '.sha')
ENCODED=$(echo -n "${NEW_CONTENT}" | base64)

gh api repos/{owner}/{repo}/contents/{path} \
  -X PUT \
  -f message="ds: sync {component} tokens from Figma polish" \
  -f content="${ENCODED}" \
  -f sha="${FILE_SHA}" \
  -f branch="${BRANCH}"

# 4. Open PR
gh pr create \
  --base main \
  --head "${BRANCH}" \
  --title "ds: sync {component} tokens from Figma" \
  --body "$(cat <<'EOF'
## Summary
Tokens updated to match DS-polished Figma component set.

| Token | Before | After |
|---|---|---|
| primary | ... | ... |
| hover bg | ... | ... |
| pressed bg | ... | ... |

Figma node: {node_url}
Generated by ds-push on {date}
EOF
)"
```

**Important:** `base64` encoding on macOS requires `base64` (no `-w0` flag). On Linux use `base64 -w0`.
**Important:** File content must be base64-encoded with no trailing newline issues — use `echo -n` or pipe through `printf '%s'`.

---

## Step 5 — Report

```
Pushed.

Files updated:
  • src/index.css — --primary, --neon-glow updated
  • tailwind.config.ts — glitch.1, glitch.2 updated

PR: https://github.com/owner/repo/pull/42

Lovable will deploy automatically once the PR is merged.
```

If no values changed between the polished Figma component and the session token-map:
```
No changes detected. The polished component matches the generated values — no push needed.
```

---

## Working Directory

Save updated token map after push:
- `working/{component}-{date}/token-map.json` — overwrite with final pushed values
- `working/{component}-{date}/push-summary.json` — before/after diff, PR URL, files touched

---

## Error Recovery

**`SHA mismatch` on PUT:** Another commit updated the file between your read and write. Re-fetch the SHA and retry.

**`base64` encoding issues:** Use `printf '%s' "${CONTENT}" | base64` to avoid trailing newline artifacts.

**File not found in repo:** The target file may be at a different path. Re-run the Step 2 discovery and look one level deeper.

**PR already exists for branch:** The branch already has an open PR from a previous run. Use `gh pr view` to find it and update instead of creating.
