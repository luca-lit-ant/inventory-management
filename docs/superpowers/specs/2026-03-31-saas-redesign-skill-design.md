# saas-redesign Skill — Design Spec

**Date:** 2026-03-31
**Status:** Approved
**Type:** Project-specific Claude Code skill

## Purpose

A slash-command skill (`/saas-redesign`) that transforms this inventory-management app from its current horizontal top-nav layout into a modern SaaS interface with a fixed left sidebar, an 8px spacing scale, and polished professional styling. The skill executes directly — no plan gate — by delegating all `.vue` edits to the `vue-expert` subagent (per CLAUDE.md mandate), then verifies the result with Playwright screenshots.

## Scope Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Generality | Project-specific | Knows exact files, routes, components. No discovery logic needed. |
| Execution model | Direct (no plan preview) | User chose speed over preview gate. |
| Visual spec | Fully prescribed | Pinned px values prevent drift between invocations. |
| Skill structure | Monolithic SKILL.md | Matches `backend-api-test` convention; spec is small enough to inline. |

## Skill File

**Location:** `.claude/skills/saas-redesign/SKILL.md`

**Frontmatter:**
```yaml
name: saas-redesign
description: Redesign this Vue 3 app into a modern SaaS layout with a fixed left sidebar nav, 8px spacing scale, and polished professional styling. Invokes vue-expert to rewrite App.vue and normalize view padding. Use when asked to "redesign the UI", "add a sidebar", or "modernize the layout".
```

## Invocation Flow

1. User runs `/saas-redesign`
2. Skill content loads into Claude's context
3. Claude reads current `client/src/App.vue` + one view file to confirm structure hasn't drifted
4. Claude delegates to `vue-expert` subagent with the full design spec embedded in the prompt
5. vue-expert rewrites `App.vue` and strips outer padding from 7 view files
6. Claude verifies: `curl localhost:3000` → 200, then two Playwright screenshots (desktop + 900px narrow)
7. Claude reports file list + screenshots to user

No user prompts mid-flow.

## Visual Specification

### Layout Structure

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

### Design Tokens

Defined as CSS custom properties in `App.vue` `:root`:

| Token | Value | Purpose |
|---|---|---|
| `--sidebar-width` | 240px | Fixed sidebar width |
| `--sidebar-width-collapsed` | 64px | Narrow viewport width |
| `--space-1` | 4px | Spacing scale (8px base) |
| `--space-2` | 8px | |
| `--space-3` | 16px | |
| `--space-4` | 24px | |
| `--space-5` | 32px | |
| `--space-6` | 48px | |
| `--bg-sidebar` | #0f172a | Slate-900 (existing brand) |
| `--bg-main` | #f8fafc | Slate-50 |
| `--text-sidebar` | #94a3b8 | Slate-400, inactive links |
| `--text-sidebar-active` | #ffffff | Active link text |
| `--accent` | #3b82f6 | Blue-500, active indicator |
| `--radius` | 6px | Border radius |

### Sidebar Component Spec

- **Container:** `<aside class="sidebar">` — `position: fixed; left: 0; top: 0; bottom: 0; width: var(--sidebar-width); display: flex; flex-direction: column`
- **Logo block:** Company name + subtitle, `padding: var(--space-4)`
- **Nav links:** Vertical stack. Each link: `height: 40px; padding: 0 var(--space-3); border-radius: var(--radius); gap: var(--space-1)` between links
- **Active state:** `color: var(--text-sidebar-active); background: rgba(59,130,246,0.1)` + 3px left border in `--accent`
- **Hover state:** `background: rgba(255,255,255,0.05)`
- **Bottom section:** `LanguageSwitcher` + `ProfileMenu` wrapped in a div with `margin-top: auto; padding: var(--space-3); border-top: 1px solid rgba(255,255,255,0.1)`

### Main Content Spec

- `.main-content`: `margin-left: var(--sidebar-width); min-height: 100vh; background: var(--bg-main)`
- `FilterBar` stays at top of main column — not in sidebar
- `<router-view>` wrapper gets `padding: var(--space-5)` (32px)
- Views drop their own outer padding — `.main-content` owns it

### Responsive

- `@media (max-width: 1024px)`: sidebar → `var(--sidebar-width-collapsed)`, nav link labels hidden (`.nav-label { display: none }`), main content `margin-left` adjusts to match
- No mobile drawer/hamburger — this is a desktop-first internal tool

## File Changes

| File | Change | Scope |
|---|---|---|
| `client/src/App.vue` | Template: `<header class="top-nav">` → `<aside class="sidebar">`. Styles: replace `.top-nav`/`.nav-container`/`.nav-tabs` blocks with sidebar + token definitions. | Template + `<style>` only |
| `client/src/views/Dashboard.vue` | Remove outer wrapper padding | `<style>` only |
| `client/src/views/Inventory.vue` | Remove outer wrapper padding | `<style>` only |
| `client/src/views/Orders.vue` | Remove outer wrapper padding | `<style>` only |
| `client/src/views/Demand.vue` | Remove outer wrapper padding | `<style>` only |
| `client/src/views/Spending.vue` | Remove outer wrapper padding | `<style>` only |
| `client/src/views/Backlog.vue` | Remove outer wrapper padding | `<style>` only |
| `client/src/views/Reports.vue` | Remove outer wrapper padding | `<style>` only |

## Explicitly Preserved

vue-expert must NOT change:

- `<script>` block in `App.vue` — all refs, computed, methods, lifecycle, imports stay identical
- `t('nav.*')` i18n keys — same labels, same locales
- `FilterBar`, `ProfileMenu`, `LanguageSwitcher`, `ProfileDetailsModal`, `TasksModal` — same components, same props, same event wiring
- Route paths (`/ /inventory /orders /demand /spending /reports`)
- Existing status colors (green/blue/yellow/red) elsewhere in the app
- Any view's `<template>` or `<script>` — only `<style>` outer-padding edits

## vue-expert Delegation Prompt Structure

The skill instructs Claude to build the delegation prompt from these sections, in order:

1. **Design tokens** — verbatim table from this spec
2. **Layout ASCII** — verbatim diagram
3. **Per-file instructions** — one paragraph per file, with current-state anchors (e.g. "replace the `.top-nav` CSS block starting near line 185")
4. **Preservation list** — verbatim from section above
5. **Output request** — "When done, list every file you edited with a one-line summary of each change"

## Verification

After vue-expert returns:

1. `curl -s -o /dev/null -w "%{http_code}" http://localhost:3000` → expect `200`
2. Playwright: navigate to `http://localhost:3000/`, screenshot full page at default viewport → sidebar visible on left, content offset 240px
3. Playwright: resize to 900px width, screenshot → sidebar collapsed to 64px, labels hidden
4. Report both screenshot paths + vue-expert's file list to user

If step 1 fails (non-200): report the HMR error from `/tmp/frontend.log` instead of screenshotting.

## Out of Scope

- Icons for nav links (sidebar uses text labels only — collapsed state shows truncated text, not icons)
- Dark/light theme toggle
- Sidebar expand/collapse user control (width is viewport-driven only)
- Touching `FilterBar.vue` internals
- Any backend changes
