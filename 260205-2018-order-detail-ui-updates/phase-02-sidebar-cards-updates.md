# Phase 02: Sidebar Cards Updates

## Context
- [Codebase Analysis](./reports/codebase-analysis.md)
- [Phase 01](./phase-01-header-and-status-dropdown.md) must be completed first
- [Code Review Report](./reports/code-reviewer-260205-2249-phase-02-review.md)

## Overview
- **Priority:** P1
- **Status:** completed
- **Effort:** 2h
- **Completed:** 2026-02-05 22:49 ICT
- **Code Review Score:** 8/10

## Key Insights
- OrderMetadataCard needs Store and External ID rows
- CustomerInfoCard currently shows External ID and Source (to be removed)
- OrderSummaryCard payment row has badge + Mark Paid link (simplify)
- OrderStatusStepper component no longer needed in page

## Requirements

### Functional
1. **Order Info Card (Sidebar)**
   - Add Store row: platform icon + domain
   - Add External ID row: mono font display
   - Remove Update Status button

2. **Customer Information Card**
   - Remove External ID row
   - Remove Source row

3. **Order Summary Card**
   - Show payment method name only
   - Remove Paid/Unpaid badge
   - Remove Mark Paid link

4. **Page Layout**
   - Remove OrderStatusStepper from sidebar

### Non-Functional
- Platform icons use existing PlatformIcon component pattern
- External ID uses monospace font for readability

## Architecture

### Updated Order Info Card Structure
```
OrderMetadataCard (renamed conceptually to "Order Info")
├── CardHeader: "Order Info"
├── CardContent
│   ├── Store row: [PlatformIcon] domain
│   ├── External ID row: [Tag icon] mono text
│   ├── Created row: [Clock icon] datetime
│   └── Updated row: [RefreshCw icon] datetime
```

### Platform Icon Config (extend existing)
```typescript
// Add Etsy to existing PLATFORM_CONFIG
ETSY: { letter: 'E', bgClass: 'bg-[#F56400]' },
```

## Related Code Files

### Modify
| File | Changes |
|------|---------|
| `order-metadata-card.tsx` | Add Store, External ID rows; remove button |
| `customer-info-card.tsx` | Remove External ID, Source sections |
| `order-summary-card.tsx` | Simplify payment row |
| `page.tsx` | Remove OrderStatusStepper usage |
| `constants.ts` | Add ETSY to PLATFORM_CONFIG |

### No New Files Required

## Implementation Steps

### Step 1: Update Platform Config (constants.ts)
```typescript
// Modify: features/orders/lib/constants.ts

export const PLATFORM_CONFIG: Record<PlatformType | 'ETSY', PlatformConfig> = {
  WOOCOMMERCE: { letter: 'W', bgClass: 'bg-[#7B3DB0]' }, // Update color per spec
  SHOPIFY: { letter: 'S', bgClass: 'bg-[#96BF48]' },
  ETSY: { letter: 'E', bgClass: 'bg-[#F56400]' },        // NEW
  MANUAL: { letter: 'M', bgClass: 'bg-[#6B7280]' },      // Update letter
};
```

