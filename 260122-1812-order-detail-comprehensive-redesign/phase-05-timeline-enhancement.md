# Phase 05: UI - Timeline Enhancement

**Priority:** P1 - High
**Status:** Complete ✅
**Completed:** 2026-01-22 19:37
**Depends on:** Phase 01, Phase 02

## Overview

Enhance OrderTimeline to display rich order events with user attribution, event icons, and proper styling for 6+ event types.

## Related Files

**Modify:**
- `apps/web/src/features/orders/components/order-detail/order-timeline.tsx`
- `apps/web/src/features/orders/types/order.types.ts` - Update TimelineEvent
- `apps/web/src/features/orders/lib/constants.ts` - Add timeline event config
- `apps/web/src/app/(main)/orders/[id]/page.tsx` - Use API timeline

## Current vs Enhanced

| Feature | Current | Enhanced |
|---------|---------|----------|
| Basic events | ✅ Generated from status | ✅ From API |
| Event types | 4 (info/success/warning/error) | 10+ order-specific types |
| User attribution | ❌ | ✅ Shows who did action |
| Event icons | Colored dots | Specific icons per type |
| Timestamps | ✅ | ✅ with relative time |

## Event Types & Styling

```typescript
export const ORDER_TIMELINE_EVENT_CONFIG = {
  ORDER_CREATED: {
    icon: 'Plus',
    color: 'blue',
    bgClass: 'bg-blue-500',
    label: 'Order Created',
  },
  ORDER_CONFIRMED: {
    icon: 'Check',
    color: 'emerald',
    bgClass: 'bg-emerald-500',
    label: 'Order Confirmed',
  },
  DESIGN_STARTED: {
    icon: 'Palette',
    color: 'amber',
    bgClass: 'bg-amber-500',
    label: 'Design Started',
  },
  DESIGN_COMPLETED: {
    icon: 'CheckCircle',
    color: 'emerald',
    bgClass: 'bg-emerald-500',
    label: 'Design Completed',
  },
  READY_TO_FULFILL: {
    icon: 'Package',
    color: 'violet',
    bgClass: 'bg-violet-500',
    label: 'Ready to Fulfill',
  },
  SHIPPED: {
    icon: 'Truck',
    color: 'cyan',
    bgClass: 'bg-cyan-500',
    label: 'Shipped',
  },
  COMPLETED: {
    icon: 'CheckCircle2',
    color: 'emerald',
    bgClass: 'bg-emerald-500',
    label: 'Completed',
  },
  CANCELLED: {
    icon: 'X',
    color: 'red',
    bgClass: 'bg-red-500',
    label: 'Cancelled',
  },
  NOTE_UPDATED: {
    icon: 'Edit',
    color: 'gray',
    bgClass: 'bg-gray-500',
    label: 'Note Updated',
  },
  PAYMENT_RECEIVED: {
    icon: 'DollarSign',
    color: 'emerald',
    bgClass: 'bg-emerald-500',
    label: 'Payment Received',
  },
};
```

## Design

```
┌─────────────────────────────────────┐
│ TIMELINE                            │
├─────────────────────────────────────┤
│ ●─ Design v2 approved              │
│ │   Admin User • 14:30 today        │
│ │                                   │
│ ●─ Designer Minh submitted v2      │
│ │   Designer Minh • 12:15 today     │
│ │                                   │
│ ●─ Design v1 rejected              │
│ │   Admin User • Yesterday 16:00    │
│ │   "Needs brighter colors"         │
│ │                                   │
│ ●─ Task assigned to Designer Minh  │
│ │   Admin User • Jan 18, 10:30      │
│ │                                   │
│ ●─ Order confirmed                 │
│ │   System • Jan 18, 09:00          │
│ │                                   │
│ ●─ Order created                   │
│     WooCommerce • Jan 18, 08:30     │
└─────────────────────────────────────┘
```

## Implementation

### Update TimelineEvent Type

```typescript
// In order.types.ts

export interface OrderTimelineEvent {
  id: string;
  orderId: string;
  type: string; // ORDER_CREATED, DESIGN_STARTED, etc.
  userId: string;
  userName: string;
  description: string;
  metadata?: Record<string, unknown>;
  createdAt: string;
}
```

### Update OrderTimeline Component

