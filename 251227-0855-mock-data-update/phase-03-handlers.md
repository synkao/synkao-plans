---
title: "Phase 03: Handler CRUD & Validation"
status: pending
priority: P1
effort: 1.5h
---

# Phase 03: Handler CRUD & Validation

**Parent:** [plan.md](./plan.md)
**Dependencies:** Phase 02 (data files)

## Overview

Update handlers với full CRUD, state machine validation, optimistic locking, multi-tenant filtering.

## Key Insights (từ Research)

- TypeScript generics cho type-safe handlers: `http.get<Params, Body, Response>`
- Optimistic locking: version field + 412 Precondition Failed
- Multi-tenant: require x-tenant-id header, filter at entry
- State machine: explicit transition rules

## Requirements

1. UPDATE: tasks.ts - Kanban operations, status transitions, optimistic lock
2. UPDATE: orders.ts - Status transitions, fulfillment updates
3. NEW: workspaces.ts - Workspace + Column CRUD
4. NEW: designs.ts - Upload/review flow

## Related Files

- `apps/web/src/mocks/handlers/*.ts`
- `apps/web/src/mocks/handlers/index.ts`

## State Transition Rules

### Order Status Transitions

```typescript
const ORDER_TRANSITIONS: Record<OrderStatus, OrderStatus[]> = {
  PENDING: ['CONFIRMED', 'CANCELLED'],
  CONFIRMED: ['IN_DESIGN', 'READY_TO_FULFILL', 'CANCELLED'],
  IN_DESIGN: ['READY_TO_FULFILL', 'CANCELLED'],
  READY_TO_FULFILL: ['SHIPPED', 'CANCELLED'],
  SHIPPED: ['COMPLETED'],
  COMPLETED: [],
  CANCELLED: [],
};
```

### Task Status Transitions

```typescript
const TASK_TRANSITIONS: Record<DesignTaskStatus, DesignTaskStatus[]> = {
  PENDING: ['TODO', 'BLOCKED'],
  TODO: ['IN_PROGRESS', 'BLOCKED'],
  IN_PROGRESS: ['REVIEW', 'BLOCKED'],
  REVIEW: ['APPROVED', 'REVISION'],
  REVISION: ['IN_PROGRESS', 'BLOCKED'],
  APPROVED: [],
  BLOCKED: ['TODO'],
};
```

### Design Status Transitions

```typescript
const DESIGN_TRANSITIONS: Record<DesignStatus, DesignStatus[]> = {
  DRAFT: ['SUBMITTED'],
  SUBMITTED: ['APPROVED', 'REJECTED'],
  APPROVED: [],
  REJECTED: [],
};
```

## Implementation Steps

### Step 3.1: Create shared handler utilities

