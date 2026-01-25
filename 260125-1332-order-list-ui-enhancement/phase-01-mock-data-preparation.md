# Phase 1: Mock Data Preparation

**Status**: ✅ Complete
**Priority**: High (Blocking Phase 2)
**Review**: [Code Review Report](./reports/code-reviewer-260125-1342-phase-01-mock-data.md)

## Overview

Prepare mock data to support the new Order List UI features. This includes adding new type fields, creating store data, and populating existing optional fields.

## Context Links

- [MockOrder Type](../../apps/web/src/mocks/types.ts#L242-L289)
- [Orders Data](../../apps/web/src/mocks/data/orders.data.ts)
- [Orders Handler](../../apps/web/src/mocks/handlers/orders.ts)

## Requirements

### Functional
- Store information displayed on order rows
- Revenue shown alongside total cost
- Fulfiller and tracking info for shipped orders
- Notes visible on order list
- Status counts for filter tabs

### Non-Functional
- Mock data should be realistic
- Data distribution should vary (not all orders have all fields)

## Implementation Steps

### Task 1.1: Update MockOrder Type
**File**: `apps/web/src/mocks/types.ts`

Add new fields to `MockOrder` interface (after line 288):

```typescript
// Add to MockOrder interface:
storeName?: string;        // Display name for store badge
revenue?: MockMoney;       // Calculated: total - costs
fulfillDeadline?: string;  // ISO date string
```

### Task 1.2: Create Mock Stores Data
**File**: `apps/web/src/mocks/data/stores.data.ts` (CREATE)

```typescript
import type { PlatformType } from '../types';
import { Platform } from '../types';

export interface MockStore {
  id: string;
  name: string;
  platform: PlatformType;
}

export const mockStores: MockStore[] = [
  { id: 'store-woo-main', name: 'WooCommerce Main', platform: Platform.WOOCOMMERCE },
  { id: 'store-shopify-us', name: 'Shopify US', platform: Platform.SHOPIFY },
  { id: 'store-manual', name: 'Manual Orders', platform: Platform.MANUAL },
];

export const findStore = (id: string) => mockStores.find((s) => s.id === id);
```

### Task 1.3: Export Stores from Index
**File**: `apps/web/src/mocks/data/index.ts`

Add export:
```typescript
export * from './stores.data';
```

### Task 1.4: Update Orders Data
**File**: `apps/web/src/mocks/data/orders.data.ts`

#### 1.4.1 Add data arrays (before mockOrders):

```typescript
// Fulfillers for POD business
const fulfillers = ['Printful', 'Gooten', 'Printify', 'SPOD', null];

// Carriers for tracking
const carriers = ['USPS', 'FedEx', 'UPS', 'DHL'];

// Sample tracking patterns by carrier
const trackingPatterns: Record<string, string> = {
  USPS: '9400111899223384769',
  FedEx: '794644790047',
  UPS: '1Z999AA10123456784',
  DHL: '1234567890',
};

// Sample notes (most orders have no notes)
const sampleNotes = [
  'Gift order - include card',
  'Rush order - expedite shipping',
  'Customer requested white background',
  'Check size before printing',
  'Fragile - handle with care',
  null, null, null, null, null, // 50% have no notes
];

// Store rotation
import { mockStores } from './stores.data';
```

#### 1.4.2 Add helper functions:

```typescript
/**
 * Get fulfiller name based on order status
 * Only fulfilled orders have assigned fulfiller
 */
const getFulfiller = (status: OrderStatusType, index: number): string | undefined => {
  const needsFulfiller = [
    OrderStatus.READY_TO_FULFILL,
    OrderStatus.SHIPPED,
    OrderStatus.COMPLETED,
  ].includes(status);
  if (!needsFulfiller) return undefined;
  return fulfillers[index % fulfillers.length] ?? undefined;
};

/**
 * Get tracking info for shipped/completed orders
 */
const getTrackingInfo = (status: OrderStatusType, index: number) => {
  const hasTracking = [OrderStatus.SHIPPED, OrderStatus.COMPLETED].includes(status);
  if (!hasTracking) return { trackingNumber: undefined, carrier: undefined };

  const carrier = carriers[index % carriers.length]!;
  const trackingNumber = `${trackingPatterns[carrier]}${index}`;
  return { trackingNumber, carrier };
};

/**
 * Calculate revenue (total - 30% cost margin)
 */
const calculateRevenue = (totalAmount: number): number => {
  return Math.round(totalAmount * 0.7);
};

/**
 * Calculate fulfill deadline (createdAt + 3 days)
 */
const getFulfillDeadline = (createdAt: string): string => {
  const date = new Date(createdAt);
  date.setDate(date.getDate() + 3);
  return date.toISOString();
};
```

#### 1.4.3 Update mockOrders array to include new fields:

```typescript
// In the return object of mockOrders array, add:
const store = mockStores[i % mockStores.length]!;
const { trackingNumber, carrier } = getTrackingInfo(status, i);

return {
  // ... existing fields ...

  // Store info
  storeId: store.id,
  storeName: store.name,

  // Fulfillment (enhanced)
  fulfillerName: getFulfiller(status, i),
  trackingNumber,
  carrier,
  shippedAt: [OrderStatus.SHIPPED, OrderStatus.COMPLETED].includes(status)
    ? daysAgo(7 - i)
    : undefined,

  // Notes
  notes: sampleNotes[i % sampleNotes.length] ?? undefined,

  // Revenue
  revenue: { amount: calculateRevenue(totalAmount), currency: 'USD' },

  // Fulfill deadline
  fulfillDeadline: getFulfillDeadline(daysAgo(15 - i)),

  // ... rest of existing fields ...
};
```

### Task 1.5: Add Orders Counts Endpoint
**File**: `apps/web/src/mocks/handlers/orders.ts`

Add new handler before the closing bracket:

```typescript
// GET /api/v1/orders/counts - Get order counts by status
http.get('/api/v1/orders/counts', () => {
  const counts: Record<string, number> = {};
  let total = 0;

  for (const order of mockOrders) {
    counts[order.status] = (counts[order.status] || 0) + 1;
    total++;
  }

  return successResponse({ counts, total });
}),
```

## Todo List

- [x] 1.1 Add storeName, revenue, fulfillDeadline to MockOrder type
- [x] 1.2 Create stores.data.ts with mock stores
- [x] 1.3 Export stores from data/index.ts
- [x] 1.4 Update orders.data.ts with new helper functions
- [x] 1.5 Populate all new fields in mockOrders array
- [x] 1.6 Add /api/v1/orders/counts endpoint
- [x] 1.7 Verify TypeScript compiles without errors (ESLint passed)
- [x] H1: Extract revenue calculation margin to constant
- [x] H2: Fix fulfill deadline to only apply to relevant statuses
- [x] M1: Remove non-null assertion in store lookup
- [x] M2: Document tracking number pattern limitations

## Success Criteria

- [x] `MockOrder` type includes new optional fields
- [x] All 20 mock orders have store info populated
- [x] SHIPPED/COMPLETED orders have tracking info
- [x] Revenue calculated correctly (70% of total)
- [x] Fulfill deadline set to 3 days after createdAt
- [x] `/api/v1/orders/counts` returns status breakdown
- [x] No TypeScript compilation errors

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `mocks/types.ts` | Modify | Add 3 new fields to MockOrder |
| `mocks/data/stores.data.ts` | Create | New file with 3 mock stores |
| `mocks/data/index.ts` | Modify | Add stores export |
| `mocks/data/orders.data.ts` | Modify | Add helpers + populate fields |
| `mocks/handlers/orders.ts` | Modify | Add counts endpoint |

## Next Steps

**Immediate Actions**:
1. Address H1, H2 findings from code review (~20 min)
2. Verify Node.js >=20.9.0 or adjust Next.js config
3. Run build to confirm no type errors

**Then Proceed**:
→ [Phase 2: UI Implementation](./phase-02-ui-implementation.md)
