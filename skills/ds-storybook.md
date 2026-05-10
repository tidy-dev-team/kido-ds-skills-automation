---
name: ds-storybook
description: >
  Generate a production-ready Storybook story file for a polished component, based on the
  Kido DS spec (variant axes) and the component's actual React prop interface.
  Triggers on phrases like "create storybook stories", "add stories", "generate storybook",
  "document this component in storybook", "add component to storybook", or after a ds-push
  when the designer asks to document the result.
---

# DS Storybook

**Spec + component source in. Story file on GitHub out.**

Read the Kido DS spec (variant axes) → read the React component interface → map axes to args →
generate CSF3 stories → push to GitHub.

---

## What This Skill Does (and Does Not Do)

**Does:** Generate a `{component}.stories.tsx` file with proper `argTypes`, named stories per
meaningful variant, pseudo-state stories, and a setup note if Storybook isn't installed yet.

**Does not:** Run `npm install`, configure Storybook from scratch, or rewrite the component.
If Storybook is missing, the skill pushes the story file and PRs the setup instructions.

---

## Inputs Required

- **Component name** — e.g. "button", "toggle"
- **Source** — where the component code lives (read):
  - **GitHub repo URL** (typical for the Hybrid mode below — the Lovable repo where `/ds-push` already landed the polished code)
  - **Local project path** (Local mode)
- **Destination** — where the story should land (write):
  - Same GitHub repo (Remote mode → PR)
  - Same local project (Local mode → direct write)
  - **A different local Storybook folder** (Hybrid mode → direct write into a separate Storybook instance)
- **Component file path** (optional) — if not provided, the skill will discover it (in the repo via `gh api`, or on disk via `find`/`glob`).
- **Storybook URL** (optional) — address of a running Storybook instance used to verify stories render. Defaults to `http://localhost:6006`. Accept any of: `localhost:6006`, `http://localhost:6006`, `https://storybook.example.com`. Normalize to a full URL with scheme before use.
- **Figma component URL** (optional, Hybrid mode) — used as additional context for variant axes, polish notes, and visual reference screenshots when generating the story.

### Picking the right mode

| Inputs | Mode |
|---|---|
| GitHub repo URL only | **Remote** — read & write the same repo, deliver as PR |
| Local project path only | **Local** — read & write the same on-disk project |
| GitHub repo URL + a separate local folder for Storybook (optionally + a Figma URL) | **Hybrid** — read from repo, write component + story + tokens into the separate local Storybook folder |
| Only a Storybook URL with no source or destination | Ask which repo/path the source code is in and which folder hosts the Storybook instance |

The three modes share generation logic and verification — only the read/write surface differs. **Hybrid mode** is the right choice when `/ds-push` has already updated a Lovable (or similar) repo and the designer wants to test the component in a separate, locally-running Storybook sandbox without forking the Lovable project.

**Intake gate.** Before any mode resolution, confirm the skill has at least: a **component name** and a **source** (GitHub repo URL or local project path). If either is missing (e.g. invoked with empty args via `/ds-guide`), ask the designer for the missing piece(s). Destination defaults to "same as source" unless the designer mentions a separate Storybook folder. Storybook URL and Figma URL remain optional; do not block on them.

---

## Step 0 — Resolve the Storybook URL

If the designer provides a Storybook URL, use it. Otherwise default to `http://localhost:6006`.

Normalize the input:
- `localhost:6006` → `http://localhost:6006`
- `6006` → `http://localhost:6006`
- `storybook.example.com` → `https://storybook.example.com`
- Strip any trailing slash.

Probe the URL to confirm it is reachable before relying on it for verification:

```bash
STORYBOOK_URL="${STORYBOOK_URL:-http://localhost:6006}"
STORYBOOK_URL="${STORYBOOK_URL%/}"
curl -fsS -o /dev/null -m 3 "${STORYBOOK_URL}/index.json" \
  && echo "reachable" \
  || echo "unreachable — verification will be skipped"
```