```typescript
// apps/web/src/mocks/handlers/utils.ts
import { HttpResponse } from 'msw';
import type { OrderStatus, DesignTaskStatus, DesignStatus } from '../types';

// === STATE MACHINE VALIDATORS ===

export const ORDER_TRANSITIONS: Record<OrderStatus, OrderStatus[]> = {
  PENDING: ['CONFIRMED', 'CANCELLED'],
  CONFIRMED: ['IN_DESIGN', 'READY_TO_FULFILL', 'CANCELLED'],
  IN_DESIGN: ['READY_TO_FULFILL', 'CANCELLED'],
  READY_TO_FULFILL: ['SHIPPED', 'CANCELLED'],
  SHIPPED: ['COMPLETED'],
  COMPLETED: [],
  CANCELLED: [],
};

export const TASK_TRANSITIONS: Record<DesignTaskStatus, DesignTaskStatus[]> = {
  PENDING: ['TODO', 'BLOCKED'],
  TODO: ['IN_PROGRESS', 'BLOCKED'],
  IN_PROGRESS: ['REVIEW', 'BLOCKED'],
  REVIEW: ['APPROVED', 'REVISION'],
  REVISION: ['IN_PROGRESS', 'BLOCKED'],
  APPROVED: [],
  BLOCKED: ['TODO'],
};

export const DESIGN_TRANSITIONS: Record<DesignStatus, DesignStatus[]> = {
  DRAFT: ['SUBMITTED'],
  SUBMITTED: ['APPROVED', 'REJECTED'],
  APPROVED: [],
  REJECTED: [],
};

export function canTransition<T extends string>(
  transitions: Record<T, T[]>,
  from: T,
  to: T
): boolean {
  return transitions[from]?.includes(to) ?? false;
}

// === ERROR RESPONSES ===

export const notFound = (entity: string) =>
  HttpResponse.json({ error: `${entity} not found` }, { status: 404 });

export const badRequest = (message: string) =>
  HttpResponse.json({ error: message }, { status: 400 });

export const conflict = (currentVersion: number, yourVersion: number) =>
  HttpResponse.json(
    {
      error: 'CONFLICT',
      message: 'Resource was modified by another user',
      currentVersion,
      yourVersion,
    },
    { status: 412 }
  );

export const invalidTransition = (from: string, to: string) =>
  HttpResponse.json(
    { error: `Invalid status transition from ${from} to ${to}` },
    { status: 400 }
  );

// === TENANT HELPERS ===

export const getTenantId = (request: Request): string | null =>
  request.headers.get('x-tenant-id');

export const requireTenant = (request: Request): string => {
  const tenantId = getTenantId(request);
  if (!tenantId) throw new Error('Missing x-tenant-id header');
  return tenantId;
};

// === TIMESTAMP HELPERS ===

export const now = () => new Date().toISOString();
```

### Step 3.2: Update tasks.ts handler

