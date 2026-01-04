# Phase 4: Verification (~45min)

## Context

- Plan: [plan.md](./plan.md)
- Depends on: [Phase 3](./phase-03-msw-handlers.md)
- Issue: #64

## Overview

Run typecheck, build, and manual verification to ensure MSW mock data updates work correctly.

## Requirements

1. Pass `pnpm typecheck`
2. Pass `pnpm build`
3. Verify existing features still work (Kanban, backlog)
4. Verify new data displays correctly

## Verification Checklist

### 1. Type Check

```bash
cd apps/web
pnpm typecheck
```

Expected: No errors

### 2. Build

```bash
pnpm build
```

Expected: Build succeeds

### 3. Dev Server Test

```bash
pnpm dev
```

Navigate and verify:

| Route | Check |
|-------|-------|
| `/design/workspace` | Kanban loads with tasks |
| `/design/backlog` | Backlog table loads |
| `/orders` | Orders list loads |

### 4. Console Check

Open DevTools console, verify:
- No MSW errors
- No type errors at runtime
- Network tab shows mocked API calls

### 5. Data Integrity Check

Verify in Network tab:
- `GET /api/tasks` returns new status values (TODO, IN_PROGRESS, etc.)
- `GET /api/orders` returns new status values (PENDING, CONFIRMED, etc.)
- Response shapes match expected format

## Common Issues & Fixes

| Issue | Fix |
|-------|-----|
| Import errors | Check export names in index.ts |
| Type mismatch | Update component props to use new types |
| Status not recognized | Update Kanban column mapping |
| Missing columnId | Ensure tasks.data.ts assigns columnId |

## Rollback Plan

If issues found:
1. Revert to previous types.ts
2. Keep enums in separate file
3. Gradual migration

## Todo List

- [ ] Run `pnpm typecheck`
- [ ] Fix any type errors
- [ ] Run `pnpm build`
- [ ] Fix any build errors
- [ ] Start dev server
- [ ] Test Kanban board loads
- [ ] Test backlog table loads
- [ ] Test orders page loads
- [ ] Check console for errors
- [ ] Verify API response shapes

## Success Criteria

- [ ] `pnpm typecheck` passes with 0 errors
- [ ] `pnpm build` succeeds
- [ ] Kanban board renders tasks correctly
- [ ] Backlog table shows tasks
- [ ] Orders page shows orders
- [ ] No console errors related to MSW or types
- [ ] Network responses use new spec values
