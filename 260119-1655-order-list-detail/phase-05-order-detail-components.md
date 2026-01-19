# Phase 5: Order Detail Components

## Context Links

- Spec: `docs/frontend/synkao-frontend-spec.md` section 3.2
- Pattern: `apps/web/src/features/design/components/drawer/task-info-section.tsx`
- Pattern: `apps/web/src/features/design/components/drawer/timeline-section.tsx`
- Constants: Phase 1 `features/orders/lib/constants.ts` (ORDER_STATUS_STEPS)

## Overview

| Field | Value |
|-------|-------|
| Priority | P1 |
| Status | pending |
| Effort | 1.5h |
| Date | 2026-01-19 |

Create components for Order Detail page: customer info, status stepper, item cards, timeline.

## Key Insights

From spec wireframe section 3.2:
- Two-column layout: Customer Info + Order Status
- Items list as cards (not table rows)
- Timeline shows chronological events
- Actions: "Xem Design" opens Task Drawer, "Tạo Fulfillment" modal

## Requirements

### Functional
- `CustomerInfoCard`: Name, email, phone, address
- `OrderStatusStepper`: 4-step progress (Mới → Design → Ready → Done)
- `OrderItemCard`: Thumbnail, name, SKU, qty, design status, designer, notes
- `OrderTimeline`: Chronological event list

### Non-functional
- Cards use glass effect styling
- Stepper shows current step highlighted
- Timeline in reverse chronological order

## Architecture

```
components/order-detail/
├── customer-info-card.tsx
├── order-status-stepper.tsx
├── order-item-card.tsx
├── order-timeline.tsx
└── index.ts
```

## Related Code Files

**To Create:**
- `features/orders/components/order-detail/customer-info-card.tsx`
- `features/orders/components/order-detail/order-status-stepper.tsx`
- `features/orders/components/order-detail/order-item-card.tsx`
- `features/orders/components/order-detail/order-timeline.tsx`
- `features/orders/components/order-detail/index.ts`

**Reference:**
- `features/design/components/drawer/timeline-section.tsx`
- `components/ui/card.tsx`

## Implementation Steps

### 1. Create CustomerInfoCard

**File:** `features/orders/components/order-detail/customer-info-card.tsx`

```typescript
'use client';

import type { MockOrder } from '../../types';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { User, Mail, Phone, MapPin } from 'lucide-react';

export interface CustomerInfoCardProps {
  order: MockOrder;
}

/**
 * Card displaying customer information
 * Shows: name, email, phone, shipping address
 */
export function CustomerInfoCard({ order }: CustomerInfoCardProps) {
  const address = order.shippingAddress;
  const addressLines = address
    ? [
        address.line1,
        address.line2,
        `${address.city}${address.state ? `, ${address.state}` : ''} ${address.postalCode || ''}`,
        address.country,
      ].filter(Boolean)
    : [];

  return (
    <Card className="border border-gray-200/50 bg-white/60 backdrop-blur-md">
      <CardHeader className="pb-3">
        <CardTitle className="text-sm font-medium text-gray-500 uppercase tracking-wider">
          Customer Info
        </CardTitle>
      </CardHeader>
      <CardContent className="space-y-3">
        {/* Name */}
        <div className="flex items-start gap-3">
          <User className="h-4 w-4 text-gray-400 mt-0.5" />
          <span className="text-gray-900 font-medium">{order.customerName}</span>
        </div>

        {/* Email */}
        {order.customerEmail && (
          <div className="flex items-start gap-3">
            <Mail className="h-4 w-4 text-gray-400 mt-0.5" />
            <a
              href={`mailto:${order.customerEmail}`}
              className="text-gray-600 hover:text-blue-600"
            >
              {order.customerEmail}
            </a>
          </div>
        )}

        {/* Phone */}
        {order.customerPhone && (
          <div className="flex items-start gap-3">
            <Phone className="h-4 w-4 text-gray-400 mt-0.5" />
            <a
              href={`tel:${order.customerPhone}`}
              className="text-gray-600 hover:text-blue-600"
            >
              {order.customerPhone}
            </a>
          </div>
        )}

        {/* Address */}
        {addressLines.length > 0 && (
          <div className="flex items-start gap-3">
            <MapPin className="h-4 w-4 text-gray-400 mt-0.5" />
            <div className="text-gray-600">
              {addressLines.map((line, idx) => (
                <div key={idx}>{line}</div>
              ))}
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

### 2. Create OrderStatusStepper

**File:** `features/orders/components/order-detail/order-status-stepper.tsx`

```typescript
'use client';