```tsx
'use client';

import type { OrderTimelineEvent } from '../../types';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { cn } from '@/lib/utils';
import {
  Plus,
  Check,
  Palette,
  CheckCircle,
  Package,
  Truck,
  X,
  Edit,
  DollarSign,
  Circle,
} from 'lucide-react';
import type { LucideIcon } from 'lucide-react';

export interface OrderTimelineProps {
  events: OrderTimelineEvent[];
  isLoading?: boolean;
}

// Event configuration
const EVENT_CONFIG: Record<string, { Icon: LucideIcon; bgClass: string }> = {
  ORDER_CREATED: { Icon: Plus, bgClass: 'bg-blue-500' },
  ORDER_CONFIRMED: { Icon: Check, bgClass: 'bg-emerald-500' },
  DESIGN_STARTED: { Icon: Palette, bgClass: 'bg-amber-500' },
  DESIGN_COMPLETED: { Icon: CheckCircle, bgClass: 'bg-emerald-500' },
  READY_TO_FULFILL: { Icon: Package, bgClass: 'bg-violet-500' },
  SHIPPED: { Icon: Truck, bgClass: 'bg-cyan-500' },
  COMPLETED: { Icon: CheckCircle, bgClass: 'bg-emerald-500' },
  CANCELLED: { Icon: X, bgClass: 'bg-red-500' },
  NOTE_UPDATED: { Icon: Edit, bgClass: 'bg-gray-500' },
  PAYMENT_RECEIVED: { Icon: DollarSign, bgClass: 'bg-emerald-500' },
};

export function OrderTimeline({ events, isLoading }: OrderTimelineProps) {
  if (isLoading) {
    return (
      <Card className="border border-gray-200/50 bg-white/60 backdrop-blur-md">
        <CardHeader className="pb-3">
          <CardTitle className="text-sm font-medium text-gray-500 uppercase tracking-wider">
            Timeline
          </CardTitle>
        </CardHeader>
        <CardContent>
          <p className="text-sm text-gray-500">Loading...</p>
        </CardContent>
      </Card>
    );
  }

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
          Timeline ({events.length})
        </CardTitle>
      </CardHeader>
      <CardContent>
        <div className="relative">
          {/* Vertical line */}
          <div className="absolute left-[11px] top-3 bottom-3 w-0.5 bg-gray-200" />

          {/* Events */}
          <div className="space-y-4">
            {events.map((event, idx) => {
              const config = EVENT_CONFIG[event.type] ?? {
                Icon: Circle,
                bgClass: 'bg-gray-400',
              };
              const { Icon } = config;
              const isLast = idx === events.length - 1;

              return (
                <div key={event.id} className="relative flex gap-3">
                  {/* Icon */}
                  <div
                    className={cn(
                      'relative z-10 h-6 w-6 rounded-full flex items-center justify-center',
                      config.bgClass,
                      isLast && 'ring-2 ring-white'
                    )}
                  >
                    <Icon className="h-3 w-3 text-white" />
                  </div>

                  {/* Content */}
                  <div className="flex-1 pb-2">
                    <p className="text-sm text-gray-900">{event.description}</p>
                    <div className="flex items-center gap-2 text-xs text-gray-500">
                      <span>{event.userName}</span>
                      <span>•</span>
                      <time>{formatEventTime(event.createdAt)}</time>
                    </div>
                    {/* Show rejection reason or other metadata */}
                    {event.metadata?.reason && (
                      <p className="mt-1 text-xs text-gray-500 italic">
                        "{event.metadata.reason}"
                      </p>
                    )}
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
  const diffMs = now.getTime() - date.getTime();
  const diffMins = Math.floor(diffMs / 60000);
  const diffHours = Math.floor(diffMs / 3600000);
  const diffDays = Math.floor(diffMs / 86400000);

  const time = date.toLocaleTimeString('en-US', {
    hour: '2-digit',
    minute: '2-digit',
    hour12: false,
  });

  if (diffMins < 60) {
    return diffMins <= 1 ? 'Just now' : `${diffMins}m ago`;
  }
  if (diffHours < 24) {
    return `${time} today`;
  }
  if (diffDays === 1) {
    return `Yesterday ${time}`;
  }
  return `${date.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })} ${time}`;
}
```

### Update page.tsx

```tsx
// Import hook
import { useOrderTimeline } from '@/features/orders';

// In component
const { data: timelineData, isLoading: timelineLoading } = useOrderTimeline(id);

// Use API events instead of generated
<OrderTimeline
  events={timelineData?.events ?? []}
  isLoading={timelineLoading}
/>
```

## Todo

- [x] Update OrderTimelineEvent type with userName, eventType
- [x] Add ORDER_TIMELINE_EVENT_CONFIG constants
- [x] Update OrderTimeline component with icons
- [x] Add user attribution display
- [x] Add metadata display (rejection reason, etc.)
- [x] Update formatEventTime with relative time
- [x] Integrate useOrderTimeline hook in page.tsx
- [x] Test with 10 event types
- [x] Verify loading state

## Code Review

**Report:** `reports/code-reviewer-260122-1933-phase05-timeline-enhancement.md`
**Score:** 9/10 (improved from 8/10)
**Status:** Approved

**Improvements Made:**
- Consolidated EVENT_CONFIG using ORDER_TIMELINE_EVENT_CONFIG constant in lib/constants.ts (fixed DRY violation)
- Added sanitizeDisplayText() for XSS protection on metadata/reason fields
- Centralized event type definitions

**Critical issues:** None resolved

**Follow-up improvements:**
- Memoize formatted times
- Add timezone awareness
- Strengthen metadata typing
- Add ARIA labels for accessibility

## Success Criteria

- Each event type has distinct icon
- User name displayed for each event
- Relative timestamps (today, yesterday, date)
- Rejection reasons shown as quotes
- Loading state works
- Count shown in header

## Next Phase

→ [Phase 06: Header Actions](phase-06-header-actions.md)