If unreachable: continue with generation and push, but skip the post-push verification step (Step 7a) and note it in the final report. Do **not** prompt the designer — Storybook is often run on demand.

If reachable, fetch the existing story index to see whether a story with the same `title` already exists:

```bash
curl -fsS "${STORYBOOK_URL}/index.json" | \
  python3 -c "import sys,json; d=json.load(sys.stdin); [print(v['title']) for v in d.get('entries',{}).values() if v.get('type')=='docs' or v.get('type')=='story']" | sort -u
```

If `Components/{ComponentName}` already exists, warn that this PR will overwrite or duplicate it depending on filename collision — proceed but flag in the PR body.

---

## Step 1 — Discover Storybook Setup

Check the project for Storybook presence.

**Remote (GitHub repo URL):**
```bash
# Check package.json for @storybook/* deps
gh api repos/{owner}/{repo}/contents/package.json --jq '.content' | base64 -d | \
  python3 -c "import sys,json; d=json.load(sys.stdin); deps={**d.get('dependencies',{}),**d.get('devDependencies',{})}; print('\n'.join(k for k in deps if 'storybook' in k.lower()))"

# Check for .storybook config dir
gh api repos/{owner}/{repo}/contents/.storybook --jq '.[].name' 2>/dev/null
```

**Local (filesystem path):**
```bash
PROJECT="${LOCAL_PROJECT_PATH/#\~/$HOME}"   # expand ~

# Storybook deps
python3 -c "import sys,json; d=json.load(open('${PROJECT}/package.json')); deps={**d.get('dependencies',{}),**d.get('devDependencies',{})}; print('\n'.join(k for k in deps if 'storybook' in k.lower()))"

# .storybook config dir
ls "${PROJECT}/.storybook" 2>/dev/null

# Existing stories directory convention (so the new file lands in the right place)
find "${PROJECT}/src" -maxdepth 4 -name "*.stories.*" 2>/dev/null | head -5
```

Use the `Read`, `Bash` (with `find`/`ls`), and `Write` tools for local mode — never `gh api`.

**Three scenarios:**

| Scenario | Action |
|---|---|
| Storybook installed + `.storybook/` config present | Generate story file only |
| Storybook installed but no `.storybook/` | Generate story file + note that config is needed |
| No Storybook at all | Generate story file + include setup instructions in PR body |

**Also check for pseudo-states addon:**
```bash
# Presence of storybook-addon-pseudo-states enables :hover/:focus stories
grep -i "pseudo-states" package.json
```

---

## Step 2 — Read Component Interface

**Remote:**
```bash
gh api repos/{owner}/{repo}/contents/{path/to/component} --jq '.content' | base64 -d
```

**Local:** use the `Read` tool on the absolute path (e.g. `${PROJECT}/src/components/ui/button.tsx`). If the component path was not provided, locate it first:
```bash
find "${PROJECT}/src" -type f \( -name "{Component}.tsx" -o -name "{component}.tsx" -o -name "{Component}.jsx" -o -name "{component}.jsx" \) 2>/dev/null
```

From the source, extract:
- **Exported props interface** — `interface ButtonProps extends ...`
- **Variant options** — from `cva(...)` variant definitions or union type literals
- **Size options** — from size variant definitions
- **Boolean props** — `disabled`, `loading`, `checked`, etc.
- **Internal props to hide** — `asChild`, `ref`, `className` — suppress in docs

If the component uses `cva` (class-variance-authority), the variant options are defined inline:
```ts
variant: {
  crazy: "...",
  outline: "...",
  ghost: "...",
}
```
→ extract `['crazy', 'outline', 'ghost', 'default', ...]`

---

## Step 3 — Read the Kido DS Spec

Load `specs/_index.json` → find the matching spec → load `specs/{component}.spec.json`.

Extract the variant axes:

