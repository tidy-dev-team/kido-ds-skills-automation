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
- **GitHub repo URL** — the target codebase
- **Component file path** (optional) — if not provided, the skill will discover it

---

## Step 1 — Discover Storybook Setup

Check the repo for Storybook presence:

```bash
# Check package.json for @storybook/* deps
gh api repos/{owner}/{repo}/contents/package.json --jq '.content' | base64 -d | \
  python3 -c "import sys,json; d=json.load(sys.stdin); deps={**d.get('dependencies',{}),**d.get('devDependencies',{})}; print('\n'.join(k for k in deps if 'storybook' in k.lower()))"

# Check for .storybook config dir
gh api repos/{owner}/{repo}/contents/.storybook --jq '.[].name' 2>/dev/null
```

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

Fetch the component source file from the repo:

```bash
gh api repos/{owner}/{repo}/contents/{path/to/component} --jq '.content' | base64 -d
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

## Step 6 — Push to GitHub

Create a branch and open a PR:

```bash
# Create branch
BRANCH="ds/storybook-{component}-{date}"
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

## Step 7 — Report

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

⚠️  Storybook not installed. Setup instructions included in PR body.
```

---

## Working Directory

Save generated story source to `working/{component}-{YYYY-MM-DD}/storybook.tsx` for reference.

---

## Notes on Framework Detection

This skill targets React + TypeScript (CSF3). If the repo uses a different framework:
- **Vue:** Use `Meta` from `@storybook/vue3`, template-style stories
- **Svelte:** Use `Meta` from `@storybook/svelte`
- **Non-TypeScript:** Drop type annotations, use `satisfies` or JSDoc

Detect framework from `package.json` dependencies before generating.
