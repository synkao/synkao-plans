# Phase 02: API Layer - New Endpoints

**Priority:** P1 - High
**Status:** ✅ Completed (2026-01-22)
**Depends on:** Phase 01
**Code Review:** [Phase 02 Code Review](reports/code-reviewer-260122-phase-02-api-endpoints.md)

## Overview

Add 3 new MSW mock API endpoints for Order Detail functionality.

## Related Files

**Modify:**
- `apps/web/src/mocks/handlers/orders.ts` - Add new endpoints
- `apps/web/src/mocks/data/index.ts` - Export timeline data

**Create:**
- `apps/web/src/features/orders/hooks/use-order-timeline.ts` - Timeline hook
- `apps/web/src/features/orders/lib/orders-client.ts` - Add new API methods (if not exists)

## New Endpoints

### 1. GET /api/v1/orders/:id/timeline

Returns order-specific timeline events.

```typescript
// Request
GET /api/v1/orders/:id/timeline

// Response 200
{
  "data": {
    "events": [
      {
        "id": "oevt-00001",
        "orderId": "ord-00001",
        "type": "ORDER_CONFIRMED",
        "userId": "usr-staff-001",
        "userName": "Admin User",
        "description": "Order confirmed",
        "createdAt": "2026-01-20T10:30:00Z"
      },
      // ... more events
    ],
    "total": 5
  }
}

// Response 404
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Order not found"
  }
}
```

### 2. PATCH /api/v1/orders/:id

Update order notes only (status readonly).

```typescript
// Request
PATCH /api/v1/orders/:id
Content-Type: application/json

{
  "notes": "Customer requests delivery before 12:00 PM"
}

// Response 200
{
  "data": { /* updated order */ }
}

// Response 404
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Order not found"
  }
}
```

### 3. POST /api/v1/orders/:id/cancel

Cancel order with optional reason.

```typescript
// Request
POST /api/v1/orders/:id/cancel
Content-Type: application/json

{
  "reason": "Customer requested cancellation"
}

// Response 200
{
  "data": { /* cancelled order with status: CANCELLED */ }
}

// Response 400
{
  "error": {
    "code": "INVALID_STATUS",
    "message": "Order cannot be cancelled in current status"
  }
}
```

## Implementation

### 1. Update orders.ts handlers

```typescript
// In mocks/handlers/orders.ts

import { getOrderTimeline } from '../data';

// Add to ordersHandlers array:

// GET /api/v1/orders/:id/timeline
http.get('/api/v1/orders/:id/timeline', ({ params }) => {
  const order = findOrder(params.id as string);
  if (!order) {
    return errors.notFound('Order');
  }

  const events = getOrderTimeline(params.id as string);
  return successResponse({ events, total: events.length });
}),

// PATCH /api/v1/orders/:id
http.patch('/api/v1/orders/:id', async ({ params, request }) => {
  const order = findOrder(params.id as string);
  if (!order) {
    return errors.notFound('Order');
  }

  const body = (await request.json()) as { notes?: string };

  // Only update notes (mock - doesn't persist)
  if (body.notes !== undefined) {
    order.notes = body.notes;
  }

  return successResponse(order);
}),

// POST /api/v1/orders/:id/cancel
http.post('/api/v1/orders/:id/cancel', async ({ params, request }) => {
  const order = findOrder(params.id as string);
  if (!order) {
    return errors.notFound('Order');
  }

  // Check if order can be cancelled
  const nonCancellable = ['SHIPPED', 'COMPLETED', 'CANCELLED'];
  if (nonCancellable.includes(order.status)) {
    return HttpResponse.json(
      { error: { code: 'INVALID_STATUS', message: 'Order cannot be cancelled' } },
      { status: 400 }
    );
  }

  const body = (await request.json()) as { reason?: string };

  // Update order status (mock - doesn't persist across requests)
  order.status = 'CANCELLED';
  order.notes = body.reason ? `Cancelled: ${body.reason}` : order.notes;

  return successResponse(order);
}),
```

### 2. Create use-order-timeline.ts hook

```typescript
// apps/web/src/features/orders/hooks/use-order-timeline.ts

import { useQuery } from '@tanstack/react-query';
import type { OrderTimelineEvent } from '../types';

interface TimelineResponse {
  events: OrderTimelineEvent[];
  total: number;
}

async function fetchOrderTimeline(orderId: string): Promise<TimelineResponse> {
  const res = await fetch(`/api/v1/orders/${orderId}/timeline`);
  if (!res.ok) throw new Error('Failed to fetch timeline');
  const json = await res.json();
  return json.data;
}

export function useOrderTimeline(orderId: string) {
  return useQuery({
    queryKey: ['orders', orderId, 'timeline'],
    queryFn: () => fetchOrderTimeline(orderId),
    enabled: !!orderId,
  });
}
```

### 3. Add mutation hooks

```typescript
// In hooks/index.ts or new file

import { useMutation, useQueryClient } from '@tanstack/react-query';

// Update order notes
export function useUpdateOrderNotes(orderId: string) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (notes: string) => {
      const res = await fetch(`/api/v1/orders/${orderId}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ notes }),
      });
      if (!res.ok) throw new Error('Failed to update notes');
      return res.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['orders', orderId] });
    },
  });
}

// Cancel order
export function useCancelOrder(orderId: string) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (reason?: string) => {
      const res = await fetch(`/api/v1/orders/${orderId}/cancel`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ reason }),
      });
      if (!res.ok) {
        const err = await res.json();
        throw new Error(err.error?.message || 'Failed to cancel order');
      }
      return res.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['orders'] });
    },
  });
}
```

## Todo

- [x] Export getOrderTimeline from data/index.ts (already exported via Phase 01)
- [x] Add GET /api/v1/orders/:id/timeline handler
- [x] Add PATCH /api/v1/orders/:id handler
- [x] Add POST /api/v1/orders/:id/cancel handler
- [x] Create useOrderTimeline hook
- [x] Create useUpdateOrderNotes hook
- [x] Create useCancelOrder hook
- [x] Update hooks/index.ts exports (all in use-orders.ts)
- [x] **[HIGH]** Fix timeline sorting (createdAt DESC) in MSW handler
- [x] **[MEDIUM]** Refine cache invalidation in useCancelOrder hook
- [ ] Test endpoints via browser DevTools

## Success Criteria

- GET /timeline returns events array sorted by createdAt desc
- PATCH updates notes and returns updated order
- POST /cancel returns 400 for shipped/completed orders
- All hooks work with TanStack Query

## Issues Found & Fixed (Code Review 2026-01-22)

### High Priority ✅ Fixed
1. **Timeline events not sorted by createdAt DESC** - Added explicit sort in MSW handler

### Medium Priority ✅ Fixed
2. **Inconsistent cache invalidation** - Refined `useCancelOrder` to invalidate detail, timeline, and lists()

See full review: [reports/code-reviewer-260122-phase-02-api-endpoints.md](reports/code-reviewer-260122-phase-02-api-endpoints.md)

## Next Phase

→ [Phase 03: Order Summary Card](phase-03-order-summary-card.md)
