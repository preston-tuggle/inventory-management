---
name: saas-redesign
description: Redesign the Vue 3 app into a modern SaaS-style interface with a vertical left sidebar navigation, consistent spacing scale, and polished professional look. Use when the user wants to convert the top-nav layout to a sidebar layout or apply a SaaS visual refresh.
---

# SaaS UI Redesign

Converts the current **horizontal top-nav layout** into a **vertical left-sidebar layout** with a modern SaaS aesthetic (think Linear, Vercel, Stripe Dashboard). The redesign is layout-and-style only — do not change business logic, API calls, composables, or data flow.

## Target Look

| Aspect | Spec |
|---|---|
| Layout | Fixed left sidebar (`260px`) + fluid content area. Two-column CSS Grid at the `.app` level. |
| Sidebar | Dark slate (`#0f172a` bg, `#94a3b8` text, `#f8fafc` active text). Logo block at top, vertical nav links, LanguageSwitcher + ProfileMenu pinned to bottom. |
| Content area | Light `#f8fafc` bg. FilterBar becomes a sticky top strip inside the content column (not full-width across the sidebar). |
| Spacing scale | 4px base: `4 / 8 / 12 / 16 / 24 / 32 / 48`. Use CSS custom properties. |
| Typography | Inter (already loaded). Page titles `1.5rem / 600`. Section headers `1rem / 600`. Body `0.875rem / 400`. |
| Surfaces | Cards: white, `1px solid #e2e8f0`, `12px` radius, `0 1px 2px rgba(0,0,0,0.04)` shadow. No hover-lift on cards (SaaS convention: hover states on interactive elements only). |
| Nav items | Full-width clickable rows, `10px 16px` padding, `8px` radius. Active: `rgba(255,255,255,0.08)` bg + `2px` left accent bar in `#3b82f6`. Hover: `rgba(255,255,255,0.04)`. |
| Icons | Each nav item gets a leading icon. Use inline SVG (Heroicons outline, 20px) — do **not** add an icon library dependency. |
| Density | Tighter than current. Reduce `.main-content` padding from `1.5rem 2rem` → `24px 32px`. Reduce card padding `1.25rem` → `20px`. |

**Keep untouched:** color semantics for badges/status, table styles, chart SVG internals, all `useFilters` / `useI18n` / `useAuth` wiring, modal components, `t()` translation keys.

---

## Workflow

### Step 1 — Audit current layout

Read these files and note anything that deviates from the baseline described below (the user may have made changes since this skill was written):

- `client/src/App.vue` — baseline: `.app` is a flex column, `.top-nav` is a sticky header with horizontal `.nav-tabs`, `FilterBar` sits between header and `<main>`, global styles live here (non-scoped `<style>`).
- `client/src/components/FilterBar.vue` — baseline: full-width horizontal strip with 4 selects + reset button.
- `client/src/components/ProfileMenu.vue` and `LanguageSwitcher.vue` — understand their root element dimensions so they fit in the sidebar footer.

If the structure has diverged significantly (e.g. nav already moved, new routes added), adapt the plan below — don't blindly apply it.

### Step 2 — Define design tokens

Add a CSS custom-property block at the top of `App.vue`'s global `<style>`:

```css
:root {
  /* Spacing scale (4px base) */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 24px;
  --space-6: 32px;
  --space-7: 48px;

  /* Layout */
  --sidebar-width: 260px;
  --content-max-width: 1600px;

  /* Sidebar palette */
  --sidebar-bg: #0f172a;
  --sidebar-text: #94a3b8;
  --sidebar-text-hover: #e2e8f0;
  --sidebar-text-active: #f8fafc;
  --sidebar-item-hover: rgba(255, 255, 255, 0.04);
  --sidebar-item-active: rgba(255, 255, 255, 0.08);
  --sidebar-accent: #3b82f6;
  --sidebar-border: rgba(255, 255, 255, 0.08);

  /* Content palette (keep existing slate system) */
  --surface-bg: #ffffff;
  --surface-border: #e2e8f0;
  --surface-radius: 12px;
  --surface-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);

  --text-primary: #0f172a;
  --text-secondary: #64748b;
  --text-tertiary: #94a3b8;
}
```

Retrofit existing global rules (`.card`, `.stat-card`, `.page-header`, etc.) to reference these tokens rather than hardcoded values. Do this surgically — only touch properties that map cleanly to a token.

### Step 3 — Restructure App.vue template

