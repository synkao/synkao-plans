# Phase 2: Create Tasks Client

**Priority:** High
**Status:** ✅ Complete (2026-01-16)
**Depends on:** Phase 1

---

## Context

- [auth-client.ts](../../apps/web/src/lib/auth-client.ts) - Reference pattern
- [workspaces.ts](../../apps/web/src/mocks/handlers/workspaces.ts) - Bulk endpoints

## Overview

Create raw fetch functions for task API following auth-client pattern.

## Implementation

### File: `apps/web/src/features/design/lib/tasks-client.ts` (NEW)

```typescript
// Tasks API Client - HTTP functions for design tasks
import type { MockTask, DesignTaskStatusType } from '@/mocks/types';

const API_BASE = '/api/v1';

// ============== TYPES ==============

export interface TaskListParams {
  page?: number;
  limit?: number;
  status?: DesignTaskStatusType;
  workspace_id?: string | 'null';  // 'null' = unassigned
  assignee_id?: string;
  is_urgent?: boolean;
}

export interface TaskListResponse {
  data: MockTask[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

export interface BulkActionResponse {
  message: string;
  affected_count: number;
  affected_ids: string[];
}

// ============== HELPERS ==============

function buildQueryString(params: TaskListParams): string {
  const searchParams = new URLSearchParams();

  if (params.page) searchParams.set('page', String(params.page));
  if (params.limit) searchParams.set('limit', String(params.limit));
  if (params.status) searchParams.set('status', params.status);
  if (params.workspace_id !== undefined) searchParams.set('workspace_id', params.workspace_id);
  if (params.assignee_id) searchParams.set('assignee_id', params.assignee_id);
  if (params.is_urgent !== undefined) searchParams.set('is_urgent', String(params.is_urgent));

  const qs = searchParams.toString();
  return qs ? `?${qs}` : '';
}

// ============== API FUNCTIONS ==============

// GET /api/v1/design/tasks
export async function getTasks(params: TaskListParams = {}): Promise<TaskListResponse> {
  const response = await fetch(`${API_BASE}/design/tasks${buildQueryString(params)}`);

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Failed to fetch tasks');
  }

  return response.json();
}

// GET /api/v1/design/tasks/:id
export async function getTask(id: string): Promise<MockTask> {
  const response = await fetch(`${API_BASE}/design/tasks/${id}`);

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Failed to fetch task');
  }

  const result = await response.json();
  return result.data;
}

// POST /api/v1/design/tasks/bulk/assign-workspace
export async function bulkAssignWorkspace(
  taskIds: string[],
  workspaceId: string
): Promise<BulkActionResponse> {
  const response = await fetch(`${API_BASE}/design/tasks/bulk/assign-workspace`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ task_ids: taskIds, workspace_id: workspaceId }),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Failed to assign workspace');
  }

  const result = await response.json();
  return result.data;
}

// POST /api/v1/design/tasks/bulk/assign-designer
export async function bulkAssignDesigner(
  taskIds: string[],
  assigneeId: string
): Promise<BulkActionResponse> {
  const response = await fetch(`${API_BASE}/design/tasks/bulk/assign-designer`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ task_ids: taskIds, assignee_id: assigneeId }),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Failed to assign designer');
  }

  const result = await response.json();
  return result.data;
}

// POST /api/v1/design/tasks/bulk/toggle-urgent
export async function bulkToggleUrgent(
  taskIds: string[],
  isUrgent: boolean
): Promise<BulkActionResponse> {
  const response = await fetch(`${API_BASE}/design/tasks/bulk/toggle-urgent`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ task_ids: taskIds, is_urgent: isUrgent }),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Failed to toggle urgent');
  }

  const result = await response.json();
  return result.data;
}
```

### File: `apps/web/src/features/design/lib/index.ts` (UPDATE)

```typescript
export * from './constants';
export * from './utils';
export * from './tasks-client';
```

## Todo

- [ ] Create tasks-client.ts with fetch functions
- [ ] Update lib/index.ts to export client
- [ ] Run compile check

## Success Criteria

- All fetch functions typed and exportable
- No TypeScript errors
