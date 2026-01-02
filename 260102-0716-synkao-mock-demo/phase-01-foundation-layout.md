# Phase 1: Foundation + Layout Components

## Context Links
- **Parent Plan:** [plan.md](./plan.md)
- **Dependencies:** None (first phase)
- **Design System:** [synkao-design-system.md](../../docs/design/synkao-design-system.md)
- **UI Spec:** [synkao-page-ui-spec.md](../../docs/design/synkao-page-ui-spec.md)

---

## Overview

| Property | Value |
|----------|-------|
| Priority | P1 |
| Status | ✓ completed |
| Effort | 1.5 days |
| Completed | 2026-01-02 08:48 UTC |
| Description | Setup route groups, install shadcn components, create layout components (GlassSidebar, AppHeader, PageHeader) |
| Review | [code-reviewer-260102-0838-phase-1-review.md](../reports/code-reviewer-260102-0838-phase-1-review.md) |

---

## Key Insights (from Research)

1. **Route Groups** - Use `(auth)` and `(main)` to isolate layouts
2. **Glass Sidebar** - `bg-white/60 backdrop-blur-md border-r border-slate-200/80`
3. **shadcn Sidebar** - Composable, supports collapsible states, uses SidebarProvider
4. **Header** - 64px height, sticky top, white background

---

## Requirements

### Functional
- F1: Install all required shadcn components
- F2: Create route group structure `(auth)` and `(main)`
- F3: GlassSidebar with navigation items
- F4: AppHeader with user dropdown
- F5: PageHeader component for page titles + breadcrumbs
- F6: Root layout with providers

### Non-Functional
- NF1: Glass morphism effect consistent
- NF2: Desktop only (1280px+ viewport)
- NF3: Sidebar 240px expanded, 64px collapsed

---

## Architecture

### Component Tree
```
RootLayout
├── Providers (Query, Theme)
└── Children
    ├── (auth)/layout.tsx
    │   └── Centered card container
    └── (main)/layout.tsx
        ├── SidebarProvider
        │   ├── GlassSidebar
        │   └── Main Content
        │       ├── AppHeader
        │       └── Page Content
```

### GlassSidebar Structure
```tsx
<Sidebar className="bg-white/60 backdrop-blur-md border-r border-slate-200/80">
  <SidebarHeader>
    <Logo />
  </SidebarHeader>
  <SidebarContent>
    <SidebarGroup>
      <SidebarMenuItem>Dashboard</SidebarMenuItem>
      <SidebarMenuItem>Orders</SidebarMenuItem>
      <SidebarMenuItem>Design (expandable)</SidebarMenuItem>
      <SidebarMenuItem>Settings</SidebarMenuItem>
    </SidebarGroup>
  </SidebarContent>
  <SidebarFooter>
    <UserInfo />
  </SidebarFooter>
</Sidebar>
```

---

## Related Code Files

### Rename (FIRST)
| From | To | Reason |
|------|-----|--------|
| `apps/web/src/app/(dashboard)/` | `apps/web/src/app/(main)/` | Consistency với plan, chứa nhiều modules |

### Create
| File Path | Description |
|-----------|-------------|
| `apps/web/src/app/(auth)/layout.tsx` | Auth layout - centered glass card, gradient bg |
| `apps/web/src/app/(main)/layout.tsx` | Main layout - SidebarProvider + GlassSidebar + AppHeader |
| `apps/web/src/app/(main)/page.tsx` | Dashboard placeholder (move từ app/page.tsx) |
| `apps/web/src/app/(main)/orders/page.tsx` | Orders list placeholder |
| `apps/web/src/app/(main)/design/backlog/page.tsx` | Design backlog placeholder |
| `apps/web/src/app/(main)/design/workspace/page.tsx` | Workspace list placeholder |
| `apps/web/src/app/(main)/settings/team/page.tsx` | Team settings placeholder |
| `apps/web/src/components/layout/glass-sidebar.tsx` | Glass morphism sidebar wrapper |
| `apps/web/src/components/layout/sidebar-nav-items.tsx` | Navigation items renderer |
| `apps/web/src/components/layout/app-header.tsx` | Top header bar (64px, sticky) |
| `apps/web/src/components/layout/page-header.tsx` | Page title + breadcrumb + actions |
| `apps/web/src/components/layout/user-menu.tsx` | User dropdown (avatar + menu) |
| `apps/web/src/components/layout/notification-bell.tsx` | Notification icon with badge (static) |
| `apps/web/src/components/layout/logo.tsx` | Logo component |
| `apps/web/src/components/layout/index.ts` | Barrel exports |
| `apps/web/src/lib/navigation.ts` | Nav config: routes, icons, badges |
| `apps/web/src/lib/constants.ts` | App constants: colors, sizes |

### Modify
| File Path | Change Description |
|-----------|-------------------|
| `apps/web/src/app/layout.tsx` | Add Be Vietnam Pro + Inter + JetBrains Mono fonts, update metadata |
| `apps/web/src/app/globals.css` | Add glass CSS utilities (.glass-card, .glass-sidebar) |
| `apps/web/tailwind.config.ts` | Extend: colors.glass, colors.phase, backdropBlur.glass |
| `apps/web/src/components/providers/index.tsx` | Wrap children với SidebarProvider |

### Delete
| File Path | Reason |
|-----------|--------|
| `apps/web/src/app/page.tsx` | Move vào (main)/page.tsx |
| `apps/web/src/components/layout/.gitkeep` | Folder sẽ có files |

