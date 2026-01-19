---
phase: 4
title: "Update Orders Handlers"
status: pending
effort: 30m
---

# Phase 04: Update Orders Handlers

## Overview

Update order handlers to use `/api/v1/orders/*` URLs and new response format.

## Context

**Convention Reference:** `docs/frontend/mock-api-convention.md` Section 5.3

## Related Code Files

- **Modify:** `apps/web/src/mocks/handlers/orders.ts`

## URL Mapping

| Current | New |
|---------|-----|
| `GET /api/orders` | `GET /api/v1/orders/` |
| `GET /api/orders/:id` | `GET /api/v1/orders/:id` |
| `GET /api/orders/:id/items` | `GET /api/v1/orders/:id/items` |
| `POST /api/orders/import` | `POST /api/v1/orders/import/execute` |
| (missing) | `POST /api/v1/orders/import/validate` |

## Implementation Steps

### 1. Update imports

```typescript
import { http } from 'msw';
import { mockOrders, findOrder, findOrderItems } from '../data';
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

### 2. Update list orders handler

```typescript
// GET /api/v1/orders/
http.get('/api/v1/orders/', ({ request }) => {
  const url = new URL(request.url);
  const { page, limit } = parsePagination(url);
  const status = url.searchParams.get('status');
  const search = url.searchParams.get('search');
  const from = url.searchParams.get('from');
  const to = url.searchParams.get('to');

  let filtered = mockOrders;

  if (status) filtered = filtered.filter(o => o.status === status);
  if (search) {
    const term = search.toLowerCase();
    filtered = filtered.filter(o =>
      o.orderNumber.toLowerCase().includes(term) ||
      o.customerName.toLowerCase().includes(term)
    );
  }
  if (from) filtered = filtered.filter(o => new Date(o.createdAt) >= new Date(from));
  if (to) filtered = filtered.filter(o => new Date(o.createdAt) <= new Date(to));

  const paginated = paginateArray(filtered, page, limit);
  const pagination = getPaginationMeta(filtered.length, page, limit);

  return listResponse(paginated, pagination);
}),
```

### 3. Update get order handler

```typescript
// GET /api/v1/orders/:id
http.get('/api/v1/orders/:id', ({ params }) => {
  const order = findOrder(params.id as string);

  if (!order) {
    return errors.notFound('Order');
  }

  return successResponse(order);
}),
```

### 4. Update get order items handler

```typescript
// GET /api/v1/orders/:id/items
http.get('/api/v1/orders/:id/items', ({ params }) => {
  const order = findOrder(params.id as string);
  if (!order) {
    return errors.notFound('Order');
  }

  const items = findOrderItems(params.id as string);
  return successResponse({ items, total: items.length });
}),
```

### 5. Add import validate handler (NEW)

```typescript
// POST /api/v1/orders/import/validate
http.post('/api/v1/orders/import/validate', async () => {
  // Simulate validation result
  return successResponse({
    valid: [
      { row: 1, order_number: 'ORD-001', customer: 'John Doe' },
      { row: 2, order_number: 'ORD-002', customer: 'Jane Smith' },
    ],
    warnings: [
      { row: 3, field: 'phone', message: 'Phone format may be invalid' },
    ],
    errors: [],
  });
}),
```

### 6. Update import execute handler

```typescript
// POST /api/v1/orders/import/execute
http.post('/api/v1/orders/import/execute', async () => {
  return successResponse({
    created_orders: 5,
    created_tasks: 8,
    message: 'Import successful',
  }, 201);
}),
```

## Todo List

- [ ] Import response helpers
- [ ] Update all URLs to `/api/v1/orders/*`
- [ ] Add pagination to list
- [ ] Add filtering (status, search, from, to)
- [ ] Add import/validate endpoint
- [ ] Wrap responses in `{ data: ... }`
- [ ] Use error helpers

## Success Criteria

- [ ] All order endpoints use `/api/v1/orders/*`
- [ ] List returns pagination
- [ ] Import validate and execute endpoints work
- [ ] Error responses have proper codes