```json
"variants": {
  "type":    { "values": ["contained", "outlined", "ghost"] },
  "size":    { "values": ["s", "m", "l"] },
  "state":   { "values": ["idle", "hover", "pressed", "focused", "disabled", "loading"] },
  "inverse": { "values": ["no", "yes"] }
}
```

---

## Step 4 — Map Axes to Story Structure

Figma variant axes don't map 1:1 to React props. Use this translation table:

| Figma axis | React prop | Storybook approach |
|---|---|---|
| `type` | `variant` prop (select) | `argTypes.variant` with `control: 'select'` |
| `size` | `size` prop (select) | `argTypes.size` with `control: 'select'` |
| `state=idle` | (default) | Implicit — the default story |
| `state=disabled` | `disabled: true` | Explicit `Disabled` story |
| `state=loading` | `loading` boolean or slot | `Loading` story if prop exists; comment if CSS-only |
| `state=hover` | none (CSS only) | `parameters: { pseudo: { hover: true } }` if addon present; else `play` function using `userEvent.hover` |
| `state=pressed` | none (CSS only) | `parameters: { pseudo: { active: true } }` if addon present |
| `state=focused` | none (CSS only) | `parameters: { pseudo: { focus: true } }` if addon present |
| `inverse=yes` | none (CSS only) | Dark mode decorator: `decorators: [darkModeDecorator]` |

**Named stories to generate (for Button):**

One story per "type" axis value + key size variants + all representable states:

```
Primary        → variant's main/contained equivalent (e.g. 'crazy')
Outline        → variant='outline'
Ghost          → variant='ghost'
SizeSm         → size='sm'
SizeLg         → size='lg'
Disabled       → disabled=true
Hovered        → pseudo: { hover: true }  (if addon) or play function
Focused        → pseudo: { focus: true }  (if addon)
AllVariants    → render table of all variants
```

Do not generate 54 individual stories (one per Figma variant). Generate meaningful named stories — 8–12 is the right range.

---

## Step 5 — Generate the Story File

**File extension rule:** Always use `.jsx` or `.tsx` — never `.js` or `.ts` — for story files that contain JSX. Vite (including the `vite:oxc` plugin in Storybook 10+) will not parse JSX in `.js` files and throws `Unexpected token` at runtime.

- TypeScript repo → `{Component}.stories.tsx`
- JavaScript repo → `{Component}.stories.jsx`

Same rule applies to the component file itself (`Button.jsx`, not `Button.js`).

Use CSF3 format (Storybook 7+). Template:

```tsx
import type { Meta, StoryObj } from '@storybook/react';
import { {ComponentName} } from './{component}';

const meta: Meta<typeof {ComponentName}> = {
  title: 'Components/{ComponentName}',
  component: {ComponentName},
  tags: ['autodocs'],
  parameters: {
    layout: 'centered',
  },
  argTypes: {
    // One entry per controllable prop
    variant: {
      control: 'select',
      options: [{variant_options}],
      description: '{variant_description}',
    },
    size: {
      control: 'select',
      options: [{size_options}],
    },
    disabled: { control: 'boolean' },
    children: { control: 'text' },
    // Internal props: suppress from docs panel
    asChild: { table: { disable: true } },
  },
};
export default meta;
type Story = StoryObj<typeof {ComponentName}>;

export const {TypeName}: Story = {
  args: { children: '{label}', variant: '{default_variant}' },
};

// ... one export per named story
```

### Pseudo-states: addon present vs absent

**If `storybook-addon-pseudo-states` is installed:**
```tsx
export const Hovered: Story = {
  args: { children: 'Button', variant: 'crazy' },
  parameters: { pseudo: { hover: true } },
};
```

**If only `@storybook/addon-interactions` is available:**
```tsx
import { userEvent, within } from '@storybook/testing-library';

export const Hovered: Story = {
  args: { children: 'Button', variant: 'crazy' },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    await userEvent.hover(canvas.getByRole('button'));
  },
};
```

