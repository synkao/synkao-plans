---
title: "MSW Mock API Convention Update"
description: "Update MSW handlers to follow /api/v1/{module}/ URL convention with proper response format"
status: completed
priority: P2
effort: 3h
branch: main
tags: [msw, mock-api, convention, frontend]
created: 2026-01-16
completed: 2026-01-16
---

# MSW Mock API Convention Update

## Overview

Update all MSW mock API handlers to follow conventions in `docs/frontend/mock-api-convention.md`:
- URL: `/api/v1/{module}/{resource}`
- Response: `{ data: ... }` wrapper
- Pagination: `{ data: [...], pagination: {...} }`
- Error: `{ error: { code, message } }`

## Current State Analysis

### Issues Found

| Category | Current | Expected |
|----------|---------|----------|
| Auth URL | `/api/auth/*` | `/api/v1/auth/*` |
| Tasks URL | `/api/tasks` | `/api/v1/design/tasks` |
| Orders URL | `/api/orders` | `/api/v1/orders/` |
| Workspaces URL | `/api/workspaces` | `/api/v1/design/workspaces` |
| Users URL | `/api/users` | `/api/v1/users/` |
| Response (single) | `{ ...entity }` | `{ data: {...} }` |
| Response (list) | `{ tasks: [], total }` | `{ data: [], pagination }` |
| Error response | `{ error: "string" }` | `{ error: { code, message } }` |

### Files to Modify

**Handlers:**
- `apps/web/src/mocks/handlers/auth.ts`
- `apps/web/src/mocks/handlers/tasks.ts`
- `apps/web/src/mocks/handlers/orders.ts`
- `apps/web/src/mocks/handlers/workspaces.ts`

**Frontend API Calls:**
- `apps/web/src/lib/auth-client.ts`
- `apps/web/src/app/(main)/design/backlog/page.tsx`
- `apps/web/src/app/dev-tools/page.tsx`
- `apps/web/src/features/design/components/workspace/create-workspace-modal.tsx`

### Files to Create

- `apps/web/src/mocks/utils/response-helpers.ts` - DRY response builders

## Phases

| Phase | Status | Description |
|-------|--------|-------------|
| [Phase 01](./phase-01-response-helpers.md) | ✅ completed | Create response helper utilities |
| [Phase 02](./phase-02-auth-handlers.md) | ✅ completed | Update auth handlers |
| [Phase 03](./phase-03-tasks-handlers.md) | ✅ completed | Update tasks handlers |
| [Phase 04](./phase-04-orders-handlers.md) | ✅ completed | Update orders handlers |
| [Phase 05](./phase-05-workspaces-handlers.md) | ✅ completed | Update workspaces handlers |
| [Phase 06](./phase-06-frontend-api-calls.md) | ✅ completed | Update frontend API calls |

## Success Criteria

- [x] All handlers use `/api/v1/{module}/{resource}` URL pattern
- [x] All responses wrapped in `{ data: ... }` format
- [x] List endpoints return pagination object
- [x] Error responses use `{ error: { code, message } }` format
- [x] Missing endpoints added (status update, assign, approve, reject)
- [x] Frontend API calls updated to match new URLs
- [x] No TypeScript compilation errors

## Dependencies

- Convention doc: `docs/frontend/mock-api-convention.md`
- MSW v2 API: Uses `http.get()`, `HttpResponse.json()`