### Step 2: Update OrderMetadataCard
```typescript
// Modify: features/orders/components/order-detail/order-metadata-card.tsx

'use client';

import type { MockOrder, PlatformType } from '../../types';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { Clock, RefreshCw, Tag, Store } from 'lucide-react';
import { PlatformIcon } from '../order-list/platform-icon';
import { mockStores, findStore } from '@/mocks/data/stores.data';

export interface OrderMetadataCardProps {
  order: MockOrder;
  // Remove onUpdateStatus prop
}

export function OrderMetadataCard({ order }: OrderMetadataCardProps) {
  const formatDateTime = (dateStr: string) => {
    return new Date(dateStr).toLocaleDateString('en-US', {
      month: 'short',
      day: 'numeric',
      year: 'numeric',
      hour: '2-digit',
      minute: '2-digit',
    });
  };

  // Get store info
  const store = order.storeId ? findStore(order.storeId) : null;

  return (
    <Card className="border border-gray-200/50 bg-white/60 backdrop-blur-md">
      <CardHeader className="pb-3">
        <CardTitle className="text-sm font-medium text-gray-500 uppercase tracking-wider">
          Order Info
        </CardTitle>
      </CardHeader>
      <CardContent className="space-y-3">
        {/* Store row */}
        {store && (
          <div className="flex items-center justify-between text-sm">
            <span className="text-gray-500 flex items-center gap-2">
              <Store className="h-4 w-4" />
              Store
            </span>
            <span className="text-gray-700 flex items-center gap-2">
              <PlatformIcon platform={store.platform} />
              {store.domain}
            </span>
          </div>
        )}

        {/* External ID row */}
        {order.externalId && (
          <div className="flex items-center justify-between text-sm">
            <span className="text-gray-500 flex items-center gap-2">
              <Tag className="h-4 w-4" />
              External ID
            </span>
            <span className="text-gray-700 font-mono text-xs">
              {order.externalId}
            </span>
          </div>
        )}

        {/* Created row */}
        <div className="flex items-center justify-between text-sm">
          <span className="text-gray-500 flex items-center gap-2">
            <Clock className="h-4 w-4" />
            Created
          </span>
          <span className="text-gray-700">{formatDateTime(order.createdAt)}</span>
        </div>

        {/* Updated row */}
        {order.updatedAt && (
          <div className="flex items-center justify-between text-sm">
            <span className="text-gray-500 flex items-center gap-2">
              <RefreshCw className="h-4 w-4" />
              Updated
            </span>
            <span className="text-gray-700">{formatDateTime(order.updatedAt)}</span>
          </div>
        )}

        {/* REMOVED: Update Status button */}
      </CardContent>
    </Card>
  );
}
```

### Step 3: Update CustomerInfoCard
```typescript
// Modify: features/orders/components/order-detail/customer-info-card.tsx

// Remove these imports:
// import { Tag, Globe } from 'lucide-react';

// Remove SOURCE_LABELS constant

// In CustomerInfoCard component, remove this entire section:
/*
{(order.externalId || order.source) && (
  <>
    <Separator />
    <div className="space-y-2">
      {order.externalId && (
        <InfoRow icon={Tag} label="External ID" value={order.externalId} />
      )}
      {order.source && (
        <InfoRow
          icon={Globe}
          label="Source"
          value={SOURCE_LABELS[order.source] ?? order.source}
        />
      )}
    </div>
  </>
)}
*/
```

### Step 4: Update OrderSummaryCard
```typescript
// Modify: features/orders/components/order-detail/order-summary-card.tsx

// Remove Badge import and PAYMENT_STATUS_CONFIG usage

// Replace payment section (lines ~66-78) with:
{/* Payment info - simplified */}
<div className="flex items-center justify-between">
  <span className="text-sm text-gray-500">Payment</span>
  <span className="text-sm text-gray-700">{paymentMethodLabel}</span>
</div>
```

### Step 5: Update Page Layout (page.tsx)
```typescript
// Modify: apps/web/src/app/(main)/orders/[id]/page.tsx

// Remove OrderStatusStepper from imports:
// import { OrderStatusStepper } from '@/features/orders';

// In the sidebar section, remove:
// <OrderStatusStepper status={order.status} />

// Update OrderMetadataCard usage (remove onUpdateStatus prop):
<OrderMetadataCard order={order} />
```

## Todo List
- [ ] Add ETSY to PLATFORM_CONFIG in constants.ts
- [ ] Update WooCommerce/Manual colors in PLATFORM_CONFIG
- [ ] Modify order-metadata-card.tsx (add Store/External ID, remove button)
- [ ] Modify customer-info-card.tsx (remove External ID/Source)
- [ ] Modify order-summary-card.tsx (simplify payment row)
- [ ] Remove OrderStatusStepper from page.tsx
- [ ] Remove onUpdateStatus prop from OrderMetadataCard usage
- [ ] Run TypeScript compiler to verify

## Success Criteria
- [ ] Order Info card shows Store and External ID rows
- [ ] Order Info card has no Update Status button
- [ ] Customer Info card has no External ID or Source rows
- [ ] Order Summary shows payment method only (no badge)
- [ ] Sidebar no longer shows stepper component
- [ ] No TypeScript errors

## Risk Assessment
| Risk | Impact | Mitigation |
|------|--------|------------|
| Store data missing | Low | Show row only when store exists |
| Platform not in config | Low | PlatformIcon has fallback |

## Security Considerations
- External ID displayed as-is (no user input)
- No API calls in this phase

## Next Steps
After completion, proceed to Phase 03: Design Areas Components
