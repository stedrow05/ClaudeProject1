# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Agenda** is a single-file browser-based calendar and to-do app deployed on GitHub Pages. The entire application — HTML structure, CSS, and JavaScript — lives in `index.html`. There is no build tool, no package manager, and no external dependencies.

To run: open `index.html` directly in a browser, or serve it with any static file server (e.g. `npx serve .`).

## Architecture

### Single-file structure
All code is in `index.html` in this order:
1. `<style>` block — all CSS, organized with `/* ── Section ── */` comments
2. HTML body — static shell (header, sidebar, calendar area, two modals)
3. `<script>` block — all JavaScript, organized with `// ═══ SECTION ═══` comments

### State and persistence
A single global `state` object holds all runtime data:
```js
state = { view, curDate, events, todos, categories, todoFilter, todoFilterDate, _color, _editId }
```
`load()` / `save()` serialize state to `localStorage` under the key `teacherAgenda2`.

### Data models
- **Event**: `{ id, title, date (YYYY-MM-DD), categoryId, startTime, endTime, repeat, repeatUntil, description, color }`
- **Todo**: `{ id, text, completed, createdAt, doDate, dueDate }` — `doDate` controls which day's list it appears in; `dueDate` is the deadline
- **Category**: `{ id, name, color }` — user-defined; defaults are Lesson, Meeting, Recess/Lunch, Special Event, Admin, Other

### Rendering pattern
All views render by building an HTML string and assigning it to `innerHTML`. The main dispatcher is `render()`, which calls `renderMonthly()`, `renderWeekly()`, or `renderDaily()` depending on `state.view`.

### Key functions
| Function | Purpose |
|---|---|
| `eventsOnDate(ds)` | Returns events for a date string, including weekly recurrences |
| `getEventColor(ev)` | Resolves effective color: category color → fallback `ev.color` |
| `evTopPx(ev, hourPx)` / `evHeightPx(ev, hourPx)` | Pixel position/height for timed events in time-grid views |
| `esc(s)` | HTML-escapes user content — must be used for all user-supplied strings rendered into HTML |

### Time grid
Weekly (`HOUR_PX = 56`) and daily (`HOUR_PX = 64`) views use absolute positioning. Hours shown: 7am–9pm (the `HOURS` array, indices 7–21). Event blocks are positioned with `top` and `height` in pixels derived from `evTopPx` / `evHeightPx`.

### Recurring events
Only `repeat: 'weekly'` is supported. Recurrence is computed on-the-fly in `eventsOnDate` — there are no expanded instances stored. Editing a recurring event edits the base record and affects all occurrences.

## Deployment
The app is hosted on GitHub Pages directly from the `main` branch. Pushing to `main` publishes automatically. The entry point must remain `index.html` at the repo root.
