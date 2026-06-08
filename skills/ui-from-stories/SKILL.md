---
name: ui-from-stories
description: >
  Generate production-ready frontend UI from user stories and acceptance criteria.
  Use this skill whenever the user provides a user story, feature spec, acceptance criteria,
  or any requirement description and wants frontend code — even if they phrase it as "build this",
  "implement the screen", "scaffold the feature", "create the form/table/dashboard/CRUD interface",
  or simply paste a story without instructions. Covers React, Next.js, Vue, Nuxt, Angular, and
  Svelte across any component library (Ant Design, MUI, Mantine, Chakra, shadcn/ui, PrimeReact,
  Bootstrap, or custom systems).
---

# UI From User Stories

Think like a Staff Frontend Engineer. Always analyse → plan → generate. Never jump straight to code.

---

## Phase 1: Project Discovery

### 1a — Read project conventions

First, read `CLAUDE.md` (if present) to pick up folder structure, import aliases, and any project-specific rules before doing anything else.

### 1b — Detect stack

Run these commands to confirm the actual stack rather than guessing:

```bash
# Detect UI library
cat package.json | grep -E '"@shadcn|radix-ui|@mui|@chakra-ui|antd|@mantine|@headlessui|daisyui|primereact"'

# Locate component directories
ls app/components/ui/ 2>/dev/null || ls src/components/ui/ 2>/dev/null || ls components/ 2>/dev/null || echo "no ui dir"
ls app/components/shared/ 2>/dev/null || ls src/components/shared/ 2>/dev/null || echo "no shared dir"

# Confirm framework and language
cat package.json | grep -E '"next|vue|nuxt|angular|svelte|react"'
ls tsconfig.json 2>/dev/null && echo "TypeScript" || echo "JavaScript"
```

Populate this table from the results:

| Dimension             | Detected                                                                         |
| --------------------- | -------------------------------------------------------------------------------- |
| **Framework**         | React / Next.js / Vue / Nuxt / Angular / Svelte                                  |
| **Language**          | TypeScript / JavaScript                                                          |
| **Styling**           | Tailwind / Styled Components / Emotion / CSS Modules / SCSS                      |
| **Component library** | Ant Design / MUI / Mantine / Chakra / shadcn/ui / PrimeReact / Bootstrap / none  |
| **Architecture**      | Atomic / Feature-based / Domain-driven / Layer-based                             |
| **Routing**           | React Router / Next App Router / Next Pages Router / Vue Router / Angular Router |

**Use whatever is already installed. Never import a component that does not exist in the project. If no library is present, use semantic HTML + Tailwind only.**

### 1c — Existing component inventory

Scan component directories and produce this table before moving to Phase 2:

| Component         | Location                        | Props / API                  | Gaps for this story                       |
| ----------------- | ------------------------------- | ---------------------------- | ----------------------------------------- |
| e.g. `DataTable`  | `src/components/DataTable.tsx`  | `columns`, `data`, `loading` | Missing `onRowClick`, no empty-state slot |
| e.g. `PageHeader` | `src/components/PageHeader.tsx` | `title`, `actions`           | None — reuse as-is                        |

**Rules:**

- Covers the need fully → **reuse as-is.** Do not recreate it.
- Partially covers the need → **reuse and list every props/behaviour gap.** Address gaps via composition or new props; do not fork the original.
- Nothing fits → create a new one following project conventions.

Carry this inventory into Phase 3 so the hierarchy reflects reuse decisions, not a greenfield design.

---

## Phase 2: Requirement Analysis

Extract from the user story:

- **Actors** — who is using this feature?
- **Goals** — what are they trying to achieve?
- **Actions** — create / edit / delete / search / filter / export / approve / reject / view
- **Entities & fields** — data model implied by the story
- **Business rules** — validations, permissions, restrictions, conditional behaviours

---

## Phase 3: UI Planning

### 3a — Select component pattern

Pick the **first** pattern that fits:

