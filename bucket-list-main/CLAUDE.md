# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A simple, client-side bucket list application built with vanilla JavaScript (no framework). Users create, manage, and track personal goals. All data persists locally via LocalStorage—no server or database required.

**Key tech:** HTML5 + CSS3 (Tailwind CDN) + Vanilla JavaScript ES6+

## Running the Application

```bash
# Option 1: Direct browser open
Open index.html directly in a web browser

# Option 2: Python dev server (recommended for development)
python -m http.server 8000
# Then visit http://localhost:8000

# Option 3: VS Code Live Server
Right-click index.html → "Open with Live Server"
```

## Architecture & Design Patterns

### Separation of Concerns

The codebase follows a simple two-module pattern:

**`js/storage.js`** – Data layer (pure object)
- Single responsibility: manage LocalStorage read/write
- No DOM dependencies
- Methods: `load()`, `save()`, `addItem()`, `updateItem()`, `deleteItem()`, `toggleComplete()`, `getStats()`, `getFilteredList()`
- Returns plain data objects; never updates UI directly

**`js/app.js`** – UI layer (class-based)
- Single responsibility: DOM manipulation and user interaction
- Calls `BucketStorage` methods to persist changes, then calls `render()` to update UI
- Never modifies data directly; always routes through storage module
- Pattern: *User action → Storage update → Full re-render*

### Data Structure

Each bucket item in LocalStorage:
```javascript
{
  id: "1730880000000",          // Timestamp-based unique ID
  title: "Goal description",     // User input (escaped when rendered)
  completed: false,              // Boolean completion status
  createdAt: "2025-11-06T...",  // ISO string creation timestamp
  completedAt: null              // ISO string completion timestamp (null if incomplete)
}
```

### Rendering & State Flow

- `render()` is the single entry point for UI updates; always called after state changes
- No partial updates—full re-render on every change (intentional; dataset is small)
- `updateStats()` recalculates all counters from current data state
- Empty state handled in `render()` when no items match filter

## Important Patterns & Conventions

### XSS Prevention
- User input is escaped via `escapeHtml()` before HTML insertion (uses textContent → innerHTML pattern)
- Applied in `createBucketItemHTML()` when displaying titles and in onclick handlers
- Follow this pattern for any future user-input display

### Performance
- DOM elements cached in `cacheElements()` on init (avoids repeated `getElementById` calls)
- No DOM polling; event-driven updates only
- Inline event handlers use arrow functions to preserve `this` context

### Filtering Logic
- `currentFilter` stored in app instance state
- `BucketStorage.getFilteredList(filter)` returns filtered data; UI renders result
- Three filter types: `'all'`, `'active'` (incomplete), `'completed'`

### UI State
- Modal is shown/hidden via class toggling (`hidden` ↔ `flex`)
- Filter button active state tracked via `active` class (styled in `css/styles.css`)
- No external form libraries; vanilla form submission and reset

## File Locations & Responsibilities

| File | Purpose |
|------|---------|
| `index.html` | DOM structure, form inputs, statistics display, modal, filter buttons |
| `js/storage.js` | Data persistence and CRUD operations |
| `js/app.js` | Event binding, rendering, user interaction handling |
| `css/styles.css` | Animations, filter button styles, responsive adjustments, dark mode support |
| `README.md` | User-facing documentation, feature list, usage guide |

## When Adding Features

1. **Adding a new CRUD operation?** Add method to `BucketStorage` first, then wire it in `BucketListApp` with a call to `this.render()` after.
2. **Modifying data structure?** Update the item object shape in both modules; re-test all operations.
3. **New UI interaction?** Add event listener in `bindEvents()`, handler method in class, and update `render()` if needed.
4. **New styling?** Use Tailwind classes in HTML; add custom CSS to `styles.css` only for animations, pseudo-states, or responsive overrides.
5. **Filtering or display logic?** Check if it belongs in `BucketStorage` (data filtering) or `BucketListApp` (UI state/rendering).

## Common Gotchas

- **Modal state reset**: `closeEditModal()` clears `editingId` and `editInput.value`—important for clean UX on cancel.
- **Filter button state**: Each filter click removes `active` from all buttons, then adds it to the clicked button. Ensure this always syncs with `currentFilter`.
- **Timestamp parsing**: `createdAt` and `completedAt` are ISO strings; use `new Date()` to parse before display formatting.
- **Empty state logic**: `render()` checks list length and shows/hides empty message—changing filter logic requires checking this.

## Browser Compatibility

Modern browsers (Chrome, Firefox, Safari, Edge). Requires:
- LocalStorage API
- ES6 classes
- Template literals
- DOM querySelector/classList APIs

---

**Last updated:** 2026-04-10