import type { OrderStatusType } from '../../types';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { ORDER_STATUS_STEPS, ORDER_STATUS_CONFIG } from '../../lib/constants';
import { cn } from '@/lib/utils';
import { Check } from 'lucide-react';

export interface OrderStatusStepperProps {
  status: OrderStatusType;
}

/**
 * Horizontal stepper showing order progress
 * Steps: Mới (PENDING/CONFIRMED) → Design → Ready → Done
 */
export function OrderStatusStepper({ status }: OrderStatusStepperProps) {
  // Map current status to step index
  const statusToStepIndex: Record<OrderStatusType, number> = {
    PENDING: 0,
    CONFIRMED: 0,
    IN_DESIGN: 1,
    READY_TO_FULFILL: 2,
    SHIPPED: 3,
    COMPLETED: 3,
    CANCELLED: -1,
  };

  const currentStepIndex = statusToStepIndex[status];
  const isCancelled = status === 'CANCELLED';

  return (
    <Card className="border border-gray-200/50 bg-white/60 backdrop-blur-md">
      <CardHeader className="pb-3">
        <CardTitle className="text-sm font-medium text-gray-500 uppercase tracking-wider">
          Order Status
        </CardTitle>
      </CardHeader>
      <CardContent>
        {isCancelled ? (
          <div className="flex items-center justify-center py-4">
            <span className="px-3 py-1 rounded-full bg-red-50 text-red-600 font-medium">
              Cancelled
            </span>
          </div>
        ) : (
          <>
            {/* Current status badge */}
            <div className="flex justify-center mb-4">
              <span
                className={cn(
                  'px-3 py-1 rounded-full font-medium text-sm',
                  ORDER_STATUS_CONFIG[status].bgClass,
                  ORDER_STATUS_CONFIG[status].textClass
                )}
              >
                {ORDER_STATUS_CONFIG[status].label}
              </span>
            </div>

            {/* Stepper */}
            <div className="flex items-center justify-between">
              {ORDER_STATUS_STEPS.map((step, idx) => {
                const isCompleted = idx < currentStepIndex;
                const isCurrent = idx === currentStepIndex;

                return (
                  <div key={step.status} className="flex items-center">
                    {/* Step circle */}
                    <div className="flex flex-col items-center">
                      <div
                        className={cn(
                          'w-8 h-8 rounded-full flex items-center justify-center border-2 transition-colors',
                          isCompleted && 'bg-emerald-500 border-emerald-500 text-white',
                          isCurrent && 'bg-blue-500 border-blue-500 text-white',
                          !isCompleted && !isCurrent && 'bg-white border-gray-300 text-gray-400'
                        )}
                      >
                        {isCompleted ? (
                          <Check className="h-4 w-4" />
                        ) : (
                          <span className="text-xs font-medium">{idx + 1}</span>
                        )}
                      </div>
                      <span
                        className={cn(
                          'mt-1 text-xs',
                          isCurrent ? 'text-blue-600 font-medium' : 'text-gray-500'
                        )}
                      >
                        {step.label}
                      </span>
                    </div>

                    {/* Connector line */}
                    {idx < ORDER_STATUS_STEPS.length - 1 && (
                      <div
                        className={cn(
                          'flex-1 h-0.5 mx-2',
                          idx < currentStepIndex ? 'bg-emerald-500' : 'bg-gray-200'
                        )}
                        style={{ minWidth: '40px' }}
                      />
                    )}
                  </div>
                );
              })}
            </div>
          </>
        )}
      </CardContent>
    </Card>
  );
}
```

### 3. Create OrderItemCard

**File:** `features/orders/components/order-detail/order-item-card.tsx`

```typescript
'use client';