| Story signals                                                      | Pattern                                          |
| ------------------------------------------------------------------ | ------------------------------------------------ |
| "view / see / display" only, no user input                         | **Presentation**                                 |
| "filter / search / sort" using URL state                           | **Presentation + searchParams**                  |
| "submit / create / edit / delete" with a form                      | **Presentation + Container** with Zod/Yup schema |
| Multiple related sub-parts sharing state (tabs, wizard, accordion) | **Compound**                                     |
| Same behaviour reused across 2+ unrelated components               | **HOC**                                          |
| Complex page mixing data fetching and actions                      | **Container / Presentation split**               |

Default when unsure: **Container / Presentation split.**

### 3b — Map story elements to semantic HTML

Use semantic HTML5 as the structural backbone. Layer the detected component library on top for interaction and visual consistency. **Never use `<div>` where a semantic element fits.**

| Story element            | Semantic element            | Use project component if it exists, else fallback |
| ------------------------ | --------------------------- | ------------------------------------------------- |
| List / data table        | `<table>` / `<ul><li>`      | Table component or `<table>` + Tailwind           |
| Create or edit form      | `<form>` + `<fieldset>`     | Card + Input + Button or plain HTML               |
| Destructive confirmation | `<dialog>`                  | Modal/Dialog component or `<dialog>`              |
| Slide-in edit panel      | `<aside>`                   | Drawer/Sheet component or positioned `<aside>`    |
| Single-value dropdown    | `<select>`                  | Select component or native `<select>`             |
| Loading placeholder      | `<div role="status">`       | Skeleton component or CSS animation               |
| Navigation tabs          | `<nav>` + `<ul><li>`        | Tabs component or compound pattern                |
| Status badge             | `<span>`                    | Badge component or Tailwind utility classes       |
| Contextual hint          | `<span aria-describedby>`   | Tooltip component or `title` attr                 |
| Image with caption       | `<figure><img><figcaption>` | —                                                 |

### 3c — Component hierarchy

Write a tree that reflects the Phase 1 inventory. Annotate each node as `[reuse]`, `[extend]`, or `[new]`:

```
UsersPage             [new]
├── PageHeader        [reuse]
├── UserFilters       [new]
├── DataTable         [extend] — needs onRowClick + empty-state slot
├── Pagination        [reuse]
└── UserModal         [new]
```

### 3d — Responsive behaviour

Default styles target **mobile** (no prefix). Progressively enhance:

| Breakpoint | Prefix | Typical change                          |
| ---------- | ------ | --------------------------------------- |
| 640px      | `sm:`  | Single-column → two-column forms        |
| 768px      | `md:`  | Stacked → side-by-side cards and fields |
| 1024px     | `lg:`  | Full sidebar, expanded tables           |
| 1280px     | `xl:`  | Max-width containers, wider grids       |

Rules:

- Tables on mobile: `overflow-x-auto` wrapper or switch to card-per-row layout.
- Touch targets: minimum **44×44px** (`min-h-11 min-w-11`).
- Avoid fixed `px` widths on containers; use relative units or Tailwind's fluid classes.
- Never hide critical content with `hidden` on mobile without an accessible alternative.

---

## Phase 4: State Coverage

Every UI must handle all relevant states. Never assume data always exists.

| State             | Render                                           |
| ----------------- | ------------------------------------------------ |
| Loading           | Skeleton or spinner                              |
| Empty             | Empty-state illustration / message               |
| Error             | Inline error with retry action                   |
| Success           | Confirmation feedback                            |
| Submitting        | Disabled controls + loading indicator            |
| Validation errors | Field-level messages via `aria-describedby`      |
| Permission denied | Gated view or redirect                           |
| No results        | Filtered-empty message distinct from empty state |

---

## Phase 5: Integration Artifacts

Generate these alongside the UI. **Do not implement APIs — define contracts only.**

- **Types** — entity types, request/response shapes
- **Form models** — form value types + validation schema structure (Zod, Yup, etc. — match project convention)
- **API contracts** — interface definitions only (e.g. `CreateUserRequest`, `UserResponse`)
- **Mock data** — realistic fixtures for development and testing

---

## Phase 6: Code Generation

Use project patterns, folder structure, and coding standards throughout.

Generate:

