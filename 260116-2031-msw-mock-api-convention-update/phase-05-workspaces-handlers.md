---
phase: 5
title: "Update Workspaces Handlers"
status: pending
effort: 30m
---

# Phase 05: Update Workspaces Handlers

## Overview

Update workspace handlers to use `/api/v1/design/workspaces/*` URLs and new response format. Bulk task operations also move to design module.

## Context

**Convention Reference:** `docs/frontend/mock-api-convention.md` Section 5.2

## Related Code Files

- **Modify:** `apps/web/src/mocks/handlers/workspaces.ts`

## URL Mapping

| Current | New |
|---------|-----|
| `GET /api/workspaces` | `GET /api/v1/design/workspaces` |
| `POST /api/workspaces` | `POST /api/v1/design/workspaces` |
| `POST /api/tasks/bulk/assign-workspace` | `POST /api/v1/design/tasks/bulk/assign-workspace` |
| `POST /api/tasks/bulk/assign-designer` | `POST /api/v1/design/tasks/bulk/assign-designer` |
| `POST /api/tasks/bulk/toggle-urgent` | `POST /api/v1/design/tasks/bulk/toggle-urgent` |

## Implementation Steps

### 1. Update imports

```typescript
import { http } from 'msw';
import { mockWorkspaces, mockTasks } from '../data';
import {
  successResponse,
  createdResponse,
  listResponse,
  bulkActionResponse,
  parsePagination,
  getPaginationMeta,
  paginateArray,
  errors,
} from '../utils';
```

### 2. Update list workspaces handler

```typescript
// GET /api/v1/design/workspaces
http.get('/api/v1/design/workspaces', ({ request }) => {
  const url = new URL(request.url);
  const { page, limit } = parsePagination(url);

  const paginated = paginateArray(mockWorkspaces, page, limit);
  const pagination = getPaginationMeta(mockWorkspaces.length, page, limit);

  return listResponse(paginated, pagination);
}),
```

### 3. Update create workspace handler

```typescript
// POST /api/v1/design/workspaces
http.post('/api/v1/design/workspaces', async ({ request }) => {
  const body = await request.json() as {
    name: string;
    ownerId: string;
    description?: string;
    color?: string;
  };

  const newWorkspace = {
    id: `ws_${Date.now()}`,
    tenantId: 'tenant_1',
    name: body.name,
    ownerId: body.ownerId,
    memberIds: [body.ownerId],
    description: body.description,
    color: body.color,
    isDefault: false,
    sortOrder: mockWorkspaces.length + 1,
  };

  mockWorkspaces.push(newWorkspace);
  return createdResponse(newWorkspace);
}),
```

### 4. Update bulk assign workspace handler

```typescript
// POST /api/v1/design/tasks/bulk/assign-workspace
http.post('/api/v1/design/tasks/bulk/assign-workspace', async ({ request }) => {
  const body = await request.json() as {
    task_ids: string[];
    workspace_id: string;
  };

  const { task_ids, workspace_id } = body;

  // Validate workspace
  const workspace = mockWorkspaces.find((w) => w.id === workspace_id);
  if (!workspace) {
    return errors.notFound('Workspace');
  }

  // Validate tasks
  const tasks = mockTasks.filter((t) => task_ids.includes(t.id));
  if (tasks.length !== task_ids.length) {
    return errors.notFound('Task');
  }

  // Update tasks
  tasks.forEach(task => {
    task.workspaceId = workspace_id;
    task.updatedAt = new Date().toISOString();
  });

  return bulkActionResponse(
    `${tasks.length} tasks assigned to workspace successfully`,
    tasks.length,
    tasks.map(t => t.id)
  );
}),
```

### 5. Update bulk assign designer handler

```typescript
// POST /api/v1/design/tasks/bulk/assign-designer
http.post('/api/v1/design/tasks/bulk/assign-designer', async ({ request }) => {
  const body = await request.json() as {
    task_ids: string[];
    assignee_id: string;
  };

  const { task_ids, assignee_id } = body;

  const tasks = mockTasks.filter((t) => task_ids.includes(t.id));
  if (tasks.length !== task_ids.length) {
    return errors.notFound('Task');
  }

  tasks.forEach(task => {
    task.assigneeId = assignee_id;
    task.assignedAt = new Date().toISOString();
    task.updatedAt = new Date().toISOString();
  });

  return bulkActionResponse(
    `${tasks.length} tasks assigned successfully`,
    tasks.length,
    tasks.map(t => t.id)
  );
}),
```

### 6. Update bulk toggle urgent handler

```typescript
// POST /api/v1/design/tasks/bulk/toggle-urgent
http.post('/api/v1/design/tasks/bulk/toggle-urgent', async ({ request }) => {
  const body = await request.json() as {
    task_ids: string[];
    is_urgent: boolean;
  };

  const { task_ids, is_urgent } = body;

  const tasks = mockTasks.filter((t) => task_ids.includes(t.id));
  if (tasks.length !== task_ids.length) {
    return errors.notFound('Task');
  }

  tasks.forEach(task => {
    task.isUrgent = is_urgent;
    task.updatedAt = new Date().toISOString();
  });

  return bulkActionResponse(
    `${tasks.length} tasks updated successfully`,
    tasks.length,
    tasks.map(t => t.id)
  );
}),
```

## Todo List

- [ ] Import response helpers
- [ ] Update workspace URLs to `/api/v1/design/workspaces/*`
- [ ] Update bulk task URLs to `/api/v1/design/tasks/bulk/*`
- [ ] Add pagination to list
- [ ] Use snake_case for request body keys (task_ids, workspace_id)
- [ ] Use bulkActionResponse for bulk operations
- [ ] Use error helpers

## Success Criteria

- [ ] All workspace endpoints use `/api/v1/design/workspaces/*`
- [ ] Bulk task endpoints use `/api/v1/design/tasks/bulk/*`
- [ ] List returns pagination
- [ ] Bulk actions return `{ message, data: { affected_count, affected_ids } }`
- [ ] Error responses have proper codes