**If no interaction addon at all:**
```tsx
// Hover state is CSS-only — test with browser DevTools:
// Elements panel → :hov → check :hover
export const Hovered: Story = {
  args: { children: 'Button', variant: 'crazy' },
  // Manually trigger: F12 → Elements → Force element state → :hover
};
```

### AllVariants story

Generate a render function that shows all variant × size combinations as a grid:

```tsx
export const AllVariants: Story = {
  render: () => (
    <div style={{ display: 'flex', flexDirection: 'column', gap: 16 }}>
      {(['crazy', 'outline', 'ghost'] as const).map(variant => (
        <div key={variant} style={{ display: 'flex', gap: 8, alignItems: 'center' }}>
          {(['sm', 'default', 'lg'] as const).map(size => (
            <Button key={size} variant={variant} size={size}>
              {variant} {size}
            </Button>
          ))}
        </div>
      ))}
    </div>
  ),
};
```

---

## Step 5a — Hybrid Mode: copy component, tokens, and immediate deps

Run this step **only in Hybrid mode** (source = GitHub repo, destination = a separate local Storybook folder). Skip in pure Remote or Local mode.

The goal: make the just-pushed component renderable inside a separate Storybook sandbox by mirroring the source repo's component file, its immediate imports, and the relevant CSS-variable / Tailwind token state. The designer is using this to *test recent tokens* — never substitute or simplify values, copy verbatim.

Set the variables once:
```bash
SRC_OWNER=...; SRC_REPO=...; SRC_REF="main"   # default branch is fine
SBK="${LOCAL_STORYBOOK_PATH/#\~/$HOME}"        # destination folder, expanded
```

### 5a.1 — Copy the component file

Read `{path/to/component}` from the source repo and write it to a parallel path under `${SBK}`. Default destination is `${SBK}/src/components/ui/{component}.tsx`; if the Storybook folder already uses a different convention (discovered in Step 1), match it.

```bash
gh api repos/${SRC_OWNER}/${SRC_REPO}/contents/{path}?ref=${SRC_REF} --jq '.content' | base64 -d
```
→ pipe content into the `Write` tool at the destination. If the file already exists, `Read` it and confirm overwrite before writing.

### 5a.2 — Copy CSS variables (always, full)

The designer wants recent tokens — copy the source's CSS-variable file in full, do not diff or selectively merge.

1. Locate the source repo's globals file. Try in order: `src/index.css`, `src/globals.css`, `src/app.css`, `app/globals.css`.
2. Read it via `gh api` and write it to the matching path inside `${SBK}`.
3. If the Storybook folder's globals file already exists at a different path (e.g. `${SBK}/.storybook/preview.css`), append the source file's `:root { ... }` and `.dark { ... }` blocks to it instead of replacing — leave the rest of the Storybook folder's existing styles intact.

Report which file was overwritten or appended to.

### 5a.3 — Copy Tailwind config keys

If the source uses Tailwind:
1. Read `tailwind.config.ts` (or `.js`) from the source repo.
2. Read `${SBK}/tailwind.config.ts` (or `.js`).
3. **Merge** `theme.extend.colors`, `theme.extend.borderRadius`, `theme.extend.fontFamily`, `theme.extend.boxShadow`, `theme.extend.keyframes`, and `theme.extend.animation` from source → destination. Source wins on key conflict.
4. Do not touch `content`, `plugins`, or top-level config in the destination — those are Storybook-folder-specific.
5. Write the merged result back via the `Edit` tool with surgical replacements per key.

If the destination has no Tailwind config, copy the source's wholesale and warn that the Storybook folder may need `tailwindcss` installed.

### 5a.4 — Detect and confirm immediate dependencies

The component likely imports from `@/components/ui/*`, `@/lib/utils`, or similar. These won't exist in the Storybook folder yet.