- **Pages** — route-level components
- **Components** — all items in the component hierarchy
- **Types** — from Phase 5
- **Constants** — status enums, config values, labels
- **Mock data** — from Phase 5
- **Validation schemas** — from Phase 5
- **Stories** — if the project uses Storybook
- **Tests** — if the project has test infrastructure

### Design system mapping

Map generic concepts to the detected library before writing JSX/templates:

| Generic   | Ant Design | MUI         | Mantine     | shadcn/ui |
| --------- | ---------- | ----------- | ----------- | --------- |
| Button    | `Button`   | `Button`    | `Button`    | `Button`  |
| Modal     | `Modal`    | `Dialog`    | `Modal`     | `Dialog`  |
| Input     | `Input`    | `TextField` | `TextInput` | `Input`   |
| Data grid | `Table`    | `DataGrid`  | `DataTable` | `Table`   |
| Select    | `Select`   | `Select`    | `Select`    | `Select`  |

Always prefer existing project components over library primitives. If no library is detected, use semantic HTML + Tailwind only.

### Accessibility requirements (WCAG 2.1 AA)

Every generated component must meet these rules — not just the planning checklist:

- All `<input>`, `<select>`, `<textarea>` must have a visible `<label>` with matching `htmlFor` ↔ `id`.
- Icon-only buttons must have `aria-label` describing the action.
- Images must have `alt` text; decorative images use `alt=""`.
- Table headers use `<th scope="col">` or `<th scope="row">`.
- Modal / Dialog / Drawer must trap focus and restore it on close — verify the library handles this; implement manually with `focus-trap` if not.
- Error messages must use `role="alert"` or be linked to the field via `aria-describedby`.
- Interactive elements must be reachable and operable via keyboard.
- Use semantic landmarks: `<main>`, `<nav>`, `<section aria-label="...">`, `<aside>`.
- Color must not be the only way to convey meaning — pair with icon or text.
- `tabIndex` values only `0` or `-1`; never positive integers.

### State management rules

- Filter / search / pagination state → **URL `searchParams`**, not `useState`.
- Local UI state (open/closed, selected tab) → `useState` / `useReducer`.
- Server data → project's existing data-fetching pattern (React Query, SWR, server components, etc.).

### Code quality rules

**Must be:** strongly typed · reusable · modular · accessible · responsive · maintainable

**Avoid:** `any` types · hardcoded strings · duplicated logic · inline styles (unless project convention) · dead code · fabricated imports (never import a component that does not exist in the project)

---

## Phase 7: Self-Review

Before responding, verify:

- [ ] Read `CLAUDE.md` and stack confirmed via bash detection
- [ ] Every component annotated `[reuse]`, `[extend]`, or `[new]` — none silently recreated
- [ ] All props/behaviour gaps from the Phase 1 inventory are addressed
- [ ] Component pattern selected from the matrix and justified
- [ ] Semantic HTML used as structural backbone throughout
- [ ] All states from Phase 4 are covered
- [ ] Filter/search/pagination state uses `searchParams`, not `useState`
- [ ] All accessibility rules from Phase 6 applied — not just noted
- [ ] Responsive at all breakpoints; touch targets ≥ 44×44px
- [ ] Fully typed — no unnecessary `any`
- [ ] No fabricated imports — every import verified to exist in the project

---

## Output Format

Always deliver in this order:

1. **Requirement analysis** — actors, goals, actions, business rules
2. **Existing component inventory** — reuse / extend / new decisions with gap notes
3. **UI architecture** — selected pattern, page structure, annotated component hierarchy
4. **State analysis** — which states apply and how they're handled
5. **Integration artifacts** — types, form models, API contracts, mock data
6. **Generated code** — one code block per file, with file path as the heading
7. **File structure** — tree of all new/modified files
8. **Summary of changes** — what was created, extended, reused, and any manual steps needed

---

## Scope Boundaries

**In scope:** pages · components · types · constants · mock data · validation schemas · stories · tests · API contract interfaces

**Out of scope:** API implementation · backend logic · database design · authentication · infrastructure · data migration · routing changes (unless the story explicitly requires a new route)
