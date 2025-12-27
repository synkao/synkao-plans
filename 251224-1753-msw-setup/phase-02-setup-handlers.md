# Phase 02: Setup Handlers

## Context

- **Plan:** [plan.md](./plan.md)
- **Issue:** #3
- **Depends on:** Phase 01
- **Working dir:** `apps/web/`

## Overview

| Field | Value |
|-------|-------|
| Date | 2025-12-24 |
| Priority | P0 |
| Effort | 1.5h |
| Status | Pending |

Tạo request handlers cho API endpoints. Phase này focus vào structure và basic responses. Mock data chi tiết sẽ được handle ở Issue #4.

## Requirements

1. Create auth handlers (login, logout, refresh, me)
2. Create tasks handlers (list, detail, update, upload)
3. Create orders handlers (list, detail, import)
4. Setup browser worker
5. Setup server (SSR) worker

## API Response Types

Cần định nghĩa types cho API responses trước khi viết handlers.

```typescript
// types/api.ts
interface User {
  id: string;
  email: string;
  name: string;
  role: 'STAFF' | 'DESIGNER' | 'ADMIN';
  avatar?: string;
}

interface Task {
  id: string;
  orderId: string;
  productName: string;
  status: 'TODO' | 'IN_PROGRESS' | 'REVIEW' | 'REVISION' | 'DONE';
  priority: 'LOW' | 'NORMAL' | 'HIGH' | 'URGENT';
  assigneeId?: string;
  designUrl?: string;
  dueDate?: string;
  createdAt: string;
  updatedAt: string;
}

interface Order {
  id: string;
  externalId: string;
  customerName: string;
  items: OrderItem[];
  status: 'PENDING' | 'PROCESSING' | 'SHIPPED' | 'DELIVERED' | 'FAILED';
  createdAt: string;
}
```

## Implementation Steps

### Step 1: Create Type Definitions

```typescript
// src/mocks/types.ts
export interface MockUser {
  id: string;
  email: string;
  name: string;
  role: 'STAFF' | 'DESIGNER' | 'ADMIN';
  avatar?: string;
}

export interface MockTask {
  id: string;
  orderId: string;
  productName: string;
  status: 'TODO' | 'IN_PROGRESS' | 'REVIEW' | 'REVISION' | 'DONE';
  priority: 'LOW' | 'NORMAL' | 'HIGH' | 'URGENT';
  assigneeId?: string;
  designUrl?: string;
  dueDate?: string;
  createdAt: string;
  updatedAt: string;
}

export interface MockOrder {
  id: string;
  externalId: string;
  customerName: string;
  status: 'PENDING' | 'PROCESSING' | 'SHIPPED' | 'DELIVERED' | 'FAILED';
  createdAt: string;
}
```

### Step 2: Create Auth Handlers

```typescript
// src/mocks/handlers/auth.ts
import { http, HttpResponse } from 'msw';

const mockUser = {
  id: '1',
  email: 'designer@synkao.com',
  name: 'Designer Demo',
  role: 'DESIGNER' as const,
};

export const authHandlers = [
  // POST /api/auth/login
  http.post('/api/auth/login', async ({ request }) => {
    const body = await request.json() as { email: string; password: string };

    if (body.email && body.password) {
      return HttpResponse.json({
        user: mockUser,
        token: 'mock-jwt-token',
      });
    }

    return HttpResponse.json(
      { error: 'Invalid credentials' },
      { status: 401 }
    );
  }),

  // POST /api/auth/logout
  http.post('/api/auth/logout', () => {
    return HttpResponse.json({ success: true });
  }),

  // POST /api/auth/refresh
  http.post('/api/auth/refresh', () => {
    return HttpResponse.json({
      token: 'mock-refreshed-token',
    });
  }),

  // GET /api/auth/me
  http.get('/api/auth/me', () => {
    return HttpResponse.json(mockUser);
  }),
];
```

### Step 3: Create Tasks Handlers

