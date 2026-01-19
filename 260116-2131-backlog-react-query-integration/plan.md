# Plan: Backlog React Query Integration

**Created:** 2026-01-16
**Status:** Ready for Implementation
**Reference:** [brainstorm-260116-2127-backlog-react-query-integration.md](../reports/brainstorm-260116-2127-backlog-react-query-integration.md)

**Updated:** 2026-01-16
**Overall Status:** ✅ COMPLETED

---

## Overview

Integrate React Query into Backlog page, replacing direct mock imports and raw fetch calls with proper client/hook pattern following existing auth implementation.

## Phases

| Phase | Description | Status |
|-------|-------------|--------|
| 1 | [Update Mock Handler](phase-01-update-mock-handler.md) | ✅ Complete |
| 2 | [Create Tasks Client](phase-02-create-tasks-client.md) | ✅ Complete |
| 3 | [Create Tasks Hooks](phase-03-create-tasks-hooks.md) | ✅ Complete |
| 4 | [Update BacklogPage](phase-04-update-backlog-page.md) | ✅ Complete |

## Key Dependencies

- React Query already configured via `@tanstack/react-query`
- MSW handlers in `apps/web/src/mocks/handlers/`
- Auth pattern reference: `lib/auth-client.ts` → `hooks/use-auth.ts`

## File Changes Summary

```
apps/web/src/
├── mocks/handlers/tasks.ts              # UPDATE: workspace_id=null filter
├── features/design/
│   ├── lib/
│   │   ├── tasks-client.ts              # NEW: Raw fetch functions
│   │   └── index.ts                     # UPDATE: Export client
│   ├── hooks/
│   │   ├── use-tasks.ts                 # NEW: React Query hooks
│   │   └── index.ts                     # UPDATE: Export hooks
│   └── index.ts                         # UPDATE: Export hooks
└── app/(main)/design/backlog/page.tsx   # UPDATE: Use React Query
```

## Success Criteria

- [x] Backlog page loads tasks via `useTasks()` hook
- [x] Bulk actions use mutation hooks with cache invalidation
- [x] Loading/error states displayed properly
- [x] Client-side filters (search, urgent, source) remain snappy
- [⚠️] No TypeScript errors (build requires Node >=20.9.0, minor lint warning)