---

## Implementation Steps

### 1. Install shadcn Components (15 min)
```bash
cd apps/web
npx shadcn-ui@latest add sidebar sheet dialog dropdown-menu avatar badge breadcrumb separator skeleton tabs progress table
```

### 2. Update Tailwind Config (15 min)
Add custom glass utilities and phase colors:
```typescript
// tailwind.config.ts
extend: {
  colors: {
    glass: {
      bg: 'rgba(255, 255, 255, 0.7)',
      hover: 'rgba(255, 255, 255, 0.85)',
      border: 'rgba(255, 255, 255, 0.2)',
    },
    // Phase colors from design spec
    phase: {
      order: '#64748B',      // Slate 500
      design: '#F59E0B',     // Amber 500
      fulfill: '#8B5CF6',    // Violet 500
      complete: '#10B981',   // Emerald 500
      cancelled: '#6B7280',  // Gray 500
    }
  },
  backdropBlur: {
    glass: '10px',
    modal: '16px',
  }
}
```

### 3. Create Navigation Config (20 min)
```typescript
// apps/web/src/lib/navigation.ts
export const mainNavigation = [
  { title: 'Dashboard', href: '/', icon: LayoutDashboard },
  { title: 'Orders', href: '/orders', icon: Package, badge: 12 },
  {
    title: 'Design',
    icon: Palette,
    children: [
      { title: 'Backlog', href: '/design/backlog', badge: 5 },
      { title: 'Workspaces', href: '/design/workspace' },
    ]
  },
  { title: 'Settings', href: '/settings/team', icon: Settings },
];
```

### 4. Create GlassSidebar Component (1 hour)
- Use shadcn Sidebar as base
- Apply glass morphism classes
- Implement navigation items from config
- Add user info in footer
- Handle active state via pathname

### 5. Create AppHeader Component (30 min)
- 64px height, sticky top
- Sidebar toggle button
- Page title (dynamic)
- Notification bell
- User dropdown menu

### 6. Create PageHeader Component (30 min)
- Title prop
- Breadcrumb prop (optional)
- Actions slot (optional)

### 7. Create Auth Layout (30 min)
- Centered container
- Glass card effect
- No sidebar/header

### 8. Create Main Layout (45 min)
- SidebarProvider wrapper
- GlassSidebar
- AppHeader
- Main content area with scroll

### 9. Update Root Layout (30 min)
- Import fonts: Be Vietnam Pro (headings), Inter (body), JetBrains Mono (code)
- Configure font variables in CSS
- Add QueryClientProvider
- Add ThemeProvider if needed

```tsx
// apps/web/src/app/layout.tsx
import { Be_Vietnam_Pro, Inter, JetBrains_Mono } from 'next/font/google';

const beVietnamPro = Be_Vietnam_Pro({
  subsets: ['vietnamese', 'latin'],
  weight: ['400', '500', '600', '700'],
  variable: '--font-heading',
});

const inter = Inter({
  subsets: ['latin', 'vietnamese'],
  variable: '--font-body',
});

const jetbrainsMono = JetBrains_Mono({
  subsets: ['latin'],
  variable: '--font-mono',
});

// In body className:
// className={`${beVietnamPro.variable} ${inter.variable} ${jetbrainsMono.variable} font-body`}
```

### 10. Create Placeholder Pages (30 min)
- Dashboard placeholder
- Orders list placeholder
- Design backlog placeholder
- Use existing mock data for counts

---

## Todo List

- [x] Install shadcn components (sidebar, sheet, dialog, dropdown-menu, avatar, badge, breadcrumb, separator, skeleton, tabs, progress, table)
- [x] Update tailwind.config.ts with glass utilities and phase colors (via globals.css @theme inline)
- [x] Create `apps/web/src/lib/navigation.ts`
- [x] Create `apps/web/src/components/layout/glass-sidebar.tsx`
- [x] Create `apps/web/src/components/layout/sidebar-nav.tsx`
- [x] Create `apps/web/src/components/layout/app-header.tsx`
- [x] Create `apps/web/src/components/layout/user-menu.tsx`
- [x] Create `apps/web/src/components/layout/notification-bell.tsx`
- [x] Create `apps/web/src/components/layout/page-header.tsx`
- [x] Create `apps/web/src/components/layout/index.ts`
- [x] Create `apps/web/src/app/(auth)/layout.tsx`
- [x] Create `apps/web/src/app/(main)/layout.tsx`
- [x] Move `apps/web/src/app/page.tsx` to `apps/web/src/app/(main)/page.tsx`
- [x] Update `apps/web/src/app/layout.tsx` with fonts (Be Vietnam Pro, Inter, JetBrains Mono)
- [x] Update globals.css with glass utilities
- [x] Test navigation between pages (build succeeds)
- [x] Verify glass morphism effect (backdrop-blur-[12px] applied)

---

## Success Criteria

- [x] All shadcn components installed successfully
- [x] Sidebar shows with glass effect
- [x] Navigation items render correctly
- [x] Active route highlighted in sidebar
- [x] Header displays with user dropdown
- [x] Auth layout renders centered
- [x] Main layout renders with sidebar + header
- [x] Page transitions work smoothly

---

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| shadcn version conflicts | Medium | Pin exact versions |
| Glass blur performance | Low | Use moderate blur (10-12px) |
| Sidebar state persistence | Low | Use localStorage via useSidebar |

---

## Next Steps

After completion, proceed to [Phase 2: Design Module](./phase-02-design-module.md)
