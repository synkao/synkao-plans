# Phase 01: Data Layer - Types & Mock Data

**Priority:** P0 - Critical
**Status:** ✅ Completed (2026-01-22)
**Review:** [Code Review Report](reports/code-reviewer-260122-1824-phase-01-data-layer.md)

## Overview

Update data types và mock data để support tất cả fields mới cho Order Detail page.

## Related Files

**Modify:**
- `apps/web/src/mocks/types.ts` - Add new type fields
- `apps/web/src/mocks/data/orders.data.ts` - Generate payment, tip, donation
- `apps/web/src/mocks/data/order-items.data.ts` - Generate designStatus, attributes, personalization
- `apps/web/src/mocks/data/index.ts` - Export new data
- `apps/web/src/features/orders/types/order.types.ts` - Re-export new types

**Create:**
- `apps/web/src/mocks/data/order-timeline-events.data.ts` - Order-specific timeline

## Implementation Steps

### 1. Update MockOrder Type

```typescript
// In mocks/types.ts - Add to MockOrder interface

// Payment (readonly from external)
billingAddress?: MockAddress;
paymentMethod?: 'COD' | 'BANK_TRANSFER' | 'CARD' | 'PAYPAL';
paymentStatus?: 'UNPAID' | 'PAID' | 'PARTIALLY_PAID' | 'REFUNDED';

// Extra fees
tip?: MockMoney;
donation?: MockMoney;
```

### 2. Add PaymentMethod & PaymentStatus Enums

```typescript
// In mocks/types.ts

export const PaymentMethod = {
  COD: 'COD',
  BANK_TRANSFER: 'BANK_TRANSFER',
  CARD: 'CARD',
  PAYPAL: 'PAYPAL',
} as const;
export type PaymentMethodType = (typeof PaymentMethod)[keyof typeof PaymentMethod];

export const PaymentStatus = {
  UNPAID: 'UNPAID',
  PAID: 'PAID',
  PARTIALLY_PAID: 'PARTIALLY_PAID',
  REFUNDED: 'REFUNDED',
} as const;
export type PaymentStatusType = (typeof PaymentStatus)[keyof typeof PaymentStatus];
```

### 3. Update MockOrderItem Type

```typescript
// In mocks/types.ts - Update MockOrderItem interface

export interface MockOrderItem {
  // ... existing fields

  // Make designStatus required (was optional)
  designStatus: ItemDesignStatusType;

  // New fields
  attributes?: Array<{ key: string; value: string }>;
  personalization?: Array<{ key: string; value: string }>;
  designFileInfo?: {
    url: string;
    fileName: string;
    fileSize: number; // bytes
    uploadedAt: string;
  };
}
```

### 4. Add OrderTimelineEventType

```typescript
// In mocks/types.ts

export const OrderTimelineEventType = {
  ORDER_CREATED: 'ORDER_CREATED',
  ORDER_CONFIRMED: 'ORDER_CONFIRMED',
  DESIGN_STARTED: 'DESIGN_STARTED',
  DESIGN_COMPLETED: 'DESIGN_COMPLETED',
  READY_TO_FULFILL: 'READY_TO_FULFILL',
  SHIPPED: 'SHIPPED',
  COMPLETED: 'COMPLETED',
  CANCELLED: 'CANCELLED',
  NOTE_UPDATED: 'NOTE_UPDATED',
  PAYMENT_RECEIVED: 'PAYMENT_RECEIVED',
} as const;
export type OrderTimelineEventTypeType =
  (typeof OrderTimelineEventType)[keyof typeof OrderTimelineEventType];

export interface MockOrderTimelineEvent {
  id: string;
  orderId: string;
  type: OrderTimelineEventTypeType;
  userId: string;
  userName?: string;
  description: string;
  metadata?: Record<string, unknown>;
  createdAt: string;
}
```

### 5. Update orders.data.ts

```typescript
// Generate billing address, payment info for each order
return {
  // ... existing fields

  billingAddress: {
    fullName: customer.name,
    phone: customer.phone,
    line1: `${100 + n} Billing Street`,
    city: 'San Francisco',
    state: 'CA',
    postalCode: '94102',
    country: 'US',
  },
  paymentMethod: paymentMethods[i % 4], // COD, BANK_TRANSFER, CARD, PAYPAL
  paymentStatus: paymentStatuses[i % 4], // Based on order status
  tip: i % 5 === 0 ? { amount: 5000, currency: 'USD' } : undefined,
  donation: i % 7 === 0 ? { amount: 2000, currency: 'USD' } : undefined,
};
```

