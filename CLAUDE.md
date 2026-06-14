# QA Tracker — Claude Code Guide

## Role

Senior software engineer mindset: analyze root causes before acting, identify edge cases, review code for quality/security/edge cases, and verify changes manually via browser dev tools (no tests exist).

Single-file vanilla JS SPA (`index.html`). No build, no framework, no tests.

## Must-Know

- **Everything is in `index.html`** (~2273 lines). No modules, no imports. All functions/variables are global.
- **No package.json, no npm, no build step.** Deploy by pushing `index.html` to GitHub Pages.
- **No TypeScript, no linting, no tests.** Validate manually or via browser dev tools.

## Conventions

- `function g(id)` = `document.getElementById(id)` — used everywhere
- `saveLocal(); push(); render();` — mutation pattern after every state change
- All dates in **UTC+7** (hardcoded WIB). `u7()` helper: `new Date(Date.now() + 7*3600000)`
- CSS custom properties for theming, `.dark` class on `<body>` for dark mode
- Inline `onclick`/`onchange` handlers, NOT `addEventListener` (except global keyboard/mouse)
- Template literals + `innerHTML` for all DOM rendering (no `createElement`)
- Constants in `UPPER_CASE`, functions in `camelCase`, global state in `let`

## Architecture

| Layer | Mechanism |
|-------|-----------|
| State | Global `let cases = []` (array of test case objects) |
| Persist | `localStorage` + Supabase REST API (no JS client lib) |
| Images | Cloudinary REST API upload, store URL |
| Auth | Hardcoded `ACCOUNTS` object (`admin`/`guest` + `view` role) |
| Sync | `setInterval(() => pull(true), 60000)` silent merge |

## Data Model (test case)

```js
{ id, testId, name, scenario, module, priority, type, env,
  input, expected, actual, status, notes, date, time, orderId,
  img, imgName, evidence[], credentials[], logs[] }
```

## Section Headers (search anchors)

Lines are organized by `/* ════════════ SECTION ════════════ */`:
`STATE`, `DARK MODE`, `AUTH`, `UTILS`, `MODAL HELPERS`, `STORAGE / SYNC`, `IMAGE HANDLING`, `FORM MODAL`, `FORM CRUD`, `DETAIL VIEW`, `SELECTION & BULK OPS`, `REORDER`, `FILTERS / SORT`, `PAGINATION`, `RENDER`, `CSV EXPORT / IMPORT`, `APP START`

## Rules

- **When a request is ambiguous, unclear, or missing context → ask.** Do not assume intent or fill in gaps without confirmation.
- **Do not over-engineer.** Match the project's simplicity: no build steps, no frameworks, no new files, no abstractions beyond what exists.
- **Prefer the simplest approach that satisfies the requirement.** Clarity and minimalism over elegance.
- **Always create a PR** for every change. Direct push to `main` only when the exact phrase **"push to main"** or **"push directly"** appears in the user instruction. "commit push git" or "push" alone do NOT qualify.
- **When switching to `main`**: always run `git pull --ff-only` first.
- **Do not add new files.** Everything stays in `index.html`.

## Gotchas

- **Hardcoded Supabase service_role key** in source (~line 757). Do not commit to a public repo without removing it.
- **Evidence limit**: `const MAX_EVIDENCE = 5` — enforce in any image addition code.
- **CSV parser** is custom (12-column only), not a library.
- **No loading/error states** on most operations — add `console.error` fallbacks.
- **Dark mode** respects `prefers-color-scheme` but user toggle takes precedence.
- **`render()` does a full DOM rebuild** — expensive but fine for this scale.