```typescript
// src/mocks/handlers/tasks.ts
import { http, HttpResponse } from 'msw';

const mockTasks = [
  {
    id: '1',
    orderId: 'ORD-001',
    productName: 'Custom T-Shirt Design',
    status: 'TODO',
    priority: 'HIGH',
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
  },
  // More mock tasks will be added in Issue #4
];

export const tasksHandlers = [
  // GET /api/tasks
  http.get('/api/tasks', ({ request }) => {
    const url = new URL(request.url);
    const status = url.searchParams.get('status');

    let filtered = [...mockTasks];
    if (status) {
      filtered = filtered.filter(t => t.status === status);
    }

    return HttpResponse.json({ tasks: filtered });
  }),

  // GET /api/tasks/:id
  http.get('/api/tasks/:id', ({ params }) => {
    const task = mockTasks.find(t => t.id === params.id);

    if (!task) {
      return HttpResponse.json(
        { error: 'Task not found' },
        { status: 404 }
      );
    }

    return HttpResponse.json(task);
  }),

  // PATCH /api/tasks/:id
  http.patch('/api/tasks/:id', async ({ params, request }) => {
    const body = await request.json() as Record<string, unknown>;
    const task = mockTasks.find(t => t.id === params.id);

    if (!task) {
      return HttpResponse.json(
        { error: 'Task not found' },
        { status: 404 }
      );
    }

    // Update task (in-memory)
    Object.assign(task, body, { updatedAt: new Date().toISOString() });

    return HttpResponse.json(task);
  }),

  // POST /api/tasks/:id/designs
  http.post('/api/tasks/:id/designs', async () => {
    return HttpResponse.json({
      designUrl: 'https://example.com/design.png',
      uploadedAt: new Date().toISOString(),
    });
  }),

  // GET /api/workspaces/:id/board
  http.get('/api/workspaces/:id/board', () => {
    return HttpResponse.json({
      columns: [
        { id: 'todo', name: 'TODO', tasks: mockTasks.filter(t => t.status === 'TODO') },
        { id: 'in-progress', name: 'IN PROGRESS', tasks: mockTasks.filter(t => t.status === 'IN_PROGRESS') },
        { id: 'review', name: 'REVIEW', tasks: mockTasks.filter(t => t.status === 'REVIEW') },
        { id: 'revision', name: 'REVISION', tasks: mockTasks.filter(t => t.status === 'REVISION') },
        { id: 'done', name: 'DONE', tasks: mockTasks.filter(t => t.status === 'DONE') },
      ],
    });
  }),
];
```

### Step 4: Create Orders Handlers

```typescript
// src/mocks/handlers/orders.ts
import { http, HttpResponse } from 'msw';

const mockOrders = [
  {
    id: '1',
    externalId: 'EXT-001',
    customerName: 'Nguyen Van A',
    status: 'PENDING',
    createdAt: new Date().toISOString(),
  },
];

export const ordersHandlers = [
  // GET /api/orders
  http.get('/api/orders', () => {
    return HttpResponse.json({ orders: mockOrders });
  }),

  // GET /api/orders/:id
  http.get('/api/orders/:id', ({ params }) => {
    const order = mockOrders.find(o => o.id === params.id);

    if (!order) {
      return HttpResponse.json(
        { error: 'Order not found' },
        { status: 404 }
      );
    }

    return HttpResponse.json(order);
  }),

  // POST /api/orders/import
  http.post('/api/orders/import', async () => {
    return HttpResponse.json({
      imported: 5,
      failed: 0,
      message: 'Import successful',
    });
  }),
];
```

### Step 5: Create Handlers Index

```typescript
// src/mocks/handlers/index.ts
import { authHandlers } from './auth';
import { tasksHandlers } from './tasks';
import { ordersHandlers } from './orders';

export const handlers = [
  ...authHandlers,
  ...tasksHandlers,
  ...ordersHandlers,
];
```

### Step 6: Setup Browser Worker

```typescript
// src/mocks/browser.ts
import { setupWorker } from 'msw/browser';
import { handlers } from './handlers';

export const worker = setupWorker(...handlers);
```

### Step 7: Setup Server (SSR)

```typescript
// src/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

## File Structure After Phase 2

```
apps/web/src/mocks/
├── browser.ts
├── server.ts
├── types.ts
├── handlers/
│   ├── index.ts
│   ├── auth.ts
│   ├── tasks.ts
│   └── orders.ts
└── data/               # For Issue #4
```

## Success Criteria

- [ ] All handlers created
- [ ] Types defined
- [ ] Browser worker setup
- [ ] Server worker setup
- [ ] No TypeScript errors

## Next Steps

→ [Phase 03: Next.js Integration](./phase-03-nextjs-integration.md)
