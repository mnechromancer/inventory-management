# SaaS UI Redesign Design Spec

**Date:** 2026-06-22
**Output:** Modified `App.vue`, new `AppSidebar.vue`, modified `FilterBar.vue`

## Goal

Redesign the inventory management app's UI from a horizontal top navigation bar to a modern SaaS-style interface with a vertical sidebar, elevated card layouts, and a polished professional aesthetic — using only global CSS changes (no view files touched).

## Decisions Made

| Decision | Choice | Reason |
|---|---|---|
| Sidebar style | Dark (`#0f172a`) | High contrast, Linear/Vercel aesthetic, already in the app's palette |
| Card style | Elevated | White cards + drop shadow on `#f1f5f9` bg — most common SaaS pattern |
| Scope | Global CSS only | Views unchanged; all card/typography improvements via unscoped globals in App.vue |
| Responsive | Desktop only | No mobile breakpoints in this pass |

## Layout Architecture

### Shell (App.vue)

CSS Grid two-column layout replaces the current flex-column `.app`:

```css
.app-shell {
  display: grid;
  grid-template-columns: 240px 1fr;
  min-height: 100vh;
  /* NO overflow: hidden/auto — breaks sticky */
}

.main-area {
  display: flex;
  flex-direction: column;
  overflow-y: auto;       /* scroll target is here, not app-shell */
  background: #f1f5f9;   /* light gray page background */
}

.topbar {
  position: sticky;
  top: 0;
  z-index: 90;
  background: #ffffff;
  border-bottom: 1px solid #e2e8f0;
  padding: 0.6rem 1.5rem;
}

.main-content {
  flex: 1;
  padding: 1.5rem 2rem;
  /* No max-width — sidebar already constrains width */
}
```

### Sidebar (AppSidebar.vue — new file)

```css
.sidebar {
  position: sticky;     /* NOT fixed — stays in grid flow */
  top: 0;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #0f172a;
  overflow-y: auto;
}
```

Structure:
- **Logo block** — "Factory IMS" + subtitle, bottom border divider
- **Nav section** — `v-for` over `navItems` array, `useRoute()` + `startsWith` for active state, inline SVG icons (no library)
- **Footer** — `ProfileMenu` + `LanguageSwitcher`, pinned via `margin-top: auto`

Nav item active state:
```js
// Root path uses strict equality; others use startsWith
item.path === '/' ? route.path === '/' : route.path.startsWith(item.path)
```

Active indicator: `3px solid #2563eb` left border via `::before` pseudo-element.

Modal events emitted up to App.vue: `@show-profile-details`, `@show-tasks`.

## Design Tokens (all existing palette values)

| Element | Value |
|---|---|
| Sidebar bg | `#0f172a` |
| Page bg | `#f1f5f9` |
| Topbar bg | `#ffffff` |
| Card bg | `#ffffff` |
| Card shadow | `0 1px 3px rgba(0,0,0,0.08), 0 4px 12px rgba(0,0,0,0.04)` |
| Card border-radius | `10px` |
| Nav default text | `#64748b` |
| Nav hover | `#f1f5f9` text + `rgba(255,255,255,0.06)` bg |
| Nav active | `#ffffff` text + `rgba(255,255,255,0.12)` bg |
| Active indicator | `3px solid #2563eb` |
| Sidebar dividers | `rgba(255,255,255,0.07)` |
| Topbar border | `1px solid #e2e8f0` |

## Global CSS Changes (App.vue unscoped styles)

### Cards
```css
.card, .metric-card {
  background: #ffffff;
  border-radius: 10px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.08), 0 4px 12px rgba(0,0,0,0.04);
  border: none;   /* remove existing 1px border */
}
```

### KPI metric cards
```css
.metric-label {
  font-size: 0.65rem;
  font-weight: 600;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}
.metric-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0f172a;
}
```

### Tables
```css
.table-container {
  background: #ffffff;
  border-radius: 10px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.08), 0 4px 12px rgba(0,0,0,0.04);
  overflow: hidden;
}
thead th {
  background: #f8fafc;
  font-size: 0.65rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #64748b;
}
```

### Page-level typography
```css
.page-title {
  font-size: 1.125rem;
  font-weight: 600;
  color: #0f172a;
}
.page-subtitle {
  font-size: 0.8rem;
  color: #64748b;
  margin-top: 2px;
}
```

## FilterBar Changes

Two targeted fixes — scoped CSS in `FilterBar.vue`:

1. Remove `position: sticky; top: 70px` (topbar owns stickiness now)
2. Remove `max-width: 1600px; margin: 0 auto` on `.filters-container`

```css
.filters-bar {
  position: static;
  max-width: none;
  padding: 0;
  border-bottom: none;
  background: transparent;
}
.filters-container {
  max-width: none;
  margin: 0;
  padding: 0;
  width: 100%;
}
```

## Files Changed

| File | Type | Change |
|---|---|---|
| `client/src/App.vue` | Modify | Grid shell, remove top-nav template + CSS, add global card/table/typography styles |
| `client/src/components/AppSidebar.vue` | Create | Logo, nav items, ProfileMenu/LanguageSwitcher footer, modal emits |
| `client/src/components/FilterBar.vue` | Modify | Remove sticky offset + max-width from scoped CSS |

## Out of Scope

- `src/views/*.vue` — untouched
- `src/main.js` — untouched
- `src/api.js`, `src/composables/` — untouched
- Mobile/responsive breakpoints
- Icon library (inline SVGs only)
- Dark mode content area
