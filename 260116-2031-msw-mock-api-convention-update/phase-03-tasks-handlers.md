---
phase: 3
title: "Update Tasks Handlers"
status: pending
effort: 45m
---

# Phase 03: Update Tasks Handlers

## Overview

Update task handlers to use `/api/v1/design/tasks/*` URLs, new response format, and add missing action endpoints.

## Context

**Convention Reference:** `docs/frontend/mock-api-convention.md` Section 5.2

## Related Code Files

- **Modify:** `apps/web/src/mocks/handlers/tasks.ts`

## URL Mapping

| Current | New |
|---------|-----|
| `GET /api/tasks` | `GET /api/v1/design/tasks` |
| `GET /api/tasks/:id` | `GET /api/v1/design/tasks/:id` |
| `PATCH /api/tasks/:id` | `PATCH /api/v1/design/tasks/:id` |
| `POST /api/tasks/:id/designs` | `POST /api/v1/design/tasks/:id/files` |
| `GET /api/tasks/:id/designs` | `GET /api/v1/design/tasks/:id/files` |
| `GET /api/workspaces/:id/board` | `GET /api/v1/design/workspaces/:id/board` |
| `GET /api/columns` | `GET /api/v1/design/columns` |
| (missing) | `PATCH /api/v1/design/tasks/:id/status` |
| (missing) | `POST /api/v1/design/tasks/:id/assign` |
| (missing) | `POST /api/v1/design/tasks/:id/approve` |
| (missing) | `POST /api/v1/design/tasks/:id/reject` |

## Implementation Steps

### 1. Update imports

```typescript
import { http } from 'msw';
import type { MockTask, DesignTaskStatusType } from '../types';
import { DesignTaskStatus } from '../types';
import { mockTasks, findTask, getTasksByStatus, findDesignVersions, mockColumns } from '../data';
import { COLUMN_IDS } from '../factories/mock-factory';
import {
  successResponse,
  listResponse,
  actionResponse,
  parsePagination,
  getPaginationMeta,
  paginateArray,
  errors,
} from '../utils';
```

### 2. Update list tasks handler

```typescript
// GET /api/v1/design/tasks
http.get('/api/v1/design/tasks', ({ request }) => {
  const url = new URL(request.url);
  const { page, limit } = parsePagination(url);
  const status = url.searchParams.get('status') as DesignTaskStatusType | null;
  const workspaceId = url.searchParams.get('workspace_id');
  const assigneeId = url.searchParams.get('assignee_id');
  const isUrgent = url.searchParams.get('is_urgent');

  let filtered = mockTasks;

  if (status) filtered = filtered.filter(t => t.status === status);
  if (workspaceId) filtered = filtered.filter(t => t.workspaceId === workspaceId);
  if (assigneeId) filtered = filtered.filter(t => t.assigneeId === assigneeId);
  if (isUrgent !== null) filtered = filtered.filter(t => t.isUrgent === (isUrgent === 'true'));

  const paginated = paginateArray(filtered, page, limit);
  const pagination = getPaginationMeta(filtered.length, page, limit);

  return listResponse(paginated, pagination);
}),
```

### 3. Update get task handler

```typescript
// GET /api/v1/design/tasks/:id
http.get('/api/v1/design/tasks/:id', ({ params }) => {
  const task = findTask(params.id as string);
  if (!task) {
    return errors.notFound('Task');
  }
  return successResponse(task);
}),
```

### 4. Update patch task handler

```typescript
// PATCH /api/v1/design/tasks/:id
http.patch('/api/v1/design/tasks/:id', async ({ params, request }) => {
  const body = await request.json() as Partial<MockTask>;
  const task = findTask(params.id as string);

  if (!task) {
    return errors.notFound('Task');
  }

  const updatedTask = { ...task, ...body, updatedAt: new Date().toISOString() };
  return successResponse(updatedTask);
}),
```

### 5. Add status update handler (NEW)