Delegate this edit to the **vue-expert** subagent (per CLAUDE.md rules). Give it this exact target structure:

```vue
<template>
  <div class="app">
    <aside class="sidebar">
      <div class="sidebar-logo">
        <h1>{{ t('nav.companyName') }}</h1>
        <span class="sidebar-subtitle">{{ t('nav.subtitle') }}</span>
      </div>

      <nav class="sidebar-nav">
        <router-link to="/" class="nav-item">
          <svg><!-- home icon --></svg>
          <span>{{ t('nav.overview') }}</span>
        </router-link>
        <router-link to="/inventory" class="nav-item">
          <svg><!-- cube icon --></svg>
          <span>{{ t('nav.inventory') }}</span>
        </router-link>
        <router-link to="/orders" class="nav-item">
          <svg><!-- clipboard icon --></svg>
          <span>{{ t('nav.orders') }}</span>
        </router-link>
        <router-link to="/spending" class="nav-item">
          <svg><!-- currency icon --></svg>
          <span>{{ t('nav.finance') }}</span>
        </router-link>
        <router-link to="/demand" class="nav-item">
          <svg><!-- chart icon --></svg>
          <span>{{ t('nav.demandForecast') }}</span>
        </router-link>
        <router-link to="/reports" class="nav-item">
          <svg><!-- document icon --></svg>
          <span>Reports</span>
        </router-link>
      </nav>

      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <div class="content-area">
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <!-- modals stay at root level, unchanged -->
    <ProfileDetailsModal ... />
    <TasksModal ... />
  </div>
</template>
```

**Critical instructions for the subagent:**
- The `<script>` block does not change. All refs, computed, methods, imports stay identical.
- Remove the `:class="{ active: ... }"` bindings — `router-link` applies `.router-link-active` automatically. Style that class instead.
- Use Heroicons outline SVGs inline (20×20 viewBox, `stroke="currentColor"`, `stroke-width="1.5"`). Suggested mappings: Overview→`home`, Inventory→`cube`, Orders→`clipboard-document-list`, Finance→`banknotes`, Demand→`chart-bar`, Reports→`document-text`.
- Keep all `t()` calls exactly as they are.

### Step 4 — Restructure App.vue layout CSS

Replace the layout portion of the global stylesheet. **Only the layout rules** — leave `.badge`, `.table-container`, `table/thead/th/td`, `.loading`, `.error` completely untouched.

```css
.app {
  display: grid;
  grid-template-columns: var(--sidebar-width) 1fr;
  min-height: 100vh;
}

/* ---------- Sidebar ---------- */
.sidebar {
  background: var(--sidebar-bg);
  display: flex;
  flex-direction: column;
  height: 100vh;
  position: sticky;
  top: 0;
  border-right: 1px solid var(--sidebar-border);
}

.sidebar-logo {
  padding: var(--space-5) var(--space-4);
  border-bottom: 1px solid var(--sidebar-border);
}

.sidebar-logo h1 {
  font-size: 1.125rem;
  font-weight: 700;
  color: var(--sidebar-text-active);
  letter-spacing: -0.01em;
  margin-bottom: var(--space-1);
}

.sidebar-subtitle {
  font-size: 0.75rem;
  color: var(--sidebar-text);
  font-weight: 400;
}

.sidebar-nav {
  flex: 1;
  padding: var(--space-3) var(--space-2);
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: 10px var(--space-4);
  color: var(--sidebar-text);
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  border-radius: 8px;
  transition: background-color 0.12s ease, color 0.12s ease;
  position: relative;
}

.nav-item svg {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

.nav-item:hover {
  background: var(--sidebar-item-hover);
  color: var(--sidebar-text-hover);
}

.nav-item.router-link-exact-active {
  background: var(--sidebar-item-active);
  color: var(--sidebar-text-active);
}

.nav-item.router-link-exact-active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 8px;
  bottom: 8px;
  width: 2px;
  background: var(--sidebar-accent);
  border-radius: 0 2px 2px 0;
}

.sidebar-footer {
  padding: var(--space-4);
  border-top: 1px solid var(--sidebar-border);
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
}

/* ---------- Content area ---------- */
.content-area {
  display: flex;
  flex-direction: column;
  min-width: 0; /* prevents grid blowout from wide tables */
}

.main-content {
  flex: 1;
  max-width: var(--content-max-width);
  width: 100%;
  margin: 0 auto;
  padding: var(--space-5) var(--space-6);
}

/* ---------- Updated surfaces ---------- */
.card,
.stat-card {
  background: var(--surface-bg);
  border: 1px solid var(--surface-border);
  border-radius: var(--surface-radius);
  box-shadow: var(--surface-shadow);
  padding: 20px;
}

.card {
  margin-bottom: var(--space-4);
}

.stat-card:hover {
  /* remove the old border/shadow hover — keep surfaces static */
  border-color: var(--surface-border);
  box-shadow: var(--surface-shadow);
}

.stats-grid {
  gap: var(--space-4);
  margin-bottom: var(--space-5);
}

.page-header {
  margin-bottom: var(--space-5);
}

.page-header h2 {
  font-size: 1.5rem;
  font-weight: 600;
}
```