```typescript
// apps/web/src/mocks/handlers/tasks.ts
import { http, HttpResponse } from 'msw';
import type { MockDesignTask, DesignTaskStatus } from '../types';
import {
  mockTasks,
  findTask,
  getTasksByStatus,
  getTasksByWorkspace,
  getBacklogTasks,
  getTasksByColumn,
  findDesignVersions,
  mockColumns,
} from '../data';
import {
  notFound,
  badRequest,
  conflict,
  invalidTransition,
  canTransition,
  TASK_TRANSITIONS,
  now,
} from './utils';

// In-memory state (mutable for mock purposes)
let tasks = [...mockTasks];

export const tasksHandlers = [
  // GET /api/tasks - List with filters
  http.get('/api/tasks', ({ request }) => {
    const url = new URL(request.url);
    const status = url.searchParams.get('status') as DesignTaskStatus | null;
    const workspaceId = url.searchParams.get('workspaceId');
    const assigneeId = url.searchParams.get('assigneeId');
    const backlog = url.searchParams.get('backlog') === 'true';

    let filtered = [...tasks];

    if (backlog) {
      filtered = filtered.filter((t) => !t.workspaceId);
    } else if (workspaceId) {
      filtered = filtered.filter((t) => t.workspaceId === workspaceId);
    }

    if (status) {
      filtered = filtered.filter((t) => t.status === status);
    }

    if (assigneeId) {
      filtered = filtered.filter((t) => t.assigneeId === assigneeId);
    }

    return HttpResponse.json({ tasks: filtered, total: filtered.length });
  }),

  // GET /api/tasks/:id
  http.get('/api/tasks/:id', ({ params }) => {
    const task = tasks.find((t) => t.id === params.id);
    if (!task) return notFound('Task');
    return HttpResponse.json(task);
  }),

  // PATCH /api/tasks/:id - Update with optimistic lock
  http.patch('/api/tasks/:id', async ({ params, request }) => {
    const body = (await request.json()) as Partial<MockDesignTask> & {
      version?: number;
    };
    const taskIndex = tasks.findIndex((t) => t.id === params.id);

    if (taskIndex === -1) return notFound('Task');

    const task = tasks[taskIndex]!;

    // Optimistic locking check
    if (body.version !== undefined && body.version !== task.version) {
      return conflict(task.version, body.version);
    }

    // Status transition validation
    if (body.status && body.status !== task.status) {
      if (!canTransition(TASK_TRANSITIONS, task.status, body.status)) {
        return invalidTransition(task.status, body.status);
      }
    }

    // Apply update
    const updated: MockDesignTask = {
      ...task,
      ...body,
      version: task.version + 1,
      updatedAt: now(),
    };

    // Set startedAt when moving to IN_PROGRESS
    if (body.status === 'IN_PROGRESS' && !task.startedAt) {
      updated.startedAt = now();
    }

    // Set completedAt when APPROVED
    if (body.status === 'APPROVED') {
      updated.completedAt = now();
    }

    tasks[taskIndex] = updated;
    return HttpResponse.json(updated);
  }),

  // POST /api/tasks/:id/move - Kanban move operation
  http.post('/api/tasks/:id/move', async ({ params, request }) => {
    const body = (await request.json()) as {
      columnId: string;
      position: number;
      version: number;
    };

    const taskIndex = tasks.findIndex((t) => t.id === params.id);
    if (taskIndex === -1) return notFound('Task');

    const task = tasks[taskIndex]!;

    // Optimistic locking
    if (body.version !== task.version) {
      return conflict(task.version, body.version);
    }

    // Find target column
    const column = mockColumns.find((c) => c.id === body.columnId);
    if (!column) return notFound('Column');

    // Validate status transition
    if (!canTransition(TASK_TRANSITIONS, task.status, column.status)) {
      return invalidTransition(task.status, column.status);
    }

    // Reorder tasks in column
    const tasksInColumn = tasks
      .filter((t) => t.columnId === body.columnId && t.id !== task.id)
      .sort((a, b) => a.position - b.position);

    // Insert at position
    tasksInColumn.splice(body.position - 1, 0, task);

    // Update positions
    tasksInColumn.forEach((t, idx) => {
      const tIdx = tasks.findIndex((x) => x.id === t.id);
      if (tIdx !== -1) {
        tasks[tIdx] = { ...tasks[tIdx]!, position: idx + 1 };
      }
    });

    // Update moved task
    const updated: MockDesignTask = {
      ...task,
      columnId: body.columnId,
      workspaceId: column.workspaceId,
      status: column.status,
      position: body.position,
      version: task.version + 1,
      updatedAt: now(),
    };

    tasks[taskIndex] = updated;
    return HttpResponse.json(updated);
  }),

  // POST /api/tasks/:id/assign
  http.post('/api/tasks/:id/assign', async ({ params, request }) => {
    const body = (await request.json()) as {
      assigneeId: string;
      assignedBy: string;
    };

    const taskIndex = tasks.findIndex((t) => t.id === params.id);
    if (taskIndex === -1) return notFound('Task');

    const task = tasks[taskIndex]!;
    const updated: MockDesignTask = {
      ...task,
      assigneeId: body.assigneeId,
      assignedBy: body.assignedBy,
      assignedAt: now(),
      version: task.version + 1,
      updatedAt: now(),
    };

    tasks[taskIndex] = updated;
    return HttpResponse.json(updated);
  }),

  // GET /api/tasks/:id/designs
  http.get('/api/tasks/:id/designs', ({ params }) => {
    const versions = findDesignVersions(params.id as string);
    return HttpResponse.json({ versions, total: versions.length });
  }),

  // GET /api/workspaces/:id/board - Kanban board
  http.get('/api/workspaces/:id/board', ({ params }) => {
    const workspaceId = params.id as string;
    const workspaceTasks = tasks.filter((t) => t.workspaceId === workspaceId);
    const columns = mockColumns
      .filter((c) => c.workspaceId === workspaceId)
      .sort((a, b) => a.sortOrder - b.sortOrder)
      .map((col) => ({
        ...col,
        tasks: workspaceTasks
          .filter((t) => t.columnId === col.id)
          .sort((a, b) => a.position - b.position),
      }));

    return HttpResponse.json({ columns });
  }),

  // GET /api/backlog - Backlog tasks
  http.get('/api/backlog', () => {
    const backlog = tasks.filter((t) => !t.workspaceId);
    return HttpResponse.json({ tasks: backlog, total: backlog.length });
  }),
];
```

### Step 3.3: Update orders.ts handler