### 6. Update order-items.data.ts

```typescript
// Add designStatus, attributes, personalization

const designStatuses: ItemDesignStatusType[] = [
  'NOT_REQUIRED', 'PENDING', 'IN_PROGRESS', 'APPROVED'
];

const attributeTemplates = {
  T_SHIRT: [
    { key: 'Color', value: 'Black' },
    { key: 'Size', value: 'L' },
    { key: 'Gender', value: 'Unisex' },
  ],
  // ... other product types
};

mockOrderItems.push({
  // ... existing fields
  designStatus: designStatuses[(itemIndex - 1) % 4],
  attributes: attributeTemplates[type],
  personalization: j === 0 ? [
    { key: 'Name', value: 'John Doe' },
    { key: 'Message', value: 'Happy Birthday!' },
  ] : undefined,
  designFileInfo: designStatuses[(itemIndex - 1) % 4] !== 'NOT_REQUIRED' ? {
    url: `https://example.com/designs/design-${itemIndex}.png`,
    fileName: `design-${itemIndex}.png`,
    fileSize: 1200000 + (itemIndex * 100000), // 1.2MB - 2.5MB
    uploadedAt: daysAgo(5 - (itemIndex % 5)),
  } : undefined,
});
```

### 7. Create order-timeline-events.data.ts

```typescript
// New file: mocks/data/order-timeline-events.data.ts

import { mockOrders } from './orders.data';
import { USER_IDS, daysAgo } from '../factories/mock-factory';

export interface MockOrderTimelineEvent {
  id: string;
  orderId: string;
  type: string;
  userId: string;
  userName: string;
  description: string;
  metadata?: Record<string, unknown>;
  createdAt: string;
}

const events: MockOrderTimelineEvent[] = [];
let eventIdx = 1;

mockOrders.forEach((order, idx) => {
  const baseDate = new Date(order.createdAt);

  // ORDER_CREATED
  events.push({
    id: `oevt-${String(eventIdx++).padStart(5, '0')}`,
    orderId: order.id,
    type: 'ORDER_CREATED',
    userId: 'system',
    userName: 'System',
    description: 'Order created from WooCommerce',
    createdAt: order.createdAt,
  });

  // Add more events based on order status...
});

export const mockOrderTimelineEvents = events;
export const getOrderTimeline = (orderId: string) =>
  events.filter(e => e.orderId === orderId)
    .sort((a, b) => new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime());
```

### 8. Update order.types.ts (feature types)

```typescript
// Re-export new types
export type {
  PaymentMethodType,
  PaymentStatusType,
  MockOrderTimelineEvent,
} from '@/mocks/types';

export { PaymentMethod, PaymentStatus, OrderTimelineEventType } from '@/mocks/types';

// Enhanced TimelineEvent for UI
export interface OrderTimelineEvent {
  id: string;
  timestamp: string;
  description: string;
  type: 'info' | 'success' | 'warning' | 'error';
  userName?: string;
  eventType?: string; // ORDER_CREATED, etc.
}
```

## Todo

- [x] Add PaymentMethod, PaymentStatus enums to types.ts
- [x] Add billingAddress, paymentMethod, paymentStatus, tip, donation to MockOrder
- [x] Update MockOrderItem with required designStatus, attributes, personalization, designFileInfo
- [x] Add OrderTimelineEventType and MockOrderTimelineEvent
- [x] Update orders.data.ts to generate new fields
- [x] Update order-items.data.ts to generate new fields
- [x] Create order-timeline-events.data.ts
- [x] Update data/index.ts exports
- [x] Update feature types re-exports
- [x] Run TypeScript build to verify no errors

**All tasks completed successfully. Score: 8.5/10**

## Success Criteria

- All new types compile without errors
- Mock data generates valid values for all new fields
- `designStatus` is present on every order item (no undefined)
- Order timeline has at least 3 events per order

## Next Phase

→ [Phase 02: API Endpoints](phase-02-api-endpoints.md)