**Delete** these now-dead rules: `.top-nav`, `.nav-container`, `.nav-container > .nav-tabs`, `.nav-container > .language-switcher`, `.logo`, `.logo h1`, `.subtitle`, `.nav-tabs`, `.nav-tabs a`, `.nav-tabs a:hover`, `.nav-tabs a.active`, `.nav-tabs a.active::after`.

### Step 5 — Adapt FilterBar

The FilterBar currently assumes full page width. It now lives inside `.content-area` and should become a slim sticky strip at the top of the content column.

Delegate to **vue-expert**. Requirements:
- Wrap in a sticky container: `position: sticky; top: 0; z-index: 10; background: #f8fafc; border-bottom: 1px solid var(--surface-border);`
- Reduce vertical padding to `var(--space-3)`.
- Keep horizontal padding aligned with `.main-content` (`var(--space-6)`).
- Selects stay side-by-side (existing grid is fine), but shrink label font to `0.75rem` with `color: var(--text-tertiary)`.
- Do not touch the `useFilters` composable wiring or the reset button logic.

### Step 6 — Fix LanguageSwitcher & ProfileMenu for dark background

These components were styled for a white header. They now sit in a dark sidebar footer.

Read both components first. Then apply **minimal** overrides — prefer adding a contextual rule in `App.vue` over editing the components themselves:

```css
.sidebar-footer .language-switcher,
.sidebar-footer .profile-menu-trigger {
  /* override text/border colors for dark bg as needed */
}
```

Only edit the component files directly if their internal structure makes contextual overrides impossible (e.g. hardcoded white backgrounds inside scoped styles). If you must edit them, delegate to **vue-expert**.

### Step 7 — Responsive fallback

Below `1024px` the sidebar collapses to icon-only (`64px` wide). Labels hide, tooltips appear on hover.

```css
@media (max-width: 1024px) {
  .app {
    grid-template-columns: 64px 1fr;
  }
  .sidebar-logo h1,
  .sidebar-subtitle,
  .nav-item span {
    display: none;
  }
  .sidebar-logo {
    padding: var(--space-4) 0;
    text-align: center;
  }
  .nav-item {
    justify-content: center;
    padding: 10px;
  }
  .sidebar-footer {
    padding: var(--space-3);
  }
  .main-content {
    padding: var(--space-4);
  }
}
```

Add `title` attributes to each `.nav-item` so the browser tooltip shows the label when collapsed.

### Step 8 — Verify visually

1. Ensure servers are running (`/start` if needed).
2. Use Playwright to screenshot each route at `1440×900` and `900×700`:
   - `http://localhost:3000/` (Overview)
   - `http://localhost:3000/inventory`
   - `http://localhost:3000/orders`
   - `http://localhost:3000/spending`
   - `http://localhost:3000/demand`
   - `http://localhost:3000/reports`
3. Check for:
   - Sidebar footer components are legible on dark bg
   - No horizontal scroll on any route
   - FilterBar sticks correctly when scrolling long tables (Inventory, Orders)
   - Active nav state highlights the correct item on each route
   - Modals (`ProfileDetailsModal`, `TasksModal`) still open and overlay correctly — their `z-index` must exceed the sidebar's

Fix any regressions before declaring done.

---

## Constraints Recap

- **No new dependencies.** Inline SVG icons only.
- **No emojis in UI.** (Project rule.)
- **No logic changes.** `<script>` blocks in `App.vue` and `FilterBar.vue` stay byte-identical except for template-required additions (e.g. none needed).
- **Delegate `.vue` edits to vue-expert.** (CLAUDE.md mandatory rule.)
- **Global styles stay in App.vue.** Don't extract to a separate CSS file — that's a different refactor.