```typescript
// apps/web/src/mocks/handlers/orders.ts
import { http, HttpResponse } from 'msw';
import type { MockOrder, OrderStatus } from '../types';
import { mockOrders, findOrderItems } from '../data';
import {
  notFound,
  badRequest,
  invalidTransition,
  canTransition,
  ORDER_TRANSITIONS,
  now,
} from './utils';

// In-memory state
let orders = [...mockOrders];

export const ordersHandlers = [
  // GET /api/orders
  http.get('/api/orders', ({ request }) => {
    const url = new URL(request.url);
    const status = url.searchParams.get('status') as OrderStatus | null;
    const search = url.searchParams.get('search');

    let filtered = [...orders];

    if (status) {
      filtered = filtered.filter((o) => o.status === status);
    }

    if (search) {
      const q = search.toLowerCase();
      filtered = filtered.filter(
        (o) =>
          o.orderNumber.toLowerCase().includes(q) ||
          o.customerName.toLowerCase().includes(q) ||
          o.customerEmail?.toLowerCase().includes(q)
      );
    }

    return HttpResponse.json({ orders: filtered, total: filtered.length });
  }),

  // GET /api/orders/:id
  http.get('/api/orders/:id', ({ params }) => {
    const order = orders.find((o) => o.id === params.id);
    if (!order) return notFound('Order');
    return HttpResponse.json(order);
  }),

  // PATCH /api/orders/:id
  http.patch('/api/orders/:id', async ({ params, request }) => {
    const body = (await request.json()) as Partial<MockOrder>;
    const orderIndex = orders.findIndex((o) => o.id === params.id);

    if (orderIndex === -1) return notFound('Order');

    const order = orders[orderIndex]!;

    // Status transition validation
    if (body.status && body.status !== order.status) {
      if (!canTransition(ORDER_TRANSITIONS, order.status, body.status)) {
        return invalidTransition(order.status, body.status);
      }

      // SHIPPED requires tracking number
      if (body.status === 'SHIPPED') {
        if (!body.trackingNumber && !order.trackingNumber) {
          return badRequest('Tracking number required for SHIPPED status');
        }
      }
    }

    const updated: MockOrder = {
      ...order,
      ...body,
      updatedAt: now(),
    };

    // Set shippedAt when transitioning to SHIPPED
    if (body.status === 'SHIPPED' && !order.shippedAt) {
      updated.shippedAt = now();
    }

    orders[orderIndex] = updated;
    return HttpResponse.json(updated);
  }),

  // POST /api/orders/:id/confirm
  http.post('/api/orders/:id/confirm', ({ params }) => {
    const orderIndex = orders.findIndex((o) => o.id === params.id);
    if (orderIndex === -1) return notFound('Order');

    const order = orders[orderIndex]!;

    if (order.status !== 'PENDING') {
      return invalidTransition(order.status, 'CONFIRMED');
    }

    const updated: MockOrder = {
      ...order,
      status: 'CONFIRMED',
      updatedAt: now(),
    };

    orders[orderIndex] = updated;
    return HttpResponse.json(updated);
  }),

  // POST /api/orders/:id/ship
  http.post('/api/orders/:id/ship', async ({ params, request }) => {
    const body = (await request.json()) as {
      trackingNumber: string;
      carrier: string;
      fulfillerName?: string;
    };

    const orderIndex = orders.findIndex((o) => o.id === params.id);
    if (orderIndex === -1) return notFound('Order');

    const order = orders[orderIndex]!;

    if (order.status !== 'READY_TO_FULFILL') {
      return invalidTransition(order.status, 'SHIPPED');
    }

    if (!body.trackingNumber) {
      return badRequest('Tracking number is required');
    }

    const updated: MockOrder = {
      ...order,
      status: 'SHIPPED',
      trackingNumber: body.trackingNumber,
      carrier: body.carrier as MockOrder['carrier'],
      fulfillerName: body.fulfillerName,
      shippedAt: now(),
      updatedAt: now(),
    };

    orders[orderIndex] = updated;
    return HttpResponse.json(updated);
  }),

  // POST /api/orders/:id/cancel
  http.post('/api/orders/:id/cancel', ({ params }) => {
    const orderIndex = orders.findIndex((o) => o.id === params.id);
    if (orderIndex === -1) return notFound('Order');

    const order = orders[orderIndex]!;

    if (order.status === 'COMPLETED') {
      return badRequest('Cannot cancel completed order');
    }

    const updated: MockOrder = {
      ...order,
      status: 'CANCELLED',
      updatedAt: now(),
    };

    orders[orderIndex] = updated;
    return HttpResponse.json(updated);
  }),

  // GET /api/orders/:id/items
  http.get('/api/orders/:id/items', ({ params }) => {
    const items = findOrderItems(params.id as string);
    return HttpResponse.json({ items, total: items.length });
  }),

  // POST /api/orders/import
  http.post('/api/orders/import', () => {
    return HttpResponse.json({
      imported: 5,
      failed: 0,
      message: 'Import successful',
    });
  }),
];
```

