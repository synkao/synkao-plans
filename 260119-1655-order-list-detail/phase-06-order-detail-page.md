# Phase 6: Order Detail Page

## Context Links

- Spec: `docs/frontend/synkao-frontend-spec.md` section 3.2
- Pattern: Task detail drawer structure
- Routing: `apps/web/src/app/(main)/orders/[id]/page.tsx`
- Components: Phase 5 order-detail components
- Hooks: Phase 1 `useOrder()`, `useOrderItems()`

## Overview

| Field | Value |
|-------|-------|
| Priority | P1 |
| Status | pending |
| Effort | 1h |
| Date | 2026-01-19 |

Create Order Detail page with routing, data fetching, and component integration.

## Key Insights

- Use dynamic route `[id]` for order ID
- Two-column grid layout: left (customer + items), right (status + timeline)
- Back button returns to order list
- "Create Fulfillment" action in header

## Requirements

### Functional
- Fetch order by ID
- Fetch order items
- Display CustomerInfoCard, OrderStatusStepper, OrderItemCards, OrderTimeline
- Back navigation to /orders
- Loading and error states

### Non-functional
- Responsive layout (stack on mobile)
- Glass card styling consistent with app

## Architecture

```
app/(main)/orders/
├── page.tsx           # List page (Phase 4)
└── [id]/
    └── page.tsx       # Detail page (this phase)
```

## Related Code Files

**To Create:**
- `apps/web/src/app/(main)/orders/[id]/page.tsx`

**To Use:**
- `useOrder(id)`, `useOrderItems(orderId)` from Phase 1
- `CustomerInfoCard`, `OrderStatusStepper`, `OrderItemCard`, `OrderTimeline` from Phase 5

## Implementation Steps

### 1. Create Order Detail Page

**File:** `apps/web/src/app/(main)/orders/[id]/page.tsx`

