---
name: saas-redesign
description: Redesign this Vue 3 app into a modern SaaS layout with a fixed left sidebar nav, 8px spacing scale, and polished professional styling. Invokes vue-expert to rewrite App.vue and normalize view padding. Use when asked to "redesign the UI", "add a sidebar", or "modernize the layout".
---

# SaaS Redesign

Transforms the inventory-management app from horizontal top-nav to a fixed left-sidebar SaaS layout. Project-specific. Executes directly via vue-expert delegation — no plan gate.

## Execution Flow

1. Read `client/src/App.vue` to confirm current structure (`.top-nav` header, `.nav-tabs` horizontal links)
2. Read one view file (e.g. `client/src/views/Dashboard.vue`) to note current outer padding pattern
3. Delegate to `vue-expert` subagent with the full design spec below
4. After vue-expert returns, verify with curl + Playwright screenshots
5. Report results

## Design Tokens

vue-expert MUST add these as CSS custom properties in `App.vue` `:root`:

```css
:root {
  --sidebar-width: 240px;
  --sidebar-width-collapsed: 64px;
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 16px;
  --space-4: 24px;
  --space-5: 32px;
  --space-6: 48px;
  --bg-sidebar: #0f172a;
  --bg-main: #f8fafc;
  --text-sidebar: #94a3b8;
  --text-sidebar-active: #ffffff;
  --accent: #3b82f6;
  --radius: 6px;
}
```

## Target Layout

```
┌─────────┬──────────────────────────┐
│ SIDEBAR │  FilterBar (full width)  │
│ 240px   ├──────────────────────────┤
│ fixed   │  <router-view>           │
│         │  padding: 32px           │
│ nav     │                          │
│ links   │                          │
│ stacked │                          │
│         │                          │
│ ─────── │                          │
│ Lang    │                          │
│ Profile │                          │
└─────────┴──────────────────────────┘
```

## Sidebar Spec

**Container** — `<aside class="sidebar">`:
- `position: fixed; left: 0; top: 0; bottom: 0`
- `width: var(--sidebar-width)`
- `background: var(--bg-sidebar)`
- `display: flex; flex-direction: column`

**Logo block** (top):
- `padding: var(--space-4)`
- Company name `h1` + subtitle span — keep existing `t('nav.companyName')` / `t('nav.subtitle')` calls

**Nav links** (`<nav class="sidebar-nav">`):
- Vertical stack, `gap: var(--space-1)`, `padding: 0 var(--space-2)`
- Each `router-link`: `height: 40px; padding: 0 var(--space-3); border-radius: var(--radius); color: var(--text-sidebar); display: flex; align-items: center`
- Hover: `background: rgba(255,255,255,0.05)`
- Active (`.router-link-active`): `color: var(--text-sidebar-active); background: rgba(59,130,246,0.1); border-left: 3px solid var(--accent)`
- Wrap link text in `<span class="nav-label">` for responsive collapse

**Bottom section**:
- `<div class="sidebar-footer">` with `margin-top: auto; padding: var(--space-3); border-top: 1px solid rgba(255,255,255,0.1)`
- Contains `LanguageSwitcher` then `ProfileMenu`

## Main Content Spec

- `.main-content`: `margin-left: var(--sidebar-width); min-height: 100vh; background: var(--bg-main); display: flex; flex-direction: column`
- `FilterBar` stays at top of main column (NOT in sidebar)
- Wrap `<router-view />` in `<div class="view-container">` with `padding: var(--space-5); flex: 1`

## Responsive

```css
@media (max-width: 1024px) {
  .sidebar { width: var(--sidebar-width-collapsed); }
  .main-content { margin-left: var(--sidebar-width-collapsed); }
  .nav-label { display: none; }
  .sidebar .logo h1, .sidebar .logo .subtitle { display: none; }
}
```

## File Changes

| File | Change |
|---|---|
| `client/src/App.vue` | Replace `<header class="top-nav">` block with `<aside class="sidebar">`. Restructure: sidebar is sibling of `.main-content`, which now wraps FilterBar + router-view. Replace `.top-nav`/`.nav-container`/`.nav-tabs` CSS (~line 185+) with sidebar styles + tokens. |
| `client/src/views/Dashboard.vue` | Remove outer wrapper padding from `<style>` — `.view-container` now owns it |
| `client/src/views/Inventory.vue` | Same — remove outer wrapper padding |
| `client/src/views/Orders.vue` | Same |
| `client/src/views/Demand.vue` | Same |
| `client/src/views/Spending.vue` | Same |
| `client/src/views/Backlog.vue` | Same |
| `client/src/views/Reports.vue` | Same |

## MUST NOT Change

- `App.vue` `<script>` block — refs, computed, methods, lifecycle, imports all stay identical
- `t('nav.*')` i18n keys and calls
- `FilterBar`, `ProfileMenu`, `LanguageSwitcher`, `ProfileDetailsModal`, `TasksModal` — same components, same props, same `@event` wiring
- Route paths and `router-link to=` targets
- Status colors (green/blue/yellow/red) elsewhere
- Any view's `<template>` or `<script>`

## Delegation Prompt Template

Send this to vue-expert (fill in the current-state observations from step 1-2):

> Redesign the inventory-management app shell into a fixed left-sidebar SaaS layout.
>
> **Current state:** App.vue has `<header class="top-nav">` with horizontal `.nav-tabs`. Views have [describe observed padding pattern].
>
> **Design tokens:** [paste the :root block from Design Tokens section]
>
> **Target layout:** [paste ASCII diagram]
>
> **Sidebar spec:** [paste Sidebar Spec section]
>
> **Main content spec:** [paste Main Content Spec section]
>
> **Responsive:** [paste Responsive @media block]
>
> **Files to edit:** [paste File Changes table]
>
> **Do NOT change:** [paste MUST NOT Change list]
>
> When done, list every file you edited with a one-line summary of each change.

## Verification

After vue-expert returns:

1. `curl -s -o /dev/null -w "%{http_code}" http://localhost:3000` — expect `200`. If not, read `/tmp/frontend.log` and report the HMR error instead of continuing.
2. Playwright: navigate to `http://localhost:3000/`, take full-page screenshot. Confirm sidebar visible on left.
3. Playwright: `browser_resize` to 900px width, screenshot. Confirm sidebar collapsed to ~64px, labels hidden.
4. Report both screenshot paths + vue-expert's file list.

## Out of Scope

- Nav link icons (collapsed state shows truncated text — acceptable for this desktop tool)
- Dark/light theme toggle
- User-controlled sidebar expand/collapse
- `FilterBar.vue` internals
- Backend changes