### Step 3.4: Create workspaces.ts handler

```typescript
// apps/web/src/mocks/handlers/workspaces.ts
import { http, HttpResponse } from 'msw';
import type { MockWorkspace, MockColumn } from '../types';
import { mockWorkspaces, mockColumns, getColumnsByWorkspace } from '../data';
import { notFound, badRequest, now } from './utils';

// In-memory state
let workspaces = [...mockWorkspaces];
let columns = [...mockColumns];

export const workspacesHandlers = [
  // GET /api/workspaces
  http.get('/api/workspaces', () => {
    const sorted = [...workspaces].sort((a, b) => a.sortOrder - b.sortOrder);
    return HttpResponse.json({ workspaces: sorted, total: sorted.length });
  }),

  // GET /api/workspaces/:id
  http.get('/api/workspaces/:id', ({ params }) => {
    const workspace = workspaces.find((w) => w.id === params.id);
    if (!workspace) return notFound('Workspace');
    return HttpResponse.json(workspace);
  }),

  // POST /api/workspaces
  http.post('/api/workspaces', async ({ request }) => {
    const body = (await request.json()) as Partial<MockWorkspace>;

    if (!body.name) {
      return badRequest('Name is required');
    }

    const id = `ws-${Date.now()}`;
    const newWorkspace: MockWorkspace = {
      id,
      tenantId: body.tenantId || 'default-tenant',
      name: body.name,
      description: body.description,
      color: body.color || '#3B82F6',
      icon: body.icon,
      isDefault: false,
      sortOrder: workspaces.length + 1,
      createdAt: now(),
      updatedAt: now(),
    };

    workspaces.push(newWorkspace);
    return HttpResponse.json(newWorkspace, { status: 201 });
  }),

  // PATCH /api/workspaces/:id
  http.patch('/api/workspaces/:id', async ({ params, request }) => {
    const body = (await request.json()) as Partial<MockWorkspace>;
    const wsIndex = workspaces.findIndex((w) => w.id === params.id);

    if (wsIndex === -1) return notFound('Workspace');

    const updated: MockWorkspace = {
      ...workspaces[wsIndex]!,
      ...body,
      updatedAt: now(),
    };

    workspaces[wsIndex] = updated;
    return HttpResponse.json(updated);
  }),

  // DELETE /api/workspaces/:id
  http.delete('/api/workspaces/:id', ({ params }) => {
    const wsIndex = workspaces.findIndex((w) => w.id === params.id);
    if (wsIndex === -1) return notFound('Workspace');

    const workspace = workspaces[wsIndex]!;
    if (workspace.isDefault) {
      return badRequest('Cannot delete default workspace');
    }

    workspaces.splice(wsIndex, 1);
    return new HttpResponse(null, { status: 204 });
  }),

  // GET /api/workspaces/:id/columns
  http.get('/api/workspaces/:id/columns', ({ params }) => {
    const wsCols = columns
      .filter((c) => c.workspaceId === params.id)
      .sort((a, b) => a.sortOrder - b.sortOrder);
    return HttpResponse.json({ columns: wsCols, total: wsCols.length });
  }),

  // POST /api/workspaces/:id/columns
  http.post('/api/workspaces/:id/columns', async ({ params, request }) => {
    const body = (await request.json()) as Partial<MockColumn>;
    const workspaceId = params.id as string;

    if (!body.name) {
      return badRequest('Name is required');
    }

    const wsCols = columns.filter((c) => c.workspaceId === workspaceId);

    const newColumn: MockColumn = {
      id: `col-${Date.now()}`,
      workspaceId,
      name: body.name,
      status: body.status || 'TODO',
      color: body.color || '#3B82F6',
      sortOrder: wsCols.length + 1,
      wipLimit: body.wipLimit,
      isSystem: false,
      createdAt: now(),
    };

    columns.push(newColumn);
    return HttpResponse.json(newColumn, { status: 201 });
  }),

  // PATCH /api/columns/:id
  http.patch('/api/columns/:id', async ({ params, request }) => {
    const body = (await request.json()) as Partial<MockColumn>;
    const colIndex = columns.findIndex((c) => c.id === params.id);

    if (colIndex === -1) return notFound('Column');

    const column = columns[colIndex]!;
    if (column.isSystem && (body.name || body.status)) {
      return badRequest('Cannot modify system column name or status');
    }

    const updated: MockColumn = {
      ...column,
      ...body,
    };

    columns[colIndex] = updated;
    return HttpResponse.json(updated);
  }),

  // DELETE /api/columns/:id
  http.delete('/api/columns/:id', ({ params }) => {
    const colIndex = columns.findIndex((c) => c.id === params.id);
    if (colIndex === -1) return notFound('Column');

    const column = columns[colIndex]!;
    if (column.isSystem) {
      return badRequest('Cannot delete system column');
    }

    columns.splice(colIndex, 1);
    return new HttpResponse(null, { status: 204 });
  }),
];
```

