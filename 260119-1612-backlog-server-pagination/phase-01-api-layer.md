# Phase 1: API Layer Updates

## Overview
Add `search` and `source` filter params to tasks API.

## Tasks

- [ ] Update MSW handler to support search & source filters
- [ ] Update TaskListParams type
- [ ] Update buildQueryString helper
- [ ] Update useBacklogTasks hook signature

---

## 1. MSW Handler

**File:** `apps/web/src/mocks/handlers/tasks.ts`

**Changes:** Add search and source param handling after line 25

```ts
// GET /api/v1/design/tasks handler - add after existing param parsing
const search = url.searchParams.get('search');
const source = url.searchParams.get('source');

// Add to filter chain (after existing filters)
if (search) {
  const s = search.toLowerCase();
  filtered = filtered.filter(t =>
    t.taskCode.toLowerCase().includes(s) ||
    t.productName.toLowerCase().includes(s) ||
    t.customerName.toLowerCase().includes(s)
  );
}
if (source) {
  filtered = filtered.filter(t => t.source === source);
}
```

---

## 2. API Client Types

**File:** `apps/web/src/features/design/lib/tasks-client.ts`

**Changes:** Add to TaskListParams interface

```ts
export interface TaskListParams {
  page?: number;
  limit?: number;
  status?: DesignTaskStatusType;
  workspace_id?: string | 'null';
  assignee_id?: string;
  is_urgent?: boolean;
  search?: string;   // NEW
  source?: string;   // NEW
}
```

**Changes:** Update buildQueryString function

```ts
if (params.search) searchParams.set('search', params.search);
if (params.source) searchParams.set('source', params.source);
```

---

## 3. Hook Update

**File:** `apps/web/src/features/design/hooks/use-tasks.ts`

**Changes:** Make useBacklogTasks accept filter params

```ts
/**
 * Hook to fetch backlog tasks with optional filters
 */
export function useBacklogTasks(params: Omit<TaskListParams, 'status'> = {}) {
  return useTasks({
    status: 'TODO',
    ...params,
  });
}
```

---

## Validation

- [ ] API returns filtered results with search param
- [ ] API returns filtered results with source param
- [ ] Pagination works with filters applied
- [ ] Empty search returns all results
