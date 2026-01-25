# Phase 2: UI Implementation

**Status**: Complete
**Priority**: High
**Dependencies**: Phase 1 (Mock Data)
**Code Review**: [Phase 2 Review Report](./reports/code-reviewer-260125-1402-phase02-ui-implementation.md)

## Overview

Implement the new Order List UI with restructured columns, inline status edit, status tabs, and enhanced item rows.

## Context Links

- [Order List Table](../../apps/web/src/features/orders/components/order-list/order-list-table.tsx)
- [Order Row](../../apps/web/src/features/orders/components/order-list/order-row.tsx)
- [Order Item Row](../../apps/web/src/features/orders/components/order-list/order-item-row.tsx)
- [Orders Page Header](../../apps/web/src/app/(main)/orders/orders-page-header.tsx)

## Requirements

### Functional
- Display BUYER, COST, FF, NOTE columns
- Store badge under order number
- Inline status edit via popover
- Status tabs with counts
- 56x56 thumbnails for items
- Design tags for items needing attention

### Non-Functional
- Responsive design (handle narrow screens)
- Accessible (keyboard navigation, ARIA)
- Performance (no unnecessary re-renders)

## Column Structure (New vs Old)

| Old Columns | New Columns |
|-------------|-------------|
| ☐ Checkbox | ☐ Checkbox |
| ORDER | ORDER + Store Badge |
| CUSTOMER | BUYER (Name + Phone + Address) |
| PHONE | → merged into BUYER |
| ADDRESS | → merged into BUYER |
| TIME | STATUS + Deadline |
| - | COST (Total + Revenue) |
| - | FF (Fulfiller + Tracking) |
| - | NOTE |
| Actions | Actions (Edit + View only) |

## Implementation Steps

### Task 2.1: Create Status Edit Popover Component
**File**: `apps/web/src/features/orders/components/order-list/status-edit-popover.tsx` (CREATE)

```typescript
'use client';

import { useState } from 'react';
import { Popover, PopoverContent, PopoverTrigger } from '@/components/ui/popover';
import { Button } from '@/components/ui/button';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { ChevronDown } from 'lucide-react';
import { OrderStatus, type OrderStatusType } from '@/mocks/types';
import { OrderStatusBadge } from '../shared/order-status-badge';

interface StatusEditPopoverProps {
  currentStatus: OrderStatusType;
  onStatusChange: (newStatus: OrderStatusType) => void;
  disabled?: boolean;
}

const statusOptions: { value: OrderStatusType; label: string }[] = [
  { value: OrderStatus.PENDING, label: 'Pending' },
  { value: OrderStatus.CONFIRMED, label: 'Confirmed' },
  { value: OrderStatus.IN_DESIGN, label: 'In Design' },
  { value: OrderStatus.READY_TO_FULFILL, label: 'Ready to Fulfill' },
  { value: OrderStatus.SHIPPED, label: 'Shipped' },
  { value: OrderStatus.COMPLETED, label: 'Completed' },
  { value: OrderStatus.CANCELLED, label: 'Cancelled' },
];

export function StatusEditPopover({ currentStatus, onStatusChange, disabled }: StatusEditPopoverProps) {
  const [open, setOpen] = useState(false);
  const [selectedStatus, setSelectedStatus] = useState(currentStatus);

  const handleUpdate = () => {
    if (selectedStatus !== currentStatus) {
      onStatusChange(selectedStatus);
    }
    setOpen(false);
  };

  const handleCancel = () => {
    setSelectedStatus(currentStatus);
    setOpen(false);
  };

  return (
    <Popover open={open} onOpenChange={setOpen}>
      <PopoverTrigger asChild disabled={disabled}>
        <button className="flex items-center gap-1 hover:opacity-80 transition-opacity">
          <OrderStatusBadge status={currentStatus} />
          <ChevronDown className="h-3 w-3 text-gray-400" />
        </button>
      </PopoverTrigger>
      <PopoverContent className="w-64 p-3" align="start">
        <div className="space-y-3">
          <Select value={selectedStatus} onValueChange={(v) => setSelectedStatus(v as OrderStatusType)}>
            <SelectTrigger>
              <SelectValue />
            </SelectTrigger>
            <SelectContent>
              {statusOptions.map((opt) => (
                <SelectItem key={opt.value} value={opt.value}>
                  {opt.label}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
          <div className="flex justify-end gap-2">
            <Button variant="outline" size="sm" onClick={handleCancel}>
              Cancel
            </Button>
            <Button size="sm" onClick={handleUpdate}>
              Update
            </Button>
          </div>
        </div>
      </PopoverContent>
    </Popover>
  );
}
```

