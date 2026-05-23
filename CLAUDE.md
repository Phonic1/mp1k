# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Memory Palace is a single-file, zero-dependency web application for studying using the memory palace (method of loci) technique. The entire application lives in `index.html` (790 lines). `memory-palace.html` is an identical copy.

## Running the App

There is no build step. Open `index.html` directly in a browser, or serve it with any static server:

```bash
python -m http.server 8000
# then open http://localhost:8000
```

There are no tests, no linter, no package manager, and no configuration files.

## Architecture

The app is structured as a single HTML file with three sections:

1. **CSS** (`<style>`, lines 7–483) — all styles inline, using CSS custom properties defined on `:root` for the dark theme (deep navy `#0a0e27`, purple accent `#667eea`, neon green `#00ff41`).

2. **Static data** (`PALACES` object, lines 501–539) — the "database". Two hardcoded palaces (`edinburgh`, `northBerwick`), each with 10 locations. Each location has `name` and `memory` string fields.

3. **`MemoryPalace` class** (lines 542–783) — the entire application controller. One instance (`app`) is created at page load and exposed as a global for inline `onclick` handlers throughout the rendered HTML.

### State Model

```js
state = {
  view: 'universe' | 'palace' | 'location',
  currentPalace: string | null,   // key into PALACES
  currentLocationIndex: number,
  isEditing: boolean
}
```

### Rendering

`render()` does full innerHTML replacement of `#app` on every state change — there is no diffing or virtual DOM. The three render methods (`renderUniverse`, `renderPalace`, `renderLocation`) return HTML strings with inline `onclick="app.methodName()"` event handlers. Breadcrumb is updated separately via `updateBreadcrumb()`.

### Persistence

On load, `loadCustomData()` reads `localStorage.memoryPalaceData` (JSON) and merges edited memories back into `PALACES` by index. `saveCustomData()` serializes the entire `PALACES` object back to localStorage. Editing happens in-place in the `PALACES` object so the source of truth is always `PALACES` at runtime.

### Adding a New Palace

Add a new key to the `PALACES` object following the existing structure:

```js
myPalace: {
  id: 'myPalace',       // must match the key
  name: 'Display Name',
  icon: '🏠',
  description: 'Theme description',
  locations: [
    { name: 'Location Name', memory: 'Memory text...' }
    // up to any number of locations
  ]
}
```

No other code changes are needed — `renderUniverse` and `loadCustomData` iterate `PALACES` dynamically.
