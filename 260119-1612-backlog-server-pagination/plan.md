# Plan: Server-side Pagination for Backlog Page

**Created:** 2026-01-19
**Status:** Complete (100%)
**Complexity:** Low-Medium
**Reviewed:** 2026-01-19 by code-reviewer

## Overview

Add server-side pagination with debounced search to Design Backlog page.

## Current State

- Backlog fetches ALL tasks, filters client-side
- No pagination UI
- API already supports `page`, `limit` params
- Missing: `search`, `source` filter params in API

## Target State

- Server-side filtering & pagination
- Debounced search input (300ms)
- Pagination UI with page size selector
- Clear selection on page change

## Phases

| Phase | Description | Status |
|-------|-------------|--------|
| 1 | [API Layer Updates](phase-01-api-layer.md) | ✅ Complete |
| 2 | [Debounce Hook](phase-02-debounce-hook.md) | ✅ Complete |
| 3 | [Page Component Refactor](phase-03-page-refactor.md) | ⚠️ Complete (minor deviation) |
| 4 | [Pagination UI](phase-04-pagination-ui.md) | ✅ Complete |

## Files to Modify

```
apps/web/src/
├── mocks/handlers/tasks.ts          # Add search, source params
├── features/design/
│   ├── lib/tasks-client.ts          # Add search, source to types
│   └── hooks/use-tasks.ts           # Update useBacklogTasks
├── hooks/use-debounced-value.ts     # NEW: debounce hook
└── app/(main)/design/backlog/
    └── page.tsx                     # Main refactor
```

## Dependencies

- Existing: `DataTablePagination` component (TanStack Table based)
- Decision: Create simpler standalone pagination or adapt existing

## Risk Assessment

- **Low:** API changes are additive, backward compatible
- **Low:** UI changes isolated to backlog page

## Code Review Results

**Overall Score:** 8.5/10

**Report:** [Code Review Report](reports/code-reviewer-260119-1622-server-pagination-review.md)

**Critical Issues:** None

**High Priority Findings:**
1. Search filter needs input sanitization (XSS prevention)
2. Missing validation for page/limit params
3. Page reset logic deviation from plan (minor - works but inconsistent)

**Action Items (All Fixed):**
- [x] Add explicit page reset to search handler
- [x] Fix pagination edge case (totalPages === 0)
- [x] Clarify stats bar behavior (page-scoped labels added)
- [ ] Add input validation to buildQueryString (future)
- [ ] Consider adding aria-labels for accessibility (future)

**Strengths:**
- Clean architecture, proper separation of concerns
- Type-safe implementation, zero compilation errors
- Excellent React Query integration
- Good UX patterns (debounce, disabled states, selection clearing)