```typescript
// PATCH /api/v1/design/tasks/:id/status
http.patch('/api/v1/design/tasks/:id/status', async ({ params, request }) => {
  const body = await request.json() as { status: DesignTaskStatusType };
  const task = findTask(params.id as string);

  if (!task) {
    return errors.notFound('Task');
  }

  // Validate status transition (simplified)
  if (task.status === DesignTaskStatus.APPROVED && body.status !== DesignTaskStatus.APPROVED) {
    return errors.businessRule('TASK_CANNOT_TRANSITION', 'Cannot change status of approved task');
  }

  const updatedTask = { ...task, status: body.status, updatedAt: new Date().toISOString() };
  return successResponse(updatedTask);
}),
```

### 6. Add assign handler (NEW)

```typescript
// POST /api/v1/design/tasks/:id/assign
http.post('/api/v1/design/tasks/:id/assign', async ({ params, request }) => {
  const body = await request.json() as { assignee_id: string };
  const task = findTask(params.id as string);

  if (!task) {
    return errors.notFound('Task');
  }

  return actionResponse('Task assigned successfully');
}),
```

### 7. Add approve handler (NEW)

```typescript
// POST /api/v1/design/tasks/:id/approve
http.post('/api/v1/design/tasks/:id/approve', async ({ params, request }) => {
  const body = await request.json() as { points?: number };
  const task = findTask(params.id as string);

  if (!task) {
    return errors.notFound('Task');
  }

  if (task.status === DesignTaskStatus.APPROVED) {
    return errors.businessRule('TASK_ALREADY_APPROVED', 'Task already approved');
  }

  return actionResponse('Task approved successfully');
}),
```

### 8. Add reject handler (NEW)

```typescript
// POST /api/v1/design/tasks/:id/reject
http.post('/api/v1/design/tasks/:id/reject', async ({ params, request }) => {
  const body = await request.json() as { reason: string };
  const task = findTask(params.id as string);

  if (!task) {
    return errors.notFound('Task');
  }

  return actionResponse('Task rejected successfully');
}),
```

### 9. Update files handlers

```typescript
// POST /api/v1/design/tasks/:id/files
http.post('/api/v1/design/tasks/:id/files', ({ params }) => {
  const task = findTask(params.id as string);
  if (!task) {
    return errors.notFound('Task');
  }

  return successResponse({
    designUrl: `https://cdn.synkao.com/designs/${task.taskCode.toLowerCase()}-new.png`,
    uploadedAt: new Date().toISOString(),
  }, 201);
}),

// GET /api/v1/design/tasks/:id/files
http.get('/api/v1/design/tasks/:id/files', ({ params }) => {
  const versions = findDesignVersions(params.id as string);
  return successResponse({ versions, total: versions.length });
}),
```

### 10. Update board and columns handlers

```typescript
// GET /api/v1/design/workspaces/:id/board
http.get('/api/v1/design/workspaces/:id/board', () => {
  return successResponse({
    columns: [
      { id: COLUMN_IDS.todo, name: 'TODO', tasks: getTasksByStatus(DesignTaskStatus.TODO) },
      { id: COLUMN_IDS.inProgress, name: 'IN PROGRESS', tasks: getTasksByStatus(DesignTaskStatus.IN_PROGRESS) },
      { id: COLUMN_IDS.review, name: 'REVIEW', tasks: getTasksByStatus(DesignTaskStatus.REVIEW) },
      { id: COLUMN_IDS.revision, name: 'REVISION', tasks: getTasksByStatus(DesignTaskStatus.REVISION) },
      { id: COLUMN_IDS.approved, name: 'APPROVED', tasks: getTasksByStatus(DesignTaskStatus.APPROVED) },
    ],
  });
}),

// GET /api/v1/design/columns
http.get('/api/v1/design/columns', () => {
  return successResponse({ columns: mockColumns });
}),
```

## Todo List

- [ ] Import response helpers
- [ ] Update all URLs to `/api/v1/design/tasks/*`
- [ ] Add pagination to list endpoint
- [ ] Add filtering (workspace_id, assignee_id, is_urgent)
- [ ] Add PATCH /:id/status handler
- [ ] Add POST /:id/assign handler
- [ ] Add POST /:id/approve handler
- [ ] Add POST /:id/reject handler
- [ ] Wrap responses in `{ data: ... }`
- [ ] Use error helpers

## Success Criteria

- [ ] All task endpoints use `/api/v1/design/tasks/*`
- [ ] List returns pagination
- [ ] All action endpoints return `{ message: "..." }`
- [ ] Error responses have proper codes
