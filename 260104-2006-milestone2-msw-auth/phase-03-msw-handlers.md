# Phase 3: MSW Handlers (~1.5h)

## Context

- Plan: [plan.md](./plan.md)
- Depends on: [Phase 2](./phase-02-msw-data-files.md)
- Issue: #64
- Research: [MSW Patterns](./research/researcher-msw-patterns.md)

## Overview

Add x-tenant-id header extraction utility. Update handlers to use new types and return spec-compliant responses.

## Requirements

1. Create context.ts for header extraction
2. Update auth.ts - return token, validate against stored token for /me
3. Update tasks.ts - use new types, filter by tenant
4. Update orders.ts - use new types
5. Add columns handler if needed

## Architecture

### Context Extraction Pattern

```typescript
// handlers/context.ts
export interface RequestContext {
  tenantId: string;
  token?: string;
}

export function extractContext(request: Request): RequestContext {
  return {
    tenantId: request.headers.get('x-tenant-id') || TENANT_ID,
    token: request.headers.get('authorization')?.replace('Bearer ', ''),
  };
}
```

### Auth Token Flow

```
Login → Returns { user, token: 'mock-jwt-{userId}' }
                          ↓
Stored in localStorage by client
                          ↓
GET /api/auth/me → Extract token from Authorization header
                → Parse userId from token
                → Return user
```

### Handler Response Types

All handlers return spec-aligned response shapes:

```typescript
// Tasks
GET /api/tasks → { tasks: MockTask[], total: number }
POST /api/tasks → MockTask
PUT /api/tasks/:id → MockTask

// Orders
GET /api/orders → { orders: MockOrder[], total: number }

// Columns
GET /api/columns → { columns: MockColumn[] }
```

## Related Files

| File | Action |
|------|--------|
| `apps/web/src/mocks/handlers/context.ts` | CREATE |
| `apps/web/src/mocks/handlers/auth.ts` | UPDATE |
| `apps/web/src/mocks/handlers/tasks.ts` | UPDATE |
| `apps/web/src/mocks/handlers/orders.ts` | UPDATE |
| `apps/web/src/mocks/handlers/columns.ts` | CREATE (optional) |
| `apps/web/src/mocks/handlers/index.ts` | UPDATE |

## Implementation Steps

### Step 1: Create context.ts

```typescript
import { TENANT_ID } from '../factories/mock-factory';

export interface RequestContext {
  tenantId: string;
  token?: string;
  userId?: string;
}

export function extractContext(request: Request): RequestContext {
  const token = request.headers.get('authorization')?.replace('Bearer ', '');
  // Token format: mock-jwt-{userId}
  const userId = token?.startsWith('mock-jwt-') ? token.replace('mock-jwt-', '') : undefined;

  return {
    tenantId: request.headers.get('x-tenant-id') || TENANT_ID,
    token,
    userId,
  };
}
```

### Step 2: Update auth.ts

**Login handler:**
- Generate token as `mock-jwt-{userId}`
- Return `{ user, token }`

**GET /api/auth/me handler:**
- Extract token from Authorization header
- Parse userId from token
- Return matching user or 401

```typescript
http.get('/api/auth/me', ({ request }) => {
  const ctx = extractContext(request);
  if (!ctx.userId) {
    return HttpResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }
  const user = findUser(ctx.userId);
  if (!user) {
    return HttpResponse.json({ error: 'User not found' }, { status: 401 });
  }
  return HttpResponse.json(user);
}),
```

### Step 3: Update tasks.ts

- Import new types
- Use `extractContext` for tenant filtering (optional, single tenant for now)
- Update response shape to `{ tasks: MockTask[], total: number }`
- Ensure CRUD operations use correct status values

### Step 4: Update orders.ts

- Import new types
- Update response shape to `{ orders: MockOrder[], total: number }`
- Add filtering by tenant (optional)

### Step 5: Create columns.ts (optional)

```typescript
import { http, HttpResponse } from 'msw';
import { mockColumns } from '../data';

export const columnsHandlers = [
  http.get('/api/columns', () => {
    return HttpResponse.json({ columns: mockColumns });
  }),
];
```

### Step 6: Update handlers/index.ts

Export all handlers:
```typescript
import { authHandlers } from './auth';
import { tasksHandlers } from './tasks';
import { ordersHandlers } from './orders';
import { columnsHandlers } from './columns';

export const handlers = [
  ...authHandlers,
  ...tasksHandlers,
  ...ordersHandlers,
  ...columnsHandlers,
];
```

## Todo List

- [ ] Create context.ts with extractContext
- [ ] Update auth.ts login to return mock-jwt-{userId} token
- [ ] Update auth.ts /me to validate token and return user
- [ ] Update tasks.ts with new types
- [ ] Update orders.ts with new types
- [ ] Create columns.ts handler (optional)
- [ ] Update handlers/index.ts
- [ ] Test handlers manually

## Success Criteria

- [ ] extractContext correctly parses Authorization header
- [ ] POST /api/auth/login returns `{ user, token }`
- [ ] GET /api/auth/me returns user when valid token, 401 otherwise
- [ ] GET /api/tasks returns spec-compliant MockTask[]
- [ ] GET /api/orders returns spec-compliant MockOrder[]
- [ ] All handlers use consistent response shapes
