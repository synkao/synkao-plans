# Phase 3: Create Tasks Hooks

**Priority:** High
**Status:** ✅ Complete (2026-01-16)
**Depends on:** Phase 2

---

## Context

- [use-auth.ts](../../apps/web/src/hooks/use-auth.ts) - Reference pattern
- [tasks-client.ts](../../apps/web/src/features/design/lib/tasks-client.ts) - Client functions

## Overview

Create React Query hooks with query key factory for proper cache management.

## Implementation

### File: `apps/web/src/features/design/hooks/use-tasks.ts` (NEW)

```typescript
// Task Hooks - React Query queries and mutations for design tasks
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import {
  getTasks,
  getTask,
  bulkAssignWorkspace,
  bulkAssignDesigner,
  bulkToggleUrgent,
  type TaskListParams,
} from '../lib/tasks-client';

// ============== QUERY KEYS ==============

export const taskKeys = {
  all: ['tasks'] as const,
  lists: () => [...taskKeys.all, 'list'] as const,
  list: (filters: TaskListParams) => [...taskKeys.lists(), filters] as const,
  details: () => [...taskKeys.all, 'detail'] as const,
  detail: (id: string) => [...taskKeys.details(), id] as const,
};

// ============== QUERIES ==============

/**
 * Hook to fetch tasks with filters
 */
export function useTasks(params: TaskListParams = {}) {
  return useQuery({
    queryKey: taskKeys.list(params),
    queryFn: () => getTasks(params),
    staleTime: 1000 * 60 * 2, // 2 minutes
  });
}

/**
 * Hook to fetch backlog tasks specifically
 * Convenience wrapper for unassigned TODO tasks
 */
export function useBacklogTasks() {
  return useTasks({
    status: 'TODO',
    workspace_id: 'null',
  });
}

/**
 * Hook to fetch single task by ID
 */
export function useTask(id: string) {
  return useQuery({
    queryKey: taskKeys.detail(id),
    queryFn: () => getTask(id),
    enabled: !!id,
  });
}

// ============== MUTATIONS ==============

/**
 * Hook for bulk assign workspace mutation
 */
export function useBulkAssignWorkspace() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ taskIds, workspaceId }: { taskIds: string[]; workspaceId: string }) =>
      bulkAssignWorkspace(taskIds, workspaceId),
    onSuccess: () => {
      // Invalidate task lists to refetch
      queryClient.invalidateQueries({ queryKey: taskKeys.lists() });
    },
  });
}

/**
 * Hook for bulk assign designer mutation
 */
export function useBulkAssignDesigner() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ taskIds, assigneeId }: { taskIds: string[]; assigneeId: string }) =>
      bulkAssignDesigner(taskIds, assigneeId),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: taskKeys.lists() });
    },
  });
}

/**
 * Hook for bulk toggle urgent mutation
 */
export function useBulkToggleUrgent() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ taskIds, isUrgent }: { taskIds: string[]; isUrgent: boolean }) =>
      bulkToggleUrgent(taskIds, isUrgent),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: taskKeys.lists() });
    },
  });
}
```

### File: `apps/web/src/features/design/hooks/index.ts` (UPDATE)

```typescript
export { useKanbanState } from './use-kanban-state';
export { useTasksByStatus } from './use-tasks-by-status';
export {
  useTasks,
  useBacklogTasks,
  useTask,
  useBulkAssignWorkspace,
  useBulkAssignDesigner,
  useBulkToggleUrgent,
  taskKeys,
} from './use-tasks';
```

### File: `apps/web/src/features/design/index.ts` (UPDATE)

Ensure hooks are re-exported from feature index.

## Todo

- [ ] Create use-tasks.ts with query/mutation hooks
- [ ] Update hooks/index.ts to export new hooks
- [ ] Verify feature index exports hooks
- [ ] Run compile check

## Success Criteria

- Query key factory enables granular cache invalidation
- Mutations invalidate relevant list queries
- TypeScript types flow correctly
