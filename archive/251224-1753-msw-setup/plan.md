---
title: "Setup Mock Service Worker (MSW)"
description: "Cấu hình MSW 2.x cho API mocking trong Next.js App Router"
status: done
priority: P0
effort: 3h
issue: 3
branch: main
tags: [frontend, setup, msw, testing]
created: 2025-12-24
---

# Setup Mock Service Worker (MSW)

## Overview

Setup MSW 2.x để mock API endpoints cho development. Cho phép frontend development độc lập với backend, test các edge cases và error states.

## Context

- **Issue:** #3
- **Depends on:** Issue #1 (Next.js setup) ✅, Issue #2 (shadcn/ui) ✅
- **Related:** Issue #4 (Mock data structure)
- **Working dir:** `apps/web/`

## Phases

| # | Phase | Status | Effort | Link |
|---|-------|--------|--------|------|
| 1 | Install MSW | ✅ Done | 30min | [phase-01](./phase-01-install-msw.md) |
| 2 | Setup Handlers | ✅ Done | 1.5h | [phase-02-setup-handlers.md](./phase-02-setup-handlers.md) |
| 3 | Next.js Integration | ✅ Done | 1h | [phase-03-nextjs-integration.md](./phase-03-nextjs-integration.md) |

## API Endpoints to Mock

### Auth
- `POST /api/auth/login` - Login
- `POST /api/auth/logout` - Logout
- `POST /api/auth/refresh` - Refresh token
- `GET /api/auth/me` - Current user

### Tasks
- `GET /api/tasks` - List with filters
- `GET /api/tasks/:id` - Task detail
- `PATCH /api/tasks/:id` - Update task
- `POST /api/tasks/:id/designs` - Upload design
- `GET /api/workspaces/:id/board` - Kanban board data

### Orders
- `GET /api/orders` - List with filters
- `GET /api/orders/:id` - Order detail
- `POST /api/orders/import` - Import CSV

## File Structure

```
apps/web/src/
├── mocks/
│   ├── browser.ts          # Browser worker setup
│   ├── server.ts           # Node server setup (SSR)
│   ├── handlers/
│   │   ├── index.ts        # Export all handlers
│   │   ├── auth.ts         # Auth handlers
│   │   ├── tasks.ts        # Tasks handlers
│   │   └── orders.ts       # Orders handlers
│   └── data/               # (Issue #4)
└── app/
    └── providers.tsx       # MSW provider
```

## Key Decisions

1. **MSW Version:** 2.x (breaking changes from 1.x)
2. **Setup type:** Browser + Node (SSR support)
3. **Handler organization:** By domain (auth, tasks, orders)
4. **Integration:** Via providers.tsx với dynamic import

## Success Criteria

- [x] MSW 2.x installed
- [x] Service worker generated (`mockServiceWorker.js`)
- [x] Basic handlers respond correctly
- [x] Works với Next.js App Router (client + SSR)
- [x] Dev server starts without errors

## Completed: 2025-12-24

**Commits:**
- `15a2a1e` - Phase 01: Install MSW
- `1543e9b` - Phase 02: Setup Handlers
- `d2de29c` - Phase 03: Next.js Integration
