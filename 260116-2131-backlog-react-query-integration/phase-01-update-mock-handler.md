# Phase 1: Update Mock Handler

**Priority:** High
**Status:** ✅ Complete (2026-01-16)

---

## Context

- [tasks.ts](../../apps/web/src/mocks/handlers/tasks.ts) - Current handler
- [brainstorm report](../reports/brainstorm-260116-2127-backlog-react-query-integration.md)

## Overview

Add support for `workspace_id=null` filter to return unassigned tasks (backlog).

## Current Behavior

```typescript
// Line 23-24 in tasks.ts
const workspaceId = url.searchParams.get('workspace_id');
if (workspaceId) filtered = filtered.filter((t) => t.workspaceId === workspaceId);
```

Currently only filters when `workspaceId` has a truthy value. Need to handle explicit `null` case.

## Implementation

### File: `apps/web/src/mocks/handlers/tasks.ts`

Update workspace_id filter logic (around line 30):

```typescript
// Handle workspace_id filter:
// - ?workspace_id=ws_123 → filter by specific workspace
// - ?workspace_id=null → filter unassigned tasks (no workspace)
// - no param → return all tasks
if (workspaceId === 'null') {
  filtered = filtered.filter((t) => !t.workspaceId);
} else if (workspaceId) {
  filtered = filtered.filter((t) => t.workspaceId === workspaceId);
}
```

## Todo

- [ ] Update workspace_id filter in GET /api/v1/design/tasks handler
- [ ] Test with `?workspace_id=null&status=TODO` params

## Success Criteria

- `GET /api/v1/design/tasks?workspace_id=null&status=TODO` returns only unassigned TODO tasks