```typescript
'use client';

import { use } from 'react';
import { useRouter } from 'next/navigation';
import { toast } from 'sonner';
import { PageHeader } from '@/components/layout';
import { Button } from '@/components/ui/button';
import { ArrowLeft, Truck } from 'lucide-react';
import {
  useOrder,
  useOrderItems,
  CustomerInfoCard,
  OrderStatusStepper,
  OrderItemCard,
  OrderTimeline,
  type MockOrderItem,
  type TimelineEvent,
} from '@/features/orders';

interface OrderDetailPageProps {
  params: Promise<{ id: string }>;
}

export default function OrderDetailPage({ params }: OrderDetailPageProps) {
  const { id } = use(params);
  const router = useRouter();

  // Fetch order and items
  const { data: order, isLoading: orderLoading, error: orderError } = useOrder(id);
  const { data: itemsData, isLoading: itemsLoading } = useOrderItems(id);

  const items = itemsData?.items ?? [];
  const isLoading = orderLoading || itemsLoading;

  // Generate mock timeline events from order data
  const timelineEvents = order ? generateTimelineEvents(order) : [];

  // Handle actions
  const handleBack = () => {
    router.push('/orders');
  };

  const handleCreateFulfillment = () => {
    toast.info('Create fulfillment modal (coming soon)');
  };

  const handleViewDesign = (item: MockOrderItem) => {
    toast.info(`View design for ${item.productName} (coming soon)`);
  };

  const handleViewTask = (item: MockOrderItem) => {
    toast.info(`View task for ${item.productName} (coming soon)`);
  };

  // Loading state
  if (isLoading) {
    return (
      <>
        <PageHeader
          title="Order Detail"
          description="Loading..."
          breadcrumbs={[
            { label: 'Dashboard', href: '/' },
            { label: 'Orders', href: '/orders' },
            { label: 'Loading...' },
          ]}
          actions={
            <Button variant="outline" onClick={handleBack}>
              <ArrowLeft className="mr-2 h-4 w-4" />
              Back
            </Button>
          }
        />
        <div className="flex items-center justify-center py-12">
          <p className="text-muted-foreground">Loading order...</p>
        </div>
      </>
    );
  }

  // Error state
  if (orderError || !order) {
    return (
      <>
        <PageHeader
          title="Order Not Found"
          description="The requested order could not be found"
          breadcrumbs={[
            { label: 'Dashboard', href: '/' },
            { label: 'Orders', href: '/orders' },
            { label: 'Error' },
          ]}
          actions={
            <Button variant="outline" onClick={handleBack}>
              <ArrowLeft className="mr-2 h-4 w-4" />
              Back to Orders
            </Button>
          }
        />
        <div className="flex items-center justify-center py-12">
          <p className="text-destructive">
            {orderError?.message || 'Order not found'}
          </p>
        </div>
      </>
    );
  }

  return (
    <>
      <PageHeader
        title={order.orderNumber}
        description={`Order from ${order.customerName}`}
        breadcrumbs={[
          { label: 'Dashboard', href: '/' },
          { label: 'Orders', href: '/orders' },
          { label: order.orderNumber },
        ]}
        actions={
          <div className="flex gap-2">
            <Button variant="outline" onClick={handleBack}>
              <ArrowLeft className="mr-2 h-4 w-4" />
              Back
            </Button>
            <Button onClick={handleCreateFulfillment}>
              <Truck className="mr-2 h-4 w-4" />
              Create Fulfillment
            </Button>
          </div>
        }
      />

      {/* Two-column layout */}
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        {/* Left column: Customer Info + Items */}
        <div className="lg:col-span-2 space-y-6">
          {/* Customer Info */}
          <CustomerInfoCard order={order} />

          {/* Items */}
          <div className="space-y-4">
            <h3 className="text-sm font-medium text-gray-500 uppercase tracking-wider">
              Items ({items.length})
            </h3>
            {items.map((item) => (
              <OrderItemCard
                key={item.id}
                item={item}
                onViewDesign={handleViewDesign}
                onViewTask={handleViewTask}
              />
            ))}
            {items.length === 0 && (
              <p className="text-sm text-gray-500">No items found</p>
            )}
          </div>
        </div>

        {/* Right column: Status + Timeline */}
        <div className="space-y-6">
          {/* Order Status */}
          <OrderStatusStepper status={order.status} />

          {/* Timeline */}
          <OrderTimeline events={timelineEvents} />
        </div>
      </div>
    </>
  );
}

/**
 * Generate timeline events from order data
 * In real app, this would come from API
 */
function generateTimelineEvents(order: { status: string; createdAt: string }): TimelineEvent[] {
  const events: TimelineEvent[] = [];
  const createdDate = new Date(order.createdAt);

  // Order created event
  events.push({
    id: 'created',
    timestamp: createdDate.toISOString(),
    description: 'Order was created',
    type: 'info',
  });

  // Add status-based events
  const statusEvents: Record<string, { desc: string; type: TimelineEvent['type']; hoursAfter: number }> = {
    CONFIRMED: { desc: 'Order confirmed', type: 'success', hoursAfter: 1 },
    IN_DESIGN: { desc: 'Design work started', type: 'info', hoursAfter: 2 },
    READY_TO_FULFILL: { desc: 'Design approved, ready for fulfillment', type: 'success', hoursAfter: 24 },
    SHIPPED: { desc: 'Order shipped', type: 'success', hoursAfter: 48 },
    COMPLETED: { desc: 'Order completed', type: 'success', hoursAfter: 72 },
    CANCELLED: { desc: 'Order cancelled', type: 'error', hoursAfter: 1 },
  };

  // Determine which events to show based on current status
  const statusOrder = ['CONFIRMED', 'IN_DESIGN', 'READY_TO_FULFILL', 'SHIPPED', 'COMPLETED'];
  const currentIdx = statusOrder.indexOf(order.status);

  if (order.status === 'CANCELLED') {
    const eventConfig = statusEvents.CANCELLED;
    events.push({
      id: 'cancelled',
      timestamp: new Date(createdDate.getTime() + eventConfig.hoursAfter * 60 * 60 * 1000).toISOString(),
      description: eventConfig.desc,
      type: eventConfig.type,
    });
  } else if (currentIdx >= 0) {
    for (let i = 0; i <= currentIdx; i++) {
      const status = statusOrder[i];
      const eventConfig = statusEvents[status];
      if (eventConfig) {
        events.push({
          id: status.toLowerCase(),
          timestamp: new Date(createdDate.getTime() + eventConfig.hoursAfter * 60 * 60 * 1000).toISOString(),
          description: eventConfig.desc,
          type: eventConfig.type,
        });
      }
    }
  }

  // Sort by timestamp descending (newest first)
  return events.sort((a, b) => new Date(b.timestamp).getTime() - new Date(a.timestamp).getTime());
}
```

## Todo List

- [ ] Create `apps/web/src/app/(main)/orders/[id]/page.tsx`
- [ ] Test navigation from order list to detail
- [ ] Test back button navigation
- [ ] Test loading state
- [ ] Test error state (invalid order ID)
- [ ] Test responsive layout on mobile
- [ ] Verify all components render correctly

## Success Criteria

- Page loads with correct order data
- Customer info displays all fields
- Status stepper shows correct step
- Items display as cards with correct info
- Timeline shows events in reverse chronological order
- Back button returns to order list
- Loading and error states work correctly

## Risk Assessment

| Risk | Mitigation |
|------|------------|
| Order not found (invalid ID) | Show error state with back button |
| Items query fails | Show empty state, order still displays |
| Timeline events not realistic | Use mock generator, enhance in Phase 7 |

## Security Considerations

- Order ID comes from URL params (validated by API)
- No user input handling beyond navigation

## Next Steps

Proceed to Phase 7: MSW Mock Enhancement (add timeline endpoint, enhance orders handler)
