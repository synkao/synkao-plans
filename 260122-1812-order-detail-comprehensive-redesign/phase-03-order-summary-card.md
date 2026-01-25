# Phase 03: UI - Order Summary Card

**Priority:** P0 - Critical
**Status:** ✅ Complete
**Depends on:** Phase 01
**Completed:** 2026-01-22 19:14
**Review:** reports/code-reviewer-260122-phase-03-order-summary-card.md

## Overview

New component displaying complete financial breakdown: subtotal, shipping, discount, tip, donation, total, payment method & status.

## Related Files

**Create:**
- `apps/web/src/features/orders/components/order-detail/order-summary-card.tsx`

**Modify:**
- `apps/web/src/features/orders/components/order-detail/index.ts` - Export new component
- `apps/web/src/app/(main)/orders/[id]/page.tsx` - Add OrderSummaryCard

## Design

```
┌─────────────────────────────────────┐
│ ORDER SUMMARY                       │
├─────────────────────────────────────┤
│ Subtotal              $380.00       │
│ Shipping               $30.00       │
│ Discount                   —        │
│ Tip                     $5.00       │
│ Donation                $2.00       │
├─────────────────────────────────────┤
│ TOTAL                 $417.00       │
├─────────────────────────────────────┤
│ Payment    COD    ⬜ Unpaid         │
└─────────────────────────────────────┘
```

## Implementation

### Component Structure

```tsx
// order-summary-card.tsx

'use client';

import type { MockOrder } from '../../types';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { Separator } from '@/components/ui/separator';
import { cn } from '@/lib/utils';

export interface OrderSummaryCardProps {
  order: MockOrder;
}

// Payment status colors
const PAYMENT_STATUS_CONFIG = {
  UNPAID: { label: 'Unpaid', bgClass: 'bg-gray-100', textClass: 'text-gray-600' },
  PAID: { label: 'Paid', bgClass: 'bg-emerald-100', textClass: 'text-emerald-700' },
  PARTIALLY_PAID: { label: 'Partial', bgClass: 'bg-amber-100', textClass: 'text-amber-700' },
  REFUNDED: { label: 'Refunded', bgClass: 'bg-red-100', textClass: 'text-red-700' },
};

// Payment method labels
const PAYMENT_METHOD_LABELS = {
  COD: 'Cash on Delivery',
  BANK_TRANSFER: 'Bank Transfer',
  CARD: 'Credit Card',
  PAYPAL: 'PayPal',
};

export function OrderSummaryCard({ order }: OrderSummaryCardProps) {
  const formatMoney = (money?: { amount: number; currency: string }) => {
    if (!money) return '—';
    return new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: money.currency,
    }).format(money.amount / 100); // Assuming cents
  };

  const paymentConfig = PAYMENT_STATUS_CONFIG[order.paymentStatus ?? 'UNPAID'];
  const paymentMethodLabel = PAYMENT_METHOD_LABELS[order.paymentMethod ?? 'COD'];

  return (
    <Card className="border border-gray-200/50 bg-white/60 backdrop-blur-md">
      <CardHeader className="pb-3">
        <CardTitle className="text-sm font-medium text-gray-500 uppercase tracking-wider">
          Order Summary
        </CardTitle>
      </CardHeader>
      <CardContent className="space-y-3">
        {/* Line items */}
        <div className="space-y-2 text-sm">
          <SummaryRow label="Subtotal" value={formatMoney(order.subtotal)} />
          <SummaryRow label="Shipping" value={formatMoney(order.shippingFee)} />
          <SummaryRow label="Discount" value={formatMoney(order.discount)} />
          {order.tip && <SummaryRow label="Tip" value={formatMoney(order.tip)} />}
          {order.donation && <SummaryRow label="Donation" value={formatMoney(order.donation)} />}
        </div>

        <Separator />

        {/* Total */}
        <div className="flex justify-between items-center">
          <span className="font-semibold text-gray-900">TOTAL</span>
          <span className="font-bold text-lg text-gray-900">
            {formatMoney(order.total)}
          </span>
        </div>

        <Separator />

        {/* Payment info */}
        <div className="flex items-center justify-between">
          <span className="text-sm text-gray-500">Payment</span>
          <div className="flex items-center gap-2">
            <span className="text-sm text-gray-700">{paymentMethodLabel}</span>
            <Badge
              variant="secondary"
              className={cn(paymentConfig.bgClass, paymentConfig.textClass)}
            >
              {paymentConfig.label}
            </Badge>
          </div>
        </div>
      </CardContent>
    </Card>
  );
}

function SummaryRow({ label, value }: { label: string; value: string }) {
  return (
    <div className="flex justify-between">
      <span className="text-gray-500">{label}</span>
      <span className="text-gray-700">{value}</span>
    </div>
  );
}
```

### Update index.ts

```typescript
// order-detail/index.ts
export { OrderSummaryCard } from './order-summary-card';
```

### Update page.tsx

```tsx
// In /orders/[id]/page.tsx

import { OrderSummaryCard } from '@/features/orders';

// In Right Column section, after OrderStatusStepper:
<OrderSummaryCard order={order} />
```

## Constants

Add to `lib/constants.ts` if needed:

```typescript
export const PAYMENT_STATUS_CONFIG: Record<PaymentStatusType, StatusConfig> = {
  UNPAID: { label: 'Unpaid', color: 'gray', bgClass: 'bg-gray-100', textClass: 'text-gray-600' },
  PAID: { label: 'Paid', color: 'emerald', bgClass: 'bg-emerald-100', textClass: 'text-emerald-700' },
  PARTIALLY_PAID: { label: 'Partial', color: 'amber', bgClass: 'bg-amber-100', textClass: 'text-amber-700' },
  REFUNDED: { label: 'Refunded', color: 'red', bgClass: 'bg-red-100', textClass: 'text-red-700' },
};
```

## Todo

- [x] Create order-summary-card.tsx component
- [x] Add formatMoney utility (inline in component)
- [x] Add PAYMENT_STATUS_CONFIG constants (inline)
- [x] Export from order-detail/index.ts
- [x] Import and use in page.tsx
- [x] Test with different payment statuses
- [x] Verify mobile responsive

## Success Criteria

- Displays all financial fields (subtotal, shipping, discount, tip, donation, total)
- Shows payment method with readable label
- Shows payment status with color-coded badge
- Handles missing optional fields gracefully (shows "—")
- Responsive on mobile

## Next Phase

→ [Phase 04: Product Cards Enhancement](phase-04-product-cards-enhancement.md)