### Task 2.2: Create Order Status Tabs Component
**File**: `apps/web/src/features/orders/components/order-list/order-status-tabs.tsx` (CREATE)

```typescript
'use client';

import { cn } from '@/lib/utils';
import { OrderStatus, type OrderStatusType } from '@/mocks/types';

interface OrderStatusTabsProps {
  counts: Record<string, number>;
  total: number;
  activeStatus: OrderStatusType | 'ALL';
  onStatusChange: (status: OrderStatusType | 'ALL') => void;
}

const tabs: { value: OrderStatusType | 'ALL'; label: string }[] = [
  { value: 'ALL', label: 'All' },
  { value: OrderStatus.PENDING, label: 'Pending' },
  { value: OrderStatus.CONFIRMED, label: 'Confirmed' },
  { value: OrderStatus.IN_DESIGN, label: 'In Design' },
  { value: OrderStatus.READY_TO_FULFILL, label: 'Ready' },
  { value: OrderStatus.SHIPPED, label: 'Shipped' },
  { value: OrderStatus.COMPLETED, label: 'Completed' },
  { value: OrderStatus.CANCELLED, label: 'Cancelled' },
];

export function OrderStatusTabs({ counts, total, activeStatus, onStatusChange }: OrderStatusTabsProps) {
  return (
    <div className="flex gap-1 overflow-x-auto pb-1">
      {tabs.map((tab) => {
        const count = tab.value === 'ALL' ? total : (counts[tab.value] || 0);
        const isActive = activeStatus === tab.value;

        return (
          <button
            key={tab.value}
            onClick={() => onStatusChange(tab.value)}
            className={cn(
              'px-3 py-1.5 text-sm rounded-md whitespace-nowrap transition-colors',
              isActive
                ? 'bg-blue-100 text-blue-700 font-medium'
                : 'text-gray-600 hover:bg-gray-100'
            )}
          >
            {tab.label}
            <span className={cn('ml-1.5', isActive ? 'text-blue-600' : 'text-gray-400')}>
              {count}
            </span>
          </button>
        );
      })}
    </div>
  );
}
```

### Task 2.3: Create useOrderCounts Hook
**File**: `apps/web/src/features/orders/hooks/use-order-counts.ts` (CREATE)

```typescript
import { useQuery } from '@tanstack/react-query';
import { ordersClient } from '../lib/orders-client';

interface OrderCountsResponse {
  counts: Record<string, number>;
  total: number;
}

export function useOrderCounts() {
  return useQuery({
    queryKey: ['orders', 'counts'],
    queryFn: async (): Promise<OrderCountsResponse> => {
      // Add this method to orders-client.ts
      const response = await fetch('/api/v1/orders/counts');
      if (!response.ok) throw new Error('Failed to fetch order counts');
      const data = await response.json();
      return data.data;
    },
    staleTime: 30 * 1000, // 30 seconds
  });
}
```

### Task 2.4: Update Order List Table Headers
**File**: `apps/web/src/features/orders/components/order-list/order-list-table.tsx`

Update table headers (replace lines ~69-97):

```tsx
<thead className="border-b border-gray-200 bg-gray-50/80">
  <tr>
    <th className="w-8" /> {/* Expand toggle */}
    <th className="w-10 px-2 py-3 text-left">
      <input type="checkbox" ... />
    </th>
    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider min-w-[140px]">
      Order
    </th>
    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider min-w-[180px]">
      Buyer
    </th>
    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider min-w-[100px]">
      Status
    </th>
    <th className="px-4 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider min-w-[100px]">
      Cost
    </th>
    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider min-w-[140px]">
      FF
    </th>
    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider min-w-[120px]">
      Note
    </th>
    <th className="w-20" /> {/* Actions */}
  </tr>
</thead>
```

### Task 2.5: Update Order Row Component (Major)
**File**: `apps/web/src/features/orders/components/order-list/order-row.tsx`