### Step 3.5: Create designs.ts handler

```typescript
// apps/web/src/mocks/handlers/designs.ts
import { http, HttpResponse } from 'msw';
import type { MockDesign, DesignStatus } from '../types';
import { mockDesigns, findTask, mockTasks } from '../data';
import {
  notFound,
  badRequest,
  invalidTransition,
  canTransition,
  DESIGN_TRANSITIONS,
  now,
} from './utils';

// In-memory state
let designs = [...mockDesigns];
let tasks = [...mockTasks];

export const designsHandlers = [
  // GET /api/designs/:id
  http.get('/api/designs/:id', ({ params }) => {
    const design = designs.find((d) => d.id === params.id);
    if (!design) return notFound('Design');
    return HttpResponse.json(design);
  }),

  // POST /api/tasks/:taskId/designs - Upload new design
  http.post('/api/tasks/:taskId/designs', async ({ params, request }) => {
    const body = (await request.json()) as {
      fileName: string;
      fileUrl: string;
      fileType: string;
      fileSize: number;
      uploadedBy: string;
      metadata?: MockDesign['metadata'];
    };

    const taskId = params.taskId as string;
    const task = tasks.find((t) => t.id === taskId);
    if (!task) return notFound('Task');

    // Get current version count
    const taskDesigns = designs.filter((d) => d.taskId === taskId);
    const version = taskDesigns.length + 1;

    const newDesign: MockDesign = {
      id: `design-${Date.now()}`,
      taskId,
      version,
      fileName: body.fileName,
      fileUrl: body.fileUrl,
      fileType: body.fileType as MockDesign['fileType'],
      fileSize: body.fileSize,
      thumbnailUrl: body.fileUrl.replace(/\.[^.]+$/, '-thumb.jpg'),
      uploadedBy: body.uploadedBy,
      uploadedAt: now(),
      status: 'DRAFT',
      metadata: body.metadata,
    };

    designs.push(newDesign);

    // Update task design count
    const taskIndex = tasks.findIndex((t) => t.id === taskId);
    if (taskIndex !== -1) {
      tasks[taskIndex] = {
        ...tasks[taskIndex]!,
        designCount: version,
        currentDesignId: newDesign.id,
        updatedAt: now(),
      };
    }

    return HttpResponse.json(newDesign, { status: 201 });
  }),

  // POST /api/designs/:id/submit
  http.post('/api/designs/:id/submit', ({ params }) => {
    const designIndex = designs.findIndex((d) => d.id === params.id);
    if (designIndex === -1) return notFound('Design');

    const design = designs[designIndex]!;

    if (design.status !== 'DRAFT') {
      return invalidTransition(design.status, 'SUBMITTED');
    }

    const updated: MockDesign = {
      ...design,
      status: 'SUBMITTED',
    };

    designs[designIndex] = updated;

    // Update task status to REVIEW
    const taskIndex = tasks.findIndex((t) => t.id === design.taskId);
    if (taskIndex !== -1) {
      const task = tasks[taskIndex]!;
      if (task.status === 'IN_PROGRESS') {
        tasks[taskIndex] = {
          ...task,
          status: 'REVIEW',
          updatedAt: now(),
        };
      }
    }

    return HttpResponse.json(updated);
  }),

  // POST /api/designs/:id/approve
  http.post('/api/designs/:id/approve', async ({ params, request }) => {
    const body = (await request.json()) as {
      reviewerId: string;
      reviewNotes?: string;
    };

    const designIndex = designs.findIndex((d) => d.id === params.id);
    if (designIndex === -1) return notFound('Design');

    const design = designs[designIndex]!;

    if (design.status !== 'SUBMITTED') {
      return invalidTransition(design.status, 'APPROVED');
    }

    const updated: MockDesign = {
      ...design,
      status: 'APPROVED',
      reviewerId: body.reviewerId,
      reviewedAt: now(),
      reviewNotes: body.reviewNotes,
    };

    designs[designIndex] = updated;

    // Update task status to APPROVED
    const taskIndex = tasks.findIndex((t) => t.id === design.taskId);
    if (taskIndex !== -1) {
      tasks[taskIndex] = {
        ...tasks[taskIndex]!,
        status: 'APPROVED',
        reviewerId: body.reviewerId,
        reviewedAt: now(),
        completedAt: now(),
        updatedAt: now(),
      };
    }

    return HttpResponse.json(updated);
  }),

  // POST /api/designs/:id/reject
  http.post('/api/designs/:id/reject', async ({ params, request }) => {
    const body = (await request.json()) as {
      reviewerId: string;
      reviewNotes: string;
    };

    if (!body.reviewNotes || body.reviewNotes.length < 10) {
      return badRequest('Review notes required (min 10 characters)');
    }

    const designIndex = designs.findIndex((d) => d.id === params.id);
    if (designIndex === -1) return notFound('Design');

    const design = designs[designIndex]!;

    if (design.status !== 'SUBMITTED') {
      return invalidTransition(design.status, 'REJECTED');
    }

    const updated: MockDesign = {
      ...design,
      status: 'REJECTED',
      reviewerId: body.reviewerId,
      reviewedAt: now(),
      reviewNotes: body.reviewNotes,
    };

    designs[designIndex] = updated;

    // Update task status to REVISION
    const taskIndex = tasks.findIndex((t) => t.id === design.taskId);
    if (taskIndex !== -1) {
      tasks[taskIndex] = {
        ...tasks[taskIndex]!,
        status: 'REVISION',
        reviewerId: body.reviewerId,
        reviewNotes: body.reviewNotes,
        updatedAt: now(),
      };
    }

    return HttpResponse.json(updated);
  }),
];
```

### Step 3.6: Update handlers/index.ts

```typescript
// apps/web/src/mocks/handlers/index.ts
import { authHandlers } from './auth';
import { tasksHandlers } from './tasks';
import { ordersHandlers } from './orders';
import { workspacesHandlers } from './workspaces';
import { designsHandlers } from './designs';

export const handlers = [
  ...authHandlers,
  ...tasksHandlers,
  ...ordersHandlers,
  ...workspacesHandlers,
  ...designsHandlers,
];
```

## Success Criteria

- [ ] All CRUD operations work (create, read, update, delete)
- [ ] State transitions validated (invalid → 400 error)
- [ ] Optimistic locking works (version mismatch → 412 error)
- [ ] Kanban board endpoint returns columns with sorted tasks
- [ ] Design upload/review flow works end-to-end

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| In-memory state loss | LOW | Mock only, acceptable for dev |
| Complex transition logic | MEDIUM | Unit test critical paths |
| Handler order conflicts | LOW | Specific routes before generic |
