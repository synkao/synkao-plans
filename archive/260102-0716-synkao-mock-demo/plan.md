---
title: "SynKao Mock Demo - Static UI/UX Validation"
description: "Implement static UI demo for stakeholder validation before production development"
status: in-progress
priority: P1
effort: 10d
branch: main
tags: [ui, demo, glassmorphism, shadcn, dnd-kit]
created: 2026-01-02
validated: 2026-01-02
phase-1-completed: 2026-01-02
---

# SynKao Mock Demo Implementation Plan

## Validation Summary

**Validated:** 2026-01-02
**Questions asked:** 4

### Confirmed Decisions
- **Kanban UX:** Thêm drag-drop với dnd-kit (+1-2 ngày)
- **Deploy:** Giữ nguyên Dockerfile standalone (Node.js, không static export)
- **Priority:** Design Module trước Orders Module
- **Mock Data:** Thêm design notes, timeline events, design versions

### Action Items
- [ ] Update Phase 2: thêm dnd-kit integration steps
- [ ] Update Phase 5: sử dụng Dockerfile hiện tại (standalone mode)
- [ ] Bổ sung mock data: design_versions.data.ts, timeline events

---

## Overview

Static UI/UX validation demo - validate design decisions w/ stakeholders BEFORE production dev.

**Approach:** Next.js Standalone + shadcn/ui + Tailwind CSS + Functional Glassmorphism + dnd-kit

**Constraints:**
- WITH drag-drop for Kanban (dnd-kit library)
- No API integration (use existing mock data)
- No form validation (hardcode success states)
- Desktop only (1280px+)
- Enhanced mock data from `apps/web/src/mocks/`

---

## Tech Stack

| Layer | Technology | Version |
|-------|------------|---------|
| Framework | Next.js | 16 |
| UI Library | shadcn/ui | latest |
| Styling | Tailwind CSS | 4 |
| Drag & Drop | dnd-kit | latest |
| State | Zustand | existing |
| Mock Data | MSW + TypeScript | existing |
| Deploy | Docker + Node.js | existing (standalone) |

---

## Existing Assets

### Mock Data (already implemented)
- **15 orders** - `apps/web/src/mocks/data/orders.data.ts`
- **45 tasks** - `apps/web/src/mocks/data/tasks.data.ts` (15 TODO, 10 IN_PROGRESS, 8 REVIEW, 5 REVISION, 7 DONE)
- **6 users** - 1 Admin, 2 Staff, 3 Designers
- **2 workspaces** - Main + Archive
- **Order items** - 1:1 with tasks

### shadcn Components (installed)
- button, card, input

### Stores (configured)
- auth.store.ts
- ui.store.ts

---

## Phase Overview

| Phase | Description | Effort | Priority | Status |
|-------|-------------|--------|----------|--------|
| [Phase 1](./phase-01-foundation-layout.md) | Foundation + Layout Components | 1.5d | P1 | ✓ DONE |
| [Phase 2](./phase-02-design-module.md) | Design Module (Kanban w/ drag-drop, Backlog, Drawer) | 3.5d | P1 | pending |
| [Phase 3](./phase-03-orders-module.md) | Orders Module (List, Detail, Import) | 1.5d | P2 | pending |
| [Phase 4](./phase-04-dashboard-settings.md) | Dashboard + Settings | 1d | P2 | pending |
| [Phase 5](./phase-05-auth-polish-deploy.md) | Auth + Polish + Deploy (standalone Docker) | 1.5d | P3 | pending |
| **Total** | | **~10d** | | 15% |

---

## Route Structure

```
app/
├── (auth)/
│   ├── layout.tsx              # Centered card, no sidebar
│   ├── login/page.tsx
│   └── forgot-password/page.tsx
├── (main)/
│   ├── layout.tsx              # Sidebar + Header
│   ├── page.tsx                # Dashboard
│   ├── orders/
│   │   ├── page.tsx            # Order list
│   │   ├── [id]/page.tsx       # Order detail
│   │   └── import/page.tsx     # Import wizard
│   ├── design/
│   │   ├── backlog/page.tsx    # Task backlog
│   │   └── workspace/
│   │       ├── page.tsx        # Workspace list
│   │       └── [id]/page.tsx   # Kanban board
│   └── settings/
│       ├── team/page.tsx       # Team management
│       └── workspaces/page.tsx # Workspace config
└── layout.tsx                  # Root layout
```