This is the most significant change. Update the entire row to show new columns:

**Key changes:**
- Add store badge under order number
- Merge customer info into BUYER column
- Add STATUS column with popover + deadline
- Add COST column with total/revenue
- Add FF column with fulfiller + tracking (copy icon)
- Add NOTE column
- Update ACTION column (remove mail)

See code snippet structure:
```tsx
{/* Order + Store Badge */}
<td className="px-4 py-3">
  <div className="flex flex-col">
    <span className="font-medium text-gray-900">{order.orderNumber}</span>
    {order.storeName && (
      <span className="text-xs text-gray-400 bg-gray-100 px-1.5 py-0.5 rounded w-fit mt-0.5">
        {order.storeName}
      </span>
    )}
  </div>
</td>

{/* Buyer (merged) */}
<td className="px-4 py-3">
  <div className="flex flex-col text-sm">
    <span className="text-gray-700 font-medium">{order.customerName}</span>
    <span className="text-gray-500 text-xs">{order.customerPhone || '—'}</span>
    <span className="text-gray-400 text-xs truncate max-w-[160px]" title={addressText}>
      {addressText}
    </span>
  </div>
</td>

{/* Status + Deadline */}
<td className="px-4 py-3">
  <div className="flex flex-col gap-1">
    <StatusEditPopover
      currentStatus={order.status}
      onStatusChange={(newStatus) => handleStatusChange(order.id, newStatus)}
    />
    {order.fulfillDeadline && (
      <span className="text-xs text-gray-400">
        Due: {formatDate(order.fulfillDeadline)}
      </span>
    )}
  </div>
</td>

{/* Cost (Total + Revenue) */}
<td className="px-4 py-3 text-right">
  <div className="flex flex-col text-sm">
    <span className="text-gray-900 font-medium">
      {formatMoney(order.total)}
    </span>
    {order.revenue && (
      <span className="text-xs text-green-600">
        +{formatMoney(order.revenue)}
      </span>
    )}
  </div>
</td>

{/* FF (Fulfiller + Tracking) */}
<td className="px-4 py-3">
  <div className="flex flex-col text-sm gap-0.5">
    {order.fulfillerName ? (
      <span className="text-gray-700">{order.fulfillerName}</span>
    ) : (
      <span className="text-gray-400 italic">Not assigned</span>
    )}
    {order.trackingNumber && (
      <div className="flex items-center gap-1">
        <span className="text-xs text-blue-600 font-mono truncate max-w-[100px]">
          {order.trackingNumber}
        </span>
        <CopyButton text={order.trackingNumber} />
      </div>
    )}
  </div>
</td>

{/* Note */}
<td className="px-4 py-3">
  <span className="text-sm text-gray-600 truncate max-w-[100px] block" title={order.notes}>
    {order.notes || '—'}
  </span>
</td>
```

### Task 2.6: Update Order Item Row
**File**: `apps/web/src/features/orders/components/order-list/order-item-row.tsx`

**Changes:**
1. Increase thumbnail size: 40x40 → 56x56
2. Update design tags logic: show only "Missing Design" or "Unmapped"

```tsx
{/* Thumbnail - increase to 56x56 */}
<div className="h-14 w-14 flex-shrink-0 overflow-hidden rounded border border-gray-200 bg-gray-100 relative">
  {item.productImageUrl ? (
    <Image
      src={item.productImageUrl}
      alt={item.productName}
      fill
      className="object-cover"
      sizes="56px"
      unoptimized
    />
  ) : (
    <div className="flex h-full w-full items-center justify-center text-xs text-gray-400">
      IMG
    </div>
  )}
</div>

{/* Design tags - only show when needs attention */}
{item.designStatus === ItemDesignStatus.PENDING && (
  <span className="text-xs bg-yellow-100 text-yellow-700 px-1.5 py-0.5 rounded">
    Missing Design
  </span>
)}
{/* Add Unmapped tag logic if applicable */}
```

### Task 2.7: Update Orders Page Header
**File**: `apps/web/src/app/(main)/orders/orders-page-header.tsx`

Add status tabs below the header:

