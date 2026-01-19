# Phase 7: MSW Mock Enhancement

## Context Links

- Existing handlers: `apps/web/src/mocks/handlers/orders.ts`
- Existing data: `apps/web/src/mocks/data/orders.data.ts`, `order-items.data.ts`
- Response helpers: `apps/web/src/mocks/utils/response-helpers.ts`
- Mock factory: `apps/web/src/mocks/factories/mock-factory.ts`

## Overview

| Field | Value |
|-------|-------|
| Priority | P2 |
| Status | pending |
| Effort | 0.5h |
| Date | 2026-01-19 |

Enhance MSW handlers to support additional filters and add timeline endpoint.

## Key Insights

- Current handlers support: status, search, from, to filters
- Need to add: design_status, fulfiller_name filters
- Optional: Add timeline endpoint for order events
- Keep changes minimal - only what's needed for Order List/Detail pages

## Requirements

### Functional
- Add `design_status` filter to GET /api/v1/orders/
- Add `fulfiller_name` filter to GET /api/v1/orders/
- (Optional) Add GET /api/v1/orders/:id/timeline endpoint

### Non-functional
- Follow existing MSW handler patterns
- Maintain response format consistency

## Architecture

```
mocks/
├── handlers/
│   └── orders.ts         # Update with new filters + timeline
├── data/
│   └── timeline-events.data.ts  # (Optional) Timeline mock data
```

## Related Code Files

**To Update:**
- `apps/web/src/mocks/handlers/orders.ts`

**To Create (Optional):**
- `apps/web/src/mocks/data/order-timeline.data.ts`

**Reference:**
- `apps/web/src/mocks/utils/response-helpers.ts`
- `apps/web/src/mocks/data/orders.data.ts`

## Implementation Steps

### 1. Update Orders Handler with Additional Filters

**File:** `apps/web/src/mocks/handlers/orders.ts`

```typescript
import { http } from 'msw';
import { mockOrders, findOrder, findOrderItems } from '../data';
import {
  successResponse,
  listResponse,
  parsePagination,
  getPaginationMeta,
  paginateArray,
  errors,
} from '../utils';

export const ordersHandlers = [
  // GET /api/v1/orders/
  http.get('/api/v1/orders/', ({ request }) => {
    const url = new URL(request.url);
    const { page, limit } = parsePagination(url);
    const status = url.searchParams.get('status');
    const designStatus = url.searchParams.get('design_status');
    const fulfillerName = url.searchParams.get('fulfiller_name');
    const search = url.searchParams.get('search');
    const from = url.searchParams.get('from');
    const to = url.searchParams.get('to');

    let filtered = [...mockOrders];

    // Filter by order status
    if (status) {
      filtered = filtered.filter((o) => o.status === status);
    }

    // Filter by design status
    if (designStatus) {
      filtered = filtered.filter((o) => o.designStatus === designStatus);
    }

    // Filter by fulfiller name
    if (fulfillerName) {
      filtered = filtered.filter((o) =>
        o.fulfillerName?.toLowerCase().includes(fulfillerName.toLowerCase())
      );
    }

    // Search by order number or customer name
    if (search) {
      const term = search.toLowerCase();
      filtered = filtered.filter(
        (o) =>
          o.orderNumber.toLowerCase().includes(term) ||
          o.customerName.toLowerCase().includes(term)
      );
    }

    // Filter by date range
    if (from) {
      filtered = filtered.filter((o) => new Date(o.createdAt) >= new Date(from));
    }
    if (to) {
      filtered = filtered.filter((o) => new Date(o.createdAt) <= new Date(to));
    }

    const paginated = paginateArray(filtered, page, limit);
    const pagination = getPaginationMeta(filtered.length, page, limit);

    return listResponse(paginated, pagination);
  }),

  // GET /api/v1/orders/:id
  http.get('/api/v1/orders/:id', ({ params }) => {
    const order = findOrder(params.id as string);

    if (!order) {
      return errors.notFound('Order');
    }

    return successResponse(order);
  }),

  // GET /api/v1/orders/:id/items
  http.get('/api/v1/orders/:id/items', ({ params }) => {
    const order = findOrder(params.id as string);
    if (!order) {
      return errors.notFound('Order');
    }

    const items = findOrderItems(params.id as string);
    return successResponse({ items, total: items.length });
  }),

  // GET /api/v1/orders/:id/timeline
  http.get('/api/v1/orders/:id/timeline', ({ params }) => {
    const order = findOrder(params.id as string);
    if (!order) {
      return errors.notFound('Order');
    }

    // Generate timeline events based on order status
    const events = generateOrderTimeline(order);
    return successResponse({ events });
  }),

  // POST /api/v1/orders/import/validate
  http.post('/api/v1/orders/import/validate', async () => {
    return successResponse({
      valid: [
        { row: 1, order_number: 'ORD-001', customer: 'John Doe' },
        { row: 2, order_number: 'ORD-002', customer: 'Jane Smith' },
      ],
      warnings: [{ row: 3, field: 'phone', message: 'Phone format may be invalid' }],
      errors: [],
    });
  }),

  // POST /api/v1/orders/import/execute
  http.post('/api/v1/orders/import/execute', async () => {
    return successResponse(
      {
        created_orders: 5,
        created_tasks: 8,
        message: 'Import successful',
      },
      201
    );
  }),
];

/**
 * Generate timeline events for an order based on its status
 */
function generateOrderTimeline(order: {
  id: string;
  status: string;
  createdAt: string;
  customerName: string;
}) {
  const events = [];
  const createdDate = new Date(order.createdAt);
  let eventId = 1;

  // Base event: order created
  events.push({
    id: `event_${order.id}_${eventId++}`,
    timestamp: createdDate.toISOString(),
    description: 'Order was created',
    type: 'info',
  });

  // Status-based events with timestamps
  const statusFlow = [
    { status: 'CONFIRMED', desc: 'Order confirmed', hoursAfter: 1, type: 'success' },
    { status: 'IN_DESIGN', desc: 'Design work started', hoursAfter: 2, type: 'info' },
    { status: 'READY_TO_FULFILL', desc: 'Design approved, ready for fulfillment', hoursAfter: 24, type: 'success' },
    { status: 'SHIPPED', desc: `Order shipped to ${order.customerName}`, hoursAfter: 48, type: 'success' },
    { status: 'COMPLETED', desc: 'Order completed', hoursAfter: 72, type: 'success' },
  ];

  const statusOrder = statusFlow.map((s) => s.status);
  const currentIdx = statusOrder.indexOf(order.status);

  // Handle cancelled status
  if (order.status === 'CANCELLED') {
    events.push({
      id: `event_${order.id}_${eventId++}`,
      timestamp: new Date(createdDate.getTime() + 1 * 60 * 60 * 1000).toISOString(),
      description: 'Order cancelled',
      type: 'error',
    });
  } else if (currentIdx >= 0) {
    // Add events up to current status
    for (let i = 0; i <= currentIdx; i++) {
      const step = statusFlow[i];
      events.push({
        id: `event_${order.id}_${eventId++}`,
        timestamp: new Date(createdDate.getTime() + step.hoursAfter * 60 * 60 * 1000).toISOString(),
        description: step.desc,
        type: step.type,
      });
    }
  }

  // Sort descending (newest first)
  return events.sort((a, b) => new Date(b.timestamp).getTime() - new Date(a.timestamp).getTime());
}
```

