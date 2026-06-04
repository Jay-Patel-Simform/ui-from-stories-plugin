---
name: ui-from-stories
description: Generate React UI components from user stories. Use when asked to "generate UI", "build from user story", "create component from story", "implement UI for", or "scaffold UI". Produces components using whatever UI library the project already uses (shadcn, MUI, Chakra, Ant Design, plain Tailwind, etc.).
---

# UI from User Stories

Generate production-ready React UI from plain-text user stories. Strictly UI only — no backend, no API integration, no routing changes unless the story explicitly requires a new route.

## Step 1 — Read project context

Before generating anything, read `CLAUDE.md` to understand folder structure, import aliases, and conventions.

Then detect what UI library the project uses:
```bash
cat package.json | grep -E '"@shadcn|radix-ui|@mui|@chakra-ui|antd|@mantine|@headlessui|daisyui'
ls app/components/ui/ 2>/dev/null || ls src/components/ui/ 2>/dev/null || ls components/ 2>/dev/null || echo "no ui dir"
ls app/components/shared/ 2>/dev/null || ls src/components/shared/ 2>/dev/null || echo "no shared dir"
```

**Use whatever UI library is already installed.** If no library is present, use semantic HTML + Tailwind utility classes only. Never import a component that does not already exist in the project.

## Step 2 — Parse the user stories

Extract from each story:
- **Actor** — who does the action (user, admin, guest)
- **Action verbs** — determines which pattern to use (see Pattern Matrix)
- **UI elements** — what the user sees or interacts with
- **Constraints** — validation rules, conditional visibility, empty states

Group related stories into one feature if they share the same data entity.

## Step 3 — Select React pattern (auto-decide)

Use this matrix. Pick the FIRST pattern that fits:

| Story signals | Pattern |
|---|---|
| "view/see/display" only, no user input | **Presentation** |
| "filter/search/sort" using URL state | **Presentation + searchParams** |
| "submit/create/edit/delete" with a form | **Presentation + Container** with Zod schema |
| Multiple related sub-parts sharing state (tabs, wizard, accordion) | **Compound Pattern** |
| Same behavior reused across 2+ unrelated components | **HOC** |
| Complex page mixing data fetching and actions | **Container/Presentation split** |

**Default when unsure:** Container/Presentation split.

## Step 4 — Map story elements to components

Use semantic HTML5 elements (`<nav>`, `<main>`, `<section>`, `<article>`, `<header>`, `<footer>`, `<aside>`, `<ul>`, `<li>`, `<form>`, `<fieldset>`, `<figure>`) as the structural backbone. Layer the project's existing UI library on top for interaction and visual consistency.

| Story element | Semantic element | Use project's component if it exists, else fallback |
|---|---|---|
| List / data table | `<table>` / `<ul><li>` | Table component or `<table>` with Tailwind |
| Create or edit form | `<form>` + `<fieldset>` | Card + Input + Button components or plain HTML |
| Destructive confirmation | `<dialog>` (managed) | Modal/Dialog component or `<dialog>` |
| Contextual hint | `<span>` + `aria-describedby` | Tooltip component or `title` attr |
| Slide-in edit panel | `<aside>` (managed) | Drawer/Sheet component or positioned `<aside>` |
| Single-value dropdown | `<select>` (managed) | Select component or native `<select>` |
| Multi-line text | `<textarea>` | Textarea component or native `<textarea>` |
| Loading placeholder | `<div role="status">` | Skeleton component or CSS animation |
| Input with prefix/suffix | `<label>` + `<input>` group | InputGroup component or flex wrapper |
| Navigation tabs | `<nav>` + `<ul><li>` | Tabs component or compound pattern |
| Image with caption | `<figure><img><figcaption>` | — |
| Status badge | `<span>` | Badge component or Tailwind utility classes |

**If a needed component is not in the project's component directories, use semantic HTML + Tailwind — do not fabricate an import.**

## Step 5 — Generate files

Follow the folder structure declared in `CLAUDE.md` exactly.

Every generated component must include:

1. **Loading state** — spinner or skeleton while async data loads (use project's loading component if one exists)
2. **Empty state** — explicit UI when a list has no items
3. **Error state** — visible message on failure, no silent crash
4. **Responsive layout** — see Responsiveness rules below
5. **Accessible markup** — see Accessibility rules below

## Responsiveness

Every component must work on mobile, tablet, and desktop.

Rules:
- Default styles target mobile (no prefix)
- Use `sm:` (640px), `md:` (768px), `lg:` (1024px), `xl:` (1280px) to progressively enhance
- Stacked on mobile → side-by-side on `md:` for form fields and card grids
- Tables: on mobile use `overflow-x-auto` wrapper or switch to card-per-row layout
- Font sizes, spacing, and padding must scale — avoid fixed px widths on containers
- Touch targets must be at least 44×44px (`min-h-11 min-w-11`)
- Never hide critical content with `hidden` on mobile without an accessible alternative

## Accessibility

Every component must meet WCAG 2.1 AA.

Rules:
- All `<input>`, `<select>`, `<textarea>` must have a visible `<label>` with matching `htmlFor` ↔ `id`
- Icon-only buttons must have `aria-label` describing the action
- Images must have `alt` text; decorative images use `alt=""`
- Table headers use `<th scope="col">` or `<th scope="row">`
- Modal/Dialog/Drawer components must trap focus and restore it on close — verify the chosen library handles this, otherwise implement manually with `focus-trap`
- Error messages must use `role="alert"` or be linked to the field via `aria-describedby`
- Interactive elements must be reachable and operable via keyboard
- Use semantic landmarks: `<main>`, `<nav>`, `<section aria-label="...">`, `<aside>`
- Color must not be the only way to convey meaning (pair with icon or text)
- Avoid `tabIndex` values other than `0` and `-1`

## What NOT to do

- Do not create API functions, query hooks, or `queryClient` calls — UI only
- Do not add routes unless the story explicitly says "navigate to" or "new page"
- Do not import components that don't exist in the project's component directories
- Do not assume shadcn, MUI, or any specific library is available — always detect first
- Do not use `<div>` where a semantic element fits
- Do not use `useState` for filter/search/pagination — those belong in URL `searchParams`