```tsx
import { OrderStatusTabs } from '@/features/orders/components/order-list/order-status-tabs';
import { useOrderCounts } from '@/features/orders/hooks/use-order-counts';

// In component:
const { data: countsData } = useOrderCounts();

// Add tabs below header actions:
<OrderStatusTabs
  counts={countsData?.counts || {}}
  total={countsData?.total || 0}
  activeStatus={statusFilter}
  onStatusChange={handleStatusFilterChange}
/>
```

### Task 2.8: Update Exports
**File**: `apps/web/src/features/orders/components/order-list/index.ts`

Add new component exports:
```typescript
export * from './status-edit-popover';
export * from './order-status-tabs';
```

**File**: `apps/web/src/features/orders/hooks/index.ts`

Add hook export:
```typescript
export * from './use-order-counts';
```

## Todo List

- [x] 2.1 Create StatusEditPopover component
- [x] 2.2 Create OrderStatusTabs component
- [x] 2.3 Create useOrderCounts hook
- [x] 2.4 Update table headers in order-list-table.tsx
- [x] 2.5 Refactor order-row.tsx with new columns
- [x] 2.6 Update order-item-row.tsx (thumbnail + tags)
- [x] 2.7 Add status tabs to orders-page-header.tsx
- [x] 2.8 Update exports (index.ts files)
- [ ] 2.9 Test all interactions work correctly (BLOCKED: needs status change handler)

## Success Criteria

- [ ] All columns display correctly with data
- [ ] Store badge shows under order number
- [ ] Buyer column shows merged info (name, phone, address)
- [ ] Status popover opens and allows status change
- [ ] Fulfill deadline shows below status
- [ ] Cost shows total and revenue
- [ ] FF shows fulfiller name and tracking with copy
- [ ] Note column shows truncated note text
- [ ] Status tabs filter orders correctly
- [ ] Tabs show correct counts
- [ ] Item thumbnails are 56x56
- [ ] Design tags show only when needed
- [ ] No TypeScript errors
- [ ] No console errors

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `order-list/status-edit-popover.tsx` | Create | Inline status edit |
| `order-list/order-status-tabs.tsx` | Create | Filter tabs component |
| `hooks/use-order-counts.ts` | Create | Fetch status counts |
| `order-list/order-list-table.tsx` | Modify | Update headers |
| `order-list/order-row.tsx` | Modify | Major refactor |
| `order-list/order-item-row.tsx` | Modify | Thumbnail + tags |
| `orders-page-header.tsx` | Modify | Add tabs |
| `order-list/index.ts` | Modify | Add exports |
| `hooks/index.ts` | Modify | Add exports |

## Security Considerations

- Status change should validate allowed transitions
- Sanitize notes display (XSS prevention)
- Tracking numbers should not be guessable

## Code Review Results (2026-01-25)

**Overall Score**: 8.5/10

### Critical Issues (Must Fix Before Merge)
1. **C1**: Missing status change handler implementation - UI functional but changes not persisted
2. **C2**: XSS vulnerability in notes display - needs sanitization

### High Priority Issues
1. **H1**: No input validation on status transitions - users can skip workflow steps
2. **H2**: Missing error boundary for table component
3. **H3**: useOrderCounts has no error handling - shows 0 counts on API failure
4. **H4**: Potential memory leak in CopyButton setTimeout
5. **H5**: Missing loading state for status change operations

### Implementation Status
- ✅ All UI components implemented correctly
- ✅ TypeScript compilation passes with no errors
- ✅ Accessibility features comprehensive (ARIA labels, keyboard nav)
- ✅ Performance optimizations in place (lazy loading, debounce, caching)
- ⚠️ Status change handler missing - critical functional gap
- ⚠️ Security vulnerability in notes field - needs sanitization
- ❌ No unit tests found

### Recommended Actions

**Before Merge**:
1. Implement status change handler in page.tsx
2. Sanitize notes display to prevent XSS
3. Add status transition validation
4. Add error handling to useOrderCounts hook
5. Fix CopyButton cleanup issue

**Next Sprint**:
6. Add error boundary for table
7. Extract formatMoney to shared utility
8. Implement optimistic updates
9. Add unit tests for components
10. Add loading skeletons

## Next Steps

After fixing critical issues:
1. Implement status change handler (C1)
2. Sanitize notes field (C2)
3. Add status transition validation (H1)
4. Browser testing of all interactions
5. Close Issue #74