### 2. (Optional) Add useOrderTimeline hook

If using the timeline endpoint instead of client-side generation:

**File:** `features/orders/hooks/use-orders.ts` (add to existing file)

```typescript
/**
 * Hook to fetch order timeline events
 */
export function useOrderTimeline(orderId: string) {
  return useQuery({
    queryKey: ['orders', 'timeline', orderId],
    queryFn: async () => {
      const response = await fetch(`/api/v1/orders/${orderId}/timeline`);
      if (!response.ok) throw new Error('Failed to fetch timeline');
      const result = await response.json();
      return result.data.events as TimelineEvent[];
    },
    enabled: !!orderId,
  });
}
```

### 3. Update data exports (if needed)

**File:** `apps/web/src/mocks/data/index.ts`

Ensure `findOrder` and `findOrderItems` are exported:

```typescript
export { mockOrders, findOrder, findOrderByNumber } from './orders.data';
export { mockOrderItems, findOrderItems, findOrderItem } from './order-items.data';
// ... other exports
```

## Todo List

- [ ] Update `mocks/handlers/orders.ts` with design_status filter
- [ ] Update `mocks/handlers/orders.ts` with fulfiller_name filter
- [ ] Add GET /api/v1/orders/:id/timeline endpoint
- [ ] Test filters work in Order List page
- [ ] Test timeline endpoint returns correct data
- [ ] Verify pagination still works with filters

## Success Criteria

- Orders can be filtered by design_status
- Orders can be filtered by fulfiller_name
- Timeline endpoint returns status-based events
- No regressions in existing order endpoints

## Risk Assessment

| Risk | Mitigation |
|------|------------|
| Breaking existing handlers | Keep changes additive, test existing flows |
| Timeline data inconsistent | Generate from order status, not stored data |

## Security Considerations

- No authentication changes needed (MSW mock)
- Input filters sanitized by URL parsing

## Next Steps

All phases complete. Run full integration test:
1. Navigate to /orders
2. Test filters and pagination
3. Click order to go to detail
4. Verify all components render
5. Test back navigation

## Summary

This phase completes the Order List & Detail implementation plan with:
- Additional filter support in MSW handlers
- Timeline endpoint for order events
- Full integration path from list to detail