import type { MockOrderItem } from '../../types';
import { Card, CardContent } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { ItemDesignStatusBadge } from '../shared';
import { ArrowRight } from 'lucide-react';

export interface OrderItemCardProps {
  item: MockOrderItem;
  designerName?: string;
  onViewDesign?: (item: MockOrderItem) => void;
  onViewTask?: (item: MockOrderItem) => void;
}

/**
 * Card displaying order item details
 * Shows: thumbnail, name, SKU, qty, design status, designer, notes
 */
export function OrderItemCard({
  item,
  designerName,
  onViewDesign,
  onViewTask,
}: OrderItemCardProps) {
  const isDesignApproved = item.designStatus === 'APPROVED';
  const hasDesign = item.designStatus && item.designStatus !== 'NOT_REQUIRED';

  return (
    <Card className="border border-gray-200/50 bg-white/60 backdrop-blur-md">
      <CardContent className="p-4">
        <div className="flex gap-4">
          {/* Thumbnail */}
          <div className="h-20 w-20 flex-shrink-0 overflow-hidden rounded-lg border border-gray-200 bg-gray-100">
            {item.productImageUrl ? (
              <img
                src={item.productImageUrl}
                alt={item.productName}
                className="h-full w-full object-cover"
              />
            ) : (
              <div className="flex h-full w-full items-center justify-center text-gray-400">
                IMG
              </div>
            )}
          </div>

          {/* Content */}
          <div className="flex-1 min-w-0">
            {/* Header: Name + Qty */}
            <div className="flex items-start justify-between gap-2">
              <div>
                <h4 className="font-medium text-gray-900">{item.productName}</h4>
                <p className="text-sm text-gray-500">SKU: {item.itemCode}</p>
              </div>
              <span className="text-sm font-medium text-gray-600">x{item.quantity}</span>
            </div>

            {/* Design Status + Designer */}
            <div className="mt-2 flex items-center gap-3">
              <ItemDesignStatusBadge status={item.designStatus} size="md" />
              {designerName && (
                <span className="text-sm text-gray-500">
                  Designer: <span className="text-gray-700">{designerName}</span>
                </span>
              )}
            </div>

            {/* Notes */}
            {item.designNotes && (
              <p className="mt-2 text-sm text-gray-600 line-clamp-2">{item.designNotes}</p>
            )}

            {/* Actions */}
            {hasDesign && (
              <div className="mt-3 flex gap-2">
                {isDesignApproved && onViewDesign && (
                  <Button
                    variant="outline"
                    size="sm"
                    onClick={() => onViewDesign(item)}
                  >
                    View Design
                    <ArrowRight className="ml-1 h-3 w-3" />
                  </Button>
                )}
                {!isDesignApproved && onViewTask && (
                  <Button
                    variant="outline"
                    size="sm"
                    onClick={() => onViewTask(item)}
                  >
                    View Task
                    <ArrowRight className="ml-1 h-3 w-3" />
                  </Button>
                )}
              </div>
            )}
          </div>
        </div>
      </CardContent>
    </Card>
  );
}
```

### 4. Create OrderTimeline

**File:** `features/orders/components/order-detail/order-timeline.tsx`

```typescript
'use client';

import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { cn } from '@/lib/utils';

export interface TimelineEvent {
  id: string;
  timestamp: string;
  description: string;
  type: 'info' | 'success' | 'warning' | 'error';
}

export interface OrderTimelineProps {
  events: TimelineEvent[];
}

const EVENT_STYLES: Record<TimelineEvent['type'], { dot: string; text: string }> = {
  info: { dot: 'bg-blue-500', text: 'text-gray-600' },
  success: { dot: 'bg-emerald-500', text: 'text-gray-600' },
  warning: { dot: 'bg-amber-500', text: 'text-gray-600' },
  error: { dot: 'bg-red-500', text: 'text-red-600' },
};

/**
 * Timeline component showing order events
 * Events in reverse chronological order (newest first)
 */
