---
name: ui-from-stories
description: Generate React UI components from user stories. Use when asked to "generate UI", "build from user story", "create component from story", "implement UI for", or "scaffold UI". Produces shadcn/ui + Tailwind components following best React patterns.
---

# UI from User Stories

Generate production-ready React UI from plain-text user stories. Strictly UI only — no backend, no API integration, no routing changes unless the story explicitly requires a new route.

## Step 1 — Read project context

Before generating anything, read `CLAUDE.md` to understand folder structure, import aliases, and conventions.

Then run:
```bash
ls app/components/ui/
ls app/components/shared/ 2>/dev/null || echo "no shared dir"
```

**Only import components that already exist in these directories.** Never invent or assume a component exists.

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

Use semantic HTML5 elements (`<nav>`, `<main>`, `<section>`, `<article>`, `<header>`, `<footer>`, `<aside>`, `<ul>`, `<li>`, `<form>`, `<fieldset>`, `<figure>`) as the structural backbone. Layer shadcn/ui on top for interaction and visual consistency.

| Story element | Semantic element | shadcn/ui component |
|---|---|---|
| List / data table | `<table>` / `<ul><li>` | `Table` |
| Create or edit form | `<form>` + `<fieldset>` | `Card` + `Field`, `Input`, `Button` |
| Destructive confirmation | `<dialog>` (managed) | `Dialog` |
| Contextual hint | `<span>` + `aria-describedby` | `Tooltip` |
| Slide-in edit panel | `<aside>` (managed) | `Sheet` |
| Single-value dropdown | `<select>` (managed) | `Select` |
| Multi-line text | `<textarea>` | `Textarea` |
| Loading placeholder | `<div role="status">` | `Skeleton` |
| Input with prefix/suffix | `<label>` + `<input>` group | `InputGroup`, `InputGroupAddon`, `InputGroupText` |
| Navigation tabs | `<nav>` + `<ul><li>` | Compound pattern or Radix Tabs |
| Image with caption | `<figure><img><figcaption>` | — |
| Status badge | `<span>` | Tailwind utility classes |

**If a needed component is not in `app/components/ui/` or `app/components/shared/`, use semantic HTML + Tailwind — do not fabricate an import.**

## Step 5 — Generate files

Follow the folder structure declared in `CLAUDE.md` exactly.

Every generated component must include:

1. **Loading state** — `Skeleton` while async data loads
2. **Empty state** — explicit UI when a list has no items (`<p className="text-muted-foreground">No items yet.</p>`)
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
- `<Dialog>` and `<Sheet>` must trap focus and restore it on close (shadcn handles this — do not override)
- Error messages must use `role="alert"` or be linked to the field via `aria-describedby`
- Interactive elements must be reachable and operable via keyboard
- Use semantic landmarks: `<main>`, `<nav>`, `<section aria-label="...">`, `<aside>`
- Color must not be the only way to convey meaning (pair with icon or text)
- Avoid `tabIndex` values other than `0` and `-1`

## What NOT to do

- Do not create API functions, query hooks, or `queryClient` calls — UI only
- Do not add routes unless the story explicitly says "navigate to" or "new page"
- Do not import components that don't exist in `app/components/ui/` or `app/components/shared/`
- Do not use `<div>` where a semantic element fits
- Do not use `useState` for filter/search/pagination — those belong in URL `searchParams`
