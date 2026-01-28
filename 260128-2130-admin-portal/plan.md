---
title: "Admin Portal Implementation"
description: "Separate admin portal with dashboard, user/space management, and role-based access"
status: pending
priority: P2
effort: 6h
branch: main
tags: [admin, dashboard, auth, recharts, ui]
created: 2026-01-28
---

# Admin Portal Implementation

## Overview

Implement a separate Admin Portal with its own route group `(admin)`, featuring a horizontal top navigation layout distinct from the main app's sidebar. The portal includes dashboard with statistics/charts, and placeholder pages for space and user management.

## Key Requirements

1. **Separate Route Group**: `(admin)` route group with own layout
2. **Role-based Access**: `AdminGuard` checks `user.role === 'ADMIN'`
3. **Horizontal Nav**: Top navigation bar (not sidebar like main app)
4. **Dashboard**: Stats cards + Recharts visualizations
5. **Navigation Integration**: "Admin Portal" in UserMenu, "Back to App" in admin header
6. **Shared Theme**: Reuse existing providers/dark mode

## Architecture

```
apps/web/src/
├── app/(admin)/
│   ├── layout.tsx           # Admin layout with AdminGuard + AdminHeader
│   ├── page.tsx             # Dashboard with stats/charts
│   ├── spaces/
│   │   ├── page.tsx         # Spaces list placeholder
│   │   └── [id]/page.tsx    # Space detail placeholder
│   └── users/
│       ├── page.tsx         # Users list placeholder
│       └── [id]/page.tsx    # User detail placeholder
├── components/
│   ├── auth/admin-guard.tsx
│   └── layout/
│       ├── admin-header.tsx
│       └── admin-nav.tsx
├── features/admin/
│   └── components/
│       ├── stats-cards.tsx
│       └── charts/
│           ├── revenue-chart.tsx
│           └── users-chart.tsx
└── lib/navigation.ts        # Add adminNavigation config
```

## Dependencies

- **New**: `recharts` package (charts library)
- **Existing**: shadcn/ui Card, lucide-react icons, Tailwind CSS

## Phase Summary

| Phase | Description | Effort |
|-------|-------------|--------|
| [Phase 1](./phase-01-setup-navigation-and-guards.md) | Navigation config + AdminGuard | 1.5h |
| [Phase 2](./phase-02-layout-and-header-components.md) | Admin layout + header components | 1.5h |
| [Phase 3](./phase-03-dashboard-with-charts.md) | Dashboard stats cards + Recharts | 2h |
| [Phase 4](./phase-04-placeholder-pages.md) | Spaces/Users placeholder pages | 1h |

## Success Criteria

- [ ] Admin users see "Admin Portal" in UserMenu dropdown
- [ ] Non-admin users cannot access /admin/* routes
- [ ] Dashboard displays stats cards with mock data
- [ ] Revenue and user charts render correctly with Recharts
- [ ] "Back to App" returns to main app
- [ ] Mobile-responsive layout
- [ ] No compilation errors

## Risk Assessment

| Risk | Mitigation |
|------|------------|
| Recharts not installed | Phase 1 includes pnpm add recharts |
| Theme conflicts | Reuse existing Providers, no new theme wrapper |
| Auth edge cases | Copy pattern from existing AuthGuard |

## Notes

- Uses existing mock user data with `role: 'ADMIN'`
- Dashboard data is fake/mock for now
- Charts use Recharts (simpler than Chart.js for React)
- No backend integration required for MVP