1. Parse `import` statements from the component source.
2. Resolve each `@/...` import against the source repo's `tsconfig.json` `paths` (typically `@/* → src/*`).
3. For each resolved path, check if the file already exists at the equivalent location in `${SBK}`.
4. **List the missing ones** and ask the designer:
   ```
   Component button.tsx imports 3 files that aren't in your Storybook folder:
     - @/lib/utils (cn helper)
     - @/components/ui/slot
     - @/components/ui/icon

   Copy these from the source repo? [y/n/list]
   ```
5. On `y`: copy each missing file verbatim (one level deep — do **not** recurse). On `list`: let the designer pick a subset. On `n`: continue and warn that the component will fail to compile in the Storybook folder.

Do not auto-recurse. If a copied dep itself has missing imports, surface them in the final report so the designer can decide on a second pass.

### 5a.5 — Optional Figma context

If a Figma component URL was provided, call `get_design_context` and `get_screenshot` once. Use the result to:
- Cross-check that the variant axes extracted from the React component match the Figma variant axes (warn on mismatch — don't fail).
- Save the screenshot to `working/{component}-{date}/figma-reference.png` for the designer to compare against rendered Storybook output.

The Figma input is reference-only — never used to override values read from the source repo.

---

## Step 6 — Deliver the Story File

### Local & Hybrid modes (filesystem path)

Write the file directly using the `Write` tool. No branch, no PR.

1. Pick the destination path. Prefer co-locating with the component (e.g. `${SBK_OR_PROJECT}/src/components/ui/button.stories.tsx`). Fall back to `${SBK_OR_PROJECT}/src/stories/` only if that's the existing convention discovered in Step 1.
2. If the file already exists, `Read` it first and confirm with the designer before overwriting.
3. Use `.tsx` for TypeScript projects, `.jsx` for JavaScript.
4. After writing, report the absolute path. The designer's running Storybook (HMR) will pick it up automatically — no rebuild needed.

In Hybrid mode, the component file (and any deps the designer approved) was already written in Step 5a — only the story file is new here.

Skip the GitHub commands entirely in Local and Hybrid modes. Skip the "PR body when Storybook is not installed" subsection — instead, print the setup instructions inline in the final report.

### Remote mode (GitHub repo URL)

Create a branch and open a PR:

```bash
# Create branch (canonical naming: ds/{action}-{component}-{date})
BRANCH="ds/storybook-{component}-$(date +%Y-%m-%d)"
MAIN_SHA=$(gh api repos/{owner}/{repo}/git/ref/heads/main --jq '.object.sha')
gh api repos/{owner}/{repo}/git/refs -X POST -f ref="refs/heads/${BRANCH}" -f sha="${MAIN_SHA}"

# Push story file — use .tsx for TypeScript repos, .jsx for JavaScript repos
FILE_PATH="src/stories/{component}.stories.tsx"  # adjust path and extension to match repo
ENCODED=$(printf '%s' "${STORY_CONTENT}" | base64)
gh api repos/{owner}/{repo}/contents/${FILE_PATH} \
  -X PUT \
  -f message="ds: add {component} storybook stories [ds-storybook]" \
  -f content="${ENCODED}" \
  -f branch="${BRANCH}"

# Open PR
gh pr create --base main --head "${BRANCH}" \
  --title "ds: add {component} stories" \
  --body "..."
```

### PR body when Storybook is not installed

If no Storybook was detected, include setup instructions in the PR body:

```markdown
## Setup required

Storybook is not yet installed in this repo. To activate these stories:

\`\`\`bash
npx storybook@latest init
npm run storybook
\`\`\`

For pseudo-state stories, also install:
\`\`\`bash
npm install -D storybook-addon-pseudo-states
\`\`\`
Then add `'storybook-addon-pseudo-states'` to the `addons` array in `.storybook/main.ts`.

The story file is valid now — it will work once Storybook is configured.
```

---

## Step 7a — Verify Against the Running Storybook (if reachable)

After the PR is opened, if the Storybook URL probed reachable in Step 0 **and** the designer is running it against a local checkout that already includes the new file (e.g. they pulled the branch), verify each generated story renders without error.

```bash
# For each story id (kebab-case of title + name), hit the iframe endpoint
STORY_ID="components-button--crazy"
curl -fsS -o /dev/null -w "%{http_code}\n" \
  "${STORYBOOK_URL}/iframe.html?id=${STORY_ID}&viewMode=story"
```

For richer verification, use the `chrome-devtools` MCP tools to navigate to each story URL and check the console for errors:

1. `mcp__chrome-devtools__new_page` → `${STORYBOOK_URL}/iframe.html?id=${STORY_ID}&viewMode=story`
2. `mcp__chrome-devtools__list_console_messages` — fail the story if any `error`-level message appears
3. `mcp__chrome-devtools__take_screenshot` — attach screenshots to the working dir for the designer

Skip verification entirely if the URL was unreachable in Step 0 — do not block on it.

---

## Step 7 — Report

**Remote mode:**
```
Button stories pushed to github.com/{owner}/{repo}.

Stories generated (9):
  Crazy           → variant='crazy', default state
  Outline         → variant='outline'
  Ghost           → variant='ghost'
  SizeSm          → variant='crazy', size='sm'
  SizeLg          → variant='crazy', size='lg'
  Disabled        → disabled=true
  Hovered         → pseudo: hover (note: addon not installed)
  Focused         → pseudo: focus (note: addon not installed)
  AllVariants     → render grid, all variant × size

PR: https://github.com/{owner}/{repo}/pull/N

Storybook URL: http://localhost:6006 (reachable — verified 9/9 stories render)

⚠️  Storybook not installed. Setup instructions included in PR body.
```

If the Storybook URL was unreachable, replace the verification line with:
```
Storybook URL: http://localhost:6006 (unreachable — verification skipped)
```

**Local mode:**
```
Button stories written to /Users/me/code/acme-ui/src/components/ui/button.stories.tsx.

Stories generated (9):
  Crazy, Outline, Ghost, SizeSm, SizeLg, Disabled, Hovered, Focused, AllVariants

Storybook URL: http://localhost:6006 (reachable — HMR will pick up the new file)
```

**Hybrid mode:**
```
Source: github.com/acme/payzo-lovable @ main
Storybook folder: /Users/me/code/ds-sandbox

Files written:
  • src/components/ui/button.tsx              (copied from source)
  • src/index.css                             (overwritten — fresh tokens)
  • tailwind.config.ts                        (theme.extend keys merged)
  • src/components/ui/button.stories.tsx      (generated)

Dependencies copied (designer-approved):
  • src/lib/utils.ts
  • src/components/ui/slot.tsx

Stories generated (9):
  Crazy, Outline, Ghost, SizeSm, SizeLg, Disabled, Hovered, Focused, AllVariants

Storybook URL: http://localhost:6006 (reachable — HMR will pick up the new files)

Figma reference: working/button-2026-04-30/figma-reference.png
```

---

## Working Directory

Save generated story source for reference. Use the same working directory the upstream skill used:
- **Workflow A:** `working/{component}-{YYYY-MM-DD}/storybook.tsx`
- **Workflow B:** `working/{project}/storybook.tsx`

See `LANGUAGE.md` for canonical paths and branch-naming conventions.

---

## Notes on Framework Detection

This skill targets React + TypeScript (CSF3). If the repo uses a different framework:
- **Vue:** Use `Meta` from `@storybook/vue3`, template-style stories
- **Svelte:** Use `Meta` from `@storybook/svelte`
- **Non-TypeScript:** Drop type annotations, use `satisfies` or JSDoc

Detect framework from `package.json` dependencies before generating.