export function OrderTimeline({ events }: OrderTimelineProps) {
  if (events.length === 0) {
    return (
      <Card className="border border-gray-200/50 bg-white/60 backdrop-blur-md">
        <CardHeader className="pb-3">
          <CardTitle className="text-sm font-medium text-gray-500 uppercase tracking-wider">
            Timeline
          </CardTitle>
        </CardHeader>
        <CardContent>
          <p className="text-sm text-gray-500">No events yet</p>
        </CardContent>
      </Card>
    );
  }

  return (
    <Card className="border border-gray-200/50 bg-white/60 backdrop-blur-md">
      <CardHeader className="pb-3">
        <CardTitle className="text-sm font-medium text-gray-500 uppercase tracking-wider">
          Timeline
        </CardTitle>
      </CardHeader>
      <CardContent>
        <div className="relative">
          {/* Vertical line */}
          <div className="absolute left-[7px] top-2 bottom-2 w-0.5 bg-gray-200" />

          {/* Events */}
          <div className="space-y-4">
            {events.map((event, idx) => {
              const styles = EVENT_STYLES[event.type];
              const isLast = idx === events.length - 1;

              return (
                <div key={event.id} className="relative flex gap-3">
                  {/* Dot */}
                  <div
                    className={cn(
                      'relative z-10 h-4 w-4 rounded-full border-2 border-white',
                      styles.dot,
                      isLast && 'ring-2 ring-gray-100'
                    )}
                  />

                  {/* Content */}
                  <div className="flex-1 pb-2">
                    <time className="text-xs text-gray-400">
                      {formatEventTime(event.timestamp)}
                    </time>
                    <p className={cn('text-sm', styles.text)}>{event.description}</p>
                  </div>
                </div>
              );
            })}
          </div>
        </div>
      </CardContent>
    </Card>
  );
}

function formatEventTime(timestamp: string): string {
  const date = new Date(timestamp);
  const now = new Date();
  const isToday = date.toDateString() === now.toDateString();

  const time = date.toLocaleTimeString('en-US', {
    hour: '2-digit',
    minute: '2-digit',
    hour12: false,
  });

  if (isToday) {
    return time;
  }

  return `${date.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })} ${time}`;
}
```

### 5. Create index file

**File:** `features/orders/components/order-detail/index.ts`

```typescript
export { CustomerInfoCard } from './customer-info-card';
export type { CustomerInfoCardProps } from './customer-info-card';

export { OrderStatusStepper } from './order-status-stepper';
export type { OrderStatusStepperProps } from './order-status-stepper';

export { OrderItemCard } from './order-item-card';
export type { OrderItemCardProps } from './order-item-card';

export { OrderTimeline } from './order-timeline';
export type { OrderTimelineProps, TimelineEvent } from './order-timeline';
```

### 6. Update features/orders/index.ts

Add export for order-detail components:

```typescript
// ... existing exports
export * from './components/order-detail';
```

## Todo List

- [ ] Create `features/orders/components/order-detail/customer-info-card.tsx`
- [ ] Create `features/orders/components/order-detail/order-status-stepper.tsx`
- [ ] Create `features/orders/components/order-detail/order-item-card.tsx`
- [ ] Create `features/orders/components/order-detail/order-timeline.tsx`
- [ ] Create `features/orders/components/order-detail/index.ts`
- [ ] Update `features/orders/index.ts` with new exports
- [ ] Visual test each component in isolation

## Success Criteria

- CustomerInfoCard shows all customer fields
- OrderStatusStepper highlights correct step based on status
- OrderItemCard shows thumbnail, info, badge, actions
- OrderTimeline renders events with correct styling
- Cancelled orders show cancelled state in stepper

## Risk Assessment

| Risk | Mitigation |
|------|------------|
| Timeline events not in mock data | Add mock events in Phase 7 or use empty state |
| Designer name not available | Show "—" if not available |
| Image loading errors | Fallback placeholder in thumbnails |

## Security Considerations

- Email/phone links use proper `mailto:` and `tel:` protocols
- No user input handling
- Image URLs from trusted mock data

## Next Steps

Proceed to Phase 6: Order Detail Page (routing, integration, data fetching)