---

## shadcn Components to Install

```bash
# Foundation
npx shadcn-ui@latest add sidebar
npx shadcn-ui@latest add sheet
npx shadcn-ui@latest add dialog

# Data Display
npx shadcn-ui@latest add data-table
npx shadcn-ui@latest add table
npx shadcn-ui@latest add tabs
npx shadcn-ui@latest add badge
npx shadcn-ui@latest add avatar

# Navigation
npx shadcn-ui@latest add dropdown-menu
npx shadcn-ui@latest add breadcrumb

# Feedback
npx shadcn-ui@latest add progress
npx shadcn-ui@latest add skeleton
npx shadcn-ui@latest add separator
```

---

## Design System Reference

### Glass Morphism CSS
```css
/* Sidebar */
background: rgba(255, 255, 255, 0.6);
backdrop-filter: blur(12px);
border-right: 1px solid rgba(226, 232, 240, 0.8);

/* Tailwind equivalent */
bg-white/60 backdrop-blur-md border-r border-slate-200/80
```

### Phase Colors
| Phase | Color | Tailwind |
|-------|-------|----------|
| Order | Slate 500 | `text-slate-500 bg-slate-100` |
| Design | Amber 500 | `text-amber-500 bg-amber-50` |
| Fulfill | Violet 500 | `text-violet-500 bg-violet-50` |
| Complete | Emerald 500 | `text-emerald-500 bg-emerald-50` |

### Status Colors
| Status | Color | Tailwind |
|--------|-------|----------|
| TODO | Blue 500 | `text-blue-500 bg-blue-50` |
| IN_PROGRESS | Amber 500 | `text-amber-500 bg-amber-50` |
| REVIEW | Violet 500 | `text-violet-500 bg-violet-50` |
| REVISION | Red 500 | `text-red-500 bg-red-50` |
| DONE | Emerald 500 | `text-emerald-500 bg-emerald-50` |

### Priority Colors
| Priority | Color | Tailwind |
|----------|-------|----------|
| URGENT | Red 500 | `text-red-500 bg-red-50` |
| HIGH | Amber 500 | `text-amber-500 bg-amber-50` |
| NORMAL | Blue 500 | `text-blue-500 bg-blue-50` |
| LOW | Gray 500 | `text-gray-500 bg-gray-100` |

---

## Key Decisions

1. **Interactive Kanban** - WITH drag-drop via dnd-kit + status dropdown fallback
2. **Enhanced Mock Data** - Existing + design_versions, timeline events
3. **Route Groups** - `(auth)` vs `(main)` for layout isolation
4. **generateStaticParams** - Pre-render all dynamic routes at build
5. **Docker Standalone** - Node.js server (giữ Dockerfile hiện tại)

---

## Dependencies Graph

```
Phase 1 (Foundation)
    ↓
Phase 2 (Design) ──┬── Phase 3 (Orders)
                   │
                   ↓
            Phase 4 (Dashboard/Settings)
                   │
                   ↓
            Phase 5 (Auth/Deploy)
```

---

## Success Criteria

- [ ] All 12 pages render correctly
- [ ] Glass morphism applied consistently
- [ ] Mock data displays on all pages
- [ ] Navigation works (sidebar, breadcrumbs)
- [ ] Responsive layout (desktop 1280px+)
- [ ] Docker build succeeds
- [ ] Deploy to Coolify works

---

## References

- [UI Spec](../../docs/design/synkao-page-ui-spec.md)
- [Design System](../../docs/design/synkao-design-system.md)
- [Research: shadcn/Glass](./research/researcher-01-shadcn-glass-morphism.md)
- [Research: Next.js Static](./research/researcher-02-nextjs-static-patterns.md)
