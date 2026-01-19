# Phase 3: Order List Components

## Context Links

- Spec: `docs/frontend/synkao-frontend-spec.md` section 3.1
- Pattern: `apps/web/src/features/design/components/backlog/backlog-table.tsx`
- Pattern: `apps/web/src/features/design/components/backlog/backlog-filter-bar.tsx`
- Pattern: `apps/web/src/features/design/components/backlog/backlog-action-bar.tsx`
- Shared: Phase 2 components (badges, fulfiller cell)

## Overview

| Field | Value |
|-------|-------|
| Priority | P1 |
| Status | pending |
| Effort | 2h |
| Date | 2026-01-19 |

Create the main Order List table with grouped rows (Approach B), filter bar, and bulk action bar.

## Key Insights

From spec wireframe:
- Order header row: checkbox, Order#, Customer, Phone, Address, Time
- Item sub-rows: Thumbnail, Product, Qty, DesignStatus, Fulfiller, Actions (Eye icon)
- **NOT expandable** - items always visible inline
- Checkbox at order level selects all items
- Grouped visual with indentation for items

## Requirements

### Functional
- `OrderRow`: Header row with order info + checkbox
- `OrderItemRow`: Item row with thumbnail, product, qty, status, fulfiller
- `OrderListTable`: Combined table rendering grouped rows
- `OrderListFilters`: Search, Status, Design, Fulfiller, Time range filters
- `BulkActionBar`: Export, Create Fulfillment, Assign Designer buttons

### Non-functional
- Responsive table (horizontal scroll on mobile)
- Keyboard accessible checkboxes
- Visual grouping via indentation and borders

## Architecture

```
components/order-list/
├── order-list-table.tsx    # Main table container
├── order-row.tsx           # Order header row
├── order-item-row.tsx      # Item sub-row
├── order-list-filters.tsx  # Filter bar
├── bulk-action-bar.tsx     # Floating action bar
└── index.ts
```

## Related Code Files

**To Create:**
- `features/orders/components/order-list/order-row.tsx`
- `features/orders/components/order-list/order-item-row.tsx`
- `features/orders/components/order-list/order-list-table.tsx`
- `features/orders/components/order-list/order-list-filters.tsx`
- `features/orders/components/order-list/bulk-action-bar.tsx`
- `features/orders/components/order-list/index.ts`

**Reference:**
- `features/design/components/backlog/backlog-table.tsx`
- `features/design/components/backlog/backlog-action-bar.tsx`

## Implementation Steps

### 1. Create OrderRow component

**File:** `features/orders/components/order-list/order-row.tsx`

```typescript
'use client';

import type { MockOrder } from '../../types';
import { cn } from '@/lib/utils';
import { formatRelativeTime } from '@/features/design/lib/utils';

export interface OrderRowProps {
  order: MockOrder;
  isSelected: boolean;
  onSelect: (orderId: string, selected: boolean) => void;
  onClick?: (order: MockOrder) => void;
}

/**
 * Header row for an order in the grouped list
 * Contains: checkbox, order#, customer, phone, address, time
 */
export function OrderRow({ order, isSelected, onSelect, onClick }: OrderRowProps) {
  const address = order.shippingAddress;
  const addressText = address
    ? `${address.line1}, ${address.city}`
    : '—';

  return (
    <tr
      className={cn(
        'border-b border-gray-200 bg-gray-50/50 transition-colors',
        isSelected && 'bg-blue-50/50',
        onClick && 'cursor-pointer hover:bg-gray-100'
      )}
      onClick={() => onClick?.(order)}
    >
      {/* Checkbox */}
      <td className="w-10 px-4 py-3">
        <input
          type="checkbox"
          checked={isSelected}
          onChange={(e) => {
            e.stopPropagation();
            onSelect(order.id, e.target.checked);
          }}
          onClick={(e) => e.stopPropagation()}
          className="h-4 w-4 rounded border-gray-300 text-blue-600 focus:ring-blue-500"
          aria-label={`Select order ${order.orderNumber}`}
        />
      </td>

      {/* Order Number */}
      <td className="px-4 py-3">
        <span className="font-medium text-gray-900">{order.orderNumber}</span>
      </td>

      {/* Customer */}
      <td className="px-4 py-3">
        <span className="text-gray-700">{order.customerName}</span>
      </td>

      {/* Phone */}
      <td className="px-4 py-3">
        <span className="text-gray-600">{order.customerPhone || '—'}</span>
      </td>

      {/* Address */}
      <td className="px-4 py-3">
        <span className="text-gray-600 truncate max-w-[200px] inline-block" title={addressText}>
          {addressText}
        </span>
      </td>

      {/* Time */}
      <td className="px-4 py-3 text-right">
        <span className="text-sm text-gray-500">
          {formatRelativeTime(order.createdAt)}
        </span>
      </td>

      {/* Empty cell for item actions column alignment */}
      <td className="w-10"></td>
    </tr>
  );
}
```

### 2. Create OrderItemRow component

**File:** `features/orders/components/order-list/order-item-row.tsx`

```typescript
'use client';

import type { MockOrderItem } from '../../types';
import { Eye } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { ItemDesignStatusBadge, FulfillerCell } from '../shared';
import { cn } from '@/lib/utils';

export interface OrderItemRowProps {
  item: MockOrderItem;
  fulfillerName?: string;
  isShipped?: boolean;
  isDelivered?: boolean;
  onViewTask?: (item: MockOrderItem) => void;
  isLastItem?: boolean;
}

/**
 * Sub-row for an order item in the grouped list
 * Contains: thumbnail, product name, qty, design status, fulfiller, action
 */
export function OrderItemRow({
  item,
  fulfillerName,
  isShipped,
  isDelivered,
  onViewTask,
  isLastItem,
}: OrderItemRowProps) {
  return (
    <tr
      className={cn(
        'border-b border-gray-100 bg-white transition-colors hover:bg-gray-50/50',
        isLastItem && 'border-b-2 border-gray-200'
      )}
    >
      {/* Empty checkbox cell - items don't have individual checkboxes */}
      <td className="w-10 px-4 py-2">
        <span className="ml-4 text-gray-300">├─</span>
      </td>

      {/* Thumbnail + Product Name (spans 2 columns) */}
      <td className="px-4 py-2" colSpan={2}>
        <div className="flex items-center gap-3">
          {/* Thumbnail */}
          <div className="h-10 w-10 flex-shrink-0 overflow-hidden rounded border border-gray-200 bg-gray-100">
            {item.productImageUrl ? (
              <img
                src={item.productImageUrl}
                alt={item.productName}
                className="h-full w-full object-cover"
              />
            ) : (
              <div className="flex h-full w-full items-center justify-center text-xs text-gray-400">
                IMG
              </div>
            )}
          </div>
          {/* Product Name */}
          <span className="text-sm text-gray-700">{item.productName}</span>
        </div>
      </td>

      {/* Quantity */}
      <td className="px-4 py-2">
        <span className="text-sm text-gray-600">x{item.quantity}</span>
      </td>

      {/* Design Status */}
      <td className="px-4 py-2">
        <ItemDesignStatusBadge status={item.designStatus} />
      </td>

      {/* Fulfiller */}
      <td className="px-4 py-2">
        <FulfillerCell
          fulfillerName={fulfillerName}
          isShipped={isShipped}
          isDelivered={isDelivered}
        />
      </td>

      {/* Action */}
      <td className="w-10 px-4 py-2">
        {onViewTask && (
          <Button
            variant="ghost"
            size="icon"
            className="h-8 w-8"
            onClick={() => onViewTask(item)}
            aria-label={`View task for ${item.productName}`}
          >
            <Eye className="h-4 w-4 text-gray-500" />
          </Button>
        )}
      </td>
    </tr>
  );
}
```

### 3. Create OrderListTable component

**File:** `features/orders/components/order-list/order-list-table.tsx`

```typescript
'use client';

import { useState, useCallback } from 'react';
import type { OrderWithItems, MockOrder, MockOrderItem } from '../../types';
import { Card } from '@/components/ui/card';
import { OrderRow } from './order-row';
import { OrderItemRow } from './order-item-row';

export interface OrderListTableProps {
  orders: OrderWithItems[];
  onOrderClick?: (order: MockOrder) => void;
  onViewItemTask?: (item: MockOrderItem) => void;
  onSelectionChange?: (selectedOrderIds: Set<string>) => void;
}

/**
 * Main table component displaying orders with grouped item rows
 * Approach B: Custom grouped rows (NOT expandable)
 */
export function OrderListTable({
  orders,
  onOrderClick,
  onViewItemTask,
  onSelectionChange,
}: OrderListTableProps) {
  const [selectedIds, setSelectedIds] = useState<Set<string>>(new Set());

  const handleSelectOrder = useCallback((orderId: string, selected: boolean) => {
    setSelectedIds((prev) => {
      const newSet = new Set(prev);
      if (selected) {
        newSet.add(orderId);
      } else {
        newSet.delete(orderId);
      }
      onSelectionChange?.(newSet);
      return newSet;
    });
  }, [onSelectionChange]);

  const handleSelectAll = useCallback(() => {
    const allSelected = selectedIds.size === orders.length;
    const newSet = allSelected
      ? new Set<string>()
      : new Set(orders.map((o) => o.id));
    setSelectedIds(newSet);
    onSelectionChange?.(newSet);
  }, [orders, selectedIds.size, onSelectionChange]);

  return (
    <Card className="border border-gray-200/50 bg-white/60 backdrop-blur-md overflow-hidden">
      <div className="overflow-x-auto">
        <table className="w-full">
          <thead className="border-b border-gray-200 bg-gray-50/80">
            <tr>
              <th className="w-10 px-4 py-3 text-left">
                <input
                  type="checkbox"
                  checked={selectedIds.size === orders.length && orders.length > 0}
                  onChange={handleSelectAll}
                  className="h-4 w-4 rounded border-gray-300"
                  aria-label="Select all orders"
                />
              </th>
              <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Order
              </th>
              <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Customer
              </th>
              <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Phone
              </th>
              <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Address
              </th>
              <th className="px-4 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">
                Time
              </th>
              <th className="w-10"></th>
            </tr>
          </thead>
          <tbody>
            {orders.map((order) => (
              <OrderGroup
                key={order.id}
                order={order}
                isSelected={selectedIds.has(order.id)}
                onSelect={handleSelectOrder}
                onOrderClick={onOrderClick}
                onViewItemTask={onViewItemTask}
              />
            ))}
          </tbody>
        </table>
      </div>

      {orders.length === 0 && (
        <div className="flex items-center justify-center py-12">
          <p className="text-sm text-gray-500">No orders found</p>
        </div>
      )}
    </Card>
  );
}

interface OrderGroupProps {
  order: OrderWithItems;
  isSelected: boolean;
  onSelect: (orderId: string, selected: boolean) => void;
  onOrderClick?: (order: MockOrder) => void;
  onViewItemTask?: (item: MockOrderItem) => void;
}

function OrderGroup({
  order,
  isSelected,
  onSelect,
  onOrderClick,
  onViewItemTask,
}: OrderGroupProps) {
  return (
    <>
      <OrderRow
        order={order}
        isSelected={isSelected}
        onSelect={onSelect}
        onClick={onOrderClick}
      />
      {order.items.map((item, idx) => (
        <OrderItemRow
          key={item.id}
          item={item}
          fulfillerName={order.fulfillerName}
          isShipped={order.status === 'SHIPPED'}
          isDelivered={order.status === 'COMPLETED'}
          onViewTask={onViewItemTask}
          isLastItem={idx === order.items.length - 1}
        />
      ))}
    </>
  );
}
```

### 4. Create OrderListFilters component

**File:** `features/orders/components/order-list/order-list-filters.tsx`

```typescript
'use client';

import type { OrderStatusType, DesignPhaseStatusType } from '../../types';
import { OrderStatus, DesignPhaseStatus } from '../../types';
import { Input } from '@/components/ui/input';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Search, Calendar } from 'lucide-react';
import { ORDER_STATUS_CONFIG, DESIGN_PHASE_CONFIG } from '../../lib/constants';

export interface OrderListFiltersProps {
  search: string;
  onSearchChange: (search: string) => void;
  status: OrderStatusType | 'ALL';
  onStatusChange: (status: OrderStatusType | 'ALL') => void;
  designStatus: DesignPhaseStatusType | 'ALL';
  onDesignStatusChange: (status: DesignPhaseStatusType | 'ALL') => void;
  // fulfillerFilter and timeRange can be added later
}

/**
 * Filter bar for order list
 * Contains: search, status dropdown, design status dropdown
 */
export function OrderListFilters({
  search,
  onSearchChange,
  status,
  onStatusChange,
  designStatus,
  onDesignStatusChange,
}: OrderListFiltersProps) {
  return (
    <div className="flex flex-wrap gap-3 items-center">
      {/* Search */}
      <div className="relative flex-1 min-w-[200px]">
        <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-gray-400" />
        <Input
          placeholder="Search orders..."
          value={search}
          onChange={(e) => onSearchChange(e.target.value)}
          className="pl-9"
        />
      </div>

      {/* Status Filter */}
      <Select value={status} onValueChange={onStatusChange}>
        <SelectTrigger className="w-40">
          <SelectValue placeholder="Status" />
        </SelectTrigger>
        <SelectContent>
          <SelectItem value="ALL">All Status</SelectItem>
          {Object.entries(ORDER_STATUS_CONFIG).map(([key, config]) => (
            <SelectItem key={key} value={key}>
              {config.label}
            </SelectItem>
          ))}
        </SelectContent>
      </Select>

      {/* Design Status Filter */}
      <Select value={designStatus} onValueChange={onDesignStatusChange}>
        <SelectTrigger className="w-40">
          <SelectValue placeholder="Design" />
        </SelectTrigger>
        <SelectContent>
          <SelectItem value="ALL">All Design</SelectItem>
          {Object.entries(DESIGN_PHASE_CONFIG)
            .filter(([key]) => key !== 'NONE')
            .map(([key, config]) => (
              <SelectItem key={key} value={key}>
                {config.label}
              </SelectItem>
            ))}
        </SelectContent>
      </Select>

      {/* Time Range placeholder - can add DateRangePicker later */}
      <div className="flex items-center gap-1 text-sm text-gray-500">
        <Calendar className="h-4 w-4" />
        <span>All time</span>
      </div>
    </div>
  );
}
```

### 5. Create BulkActionBar component

**File:** `features/orders/components/order-list/bulk-action-bar.tsx`

```typescript
'use client';

import { Button } from '@/components/ui/button';
import { Download, Truck, UserPlus, X, CheckCircle } from 'lucide-react';
import { cn } from '@/lib/utils';

export interface BulkActionBarProps {
  selectedCount: number;
  itemCount?: number;  // Total items across selected orders
  onExport: () => void;
  onCreateFulfillment: () => void;
  onAssignDesigner: () => void;
  onClearSelection: () => void;
}

/**
 * Floating action bar when orders are selected
 * Actions: Export, Create Fulfillment, Assign Designer
 */
export function BulkActionBar({
  selectedCount,
  itemCount,
  onExport,
  onCreateFulfillment,
  onAssignDesigner,
  onClearSelection,
}: BulkActionBarProps) {
  if (selectedCount === 0) return null;

  const itemsText = itemCount ? ` (${itemCount} items)` : '';

  return (
    <div
      className={cn(
        'fixed bottom-6 left-1/2 z-50 -translate-x-1/2',
        'rounded-lg border border-gray-200/50 bg-white/90 px-6 py-3 shadow-lg backdrop-blur-md',
        'flex items-center gap-4'
      )}
    >
      {/* Selection count */}
      <div className="flex items-center gap-2">
        <CheckCircle className="h-5 w-5 text-blue-600" />
        <span className="text-sm font-medium text-gray-700">
          {selectedCount} orders selected{itemsText}
        </span>
      </div>

      <div className="h-6 w-px bg-gray-200" />

      {/* Actions */}
      <div className="flex items-center gap-2">
        <Button variant="outline" size="sm" onClick={onExport}>
          <Download className="h-4 w-4 mr-1" />
          Export
        </Button>

        <Button variant="outline" size="sm" onClick={onCreateFulfillment}>
          <Truck className="h-4 w-4 mr-1" />
          Create Fulfillment
        </Button>

        <Button variant="outline" size="sm" onClick={onAssignDesigner}>
          <UserPlus className="h-4 w-4 mr-1" />
          Assign Designer
        </Button>

        <Button
          variant="ghost"
          size="sm"
          onClick={onClearSelection}
          className="text-gray-500"
        >
          <X className="h-4 w-4" />
        </Button>
      </div>
    </div>
  );
}
```

### 6. Create index file

**File:** `features/orders/components/order-list/index.ts`

```typescript
export { OrderListTable } from './order-list-table';
export type { OrderListTableProps } from './order-list-table';

export { OrderRow } from './order-row';
export type { OrderRowProps } from './order-row';

export { OrderItemRow } from './order-item-row';
export type { OrderItemRowProps } from './order-item-row';

export { OrderListFilters } from './order-list-filters';
export type { OrderListFiltersProps } from './order-list-filters';

export { BulkActionBar } from './bulk-action-bar';
export type { BulkActionBarProps } from './bulk-action-bar';
```

## Todo List

- [ ] Create `features/orders/components/order-list/order-row.tsx`
- [ ] Create `features/orders/components/order-list/order-item-row.tsx`
- [ ] Create `features/orders/components/order-list/order-list-table.tsx`
- [ ] Create `features/orders/components/order-list/order-list-filters.tsx`
- [ ] Create `features/orders/components/order-list/bulk-action-bar.tsx`
- [ ] Create `features/orders/components/order-list/index.ts`
- [ ] Visual test grouped row rendering
- [ ] Test checkbox selection behavior
- [ ] Verify filter components render

## Success Criteria

- Orders display with header rows + item sub-rows
- Visual grouping clear (indentation, borders)
- Checkbox at order level works
- Select all checkbox works
- Filters render with dropdowns
- Bulk action bar appears when selection > 0

## Risk Assessment

| Risk | Mitigation |
|------|------------|
| Complex table rendering performance | Keep component composition simple, avoid re-renders |
| Responsive layout breaks | Use overflow-x-auto, test on mobile |
| Selection state sync | Use controlled state in parent |

## Security Considerations

- No user input beyond search text (sanitized by React)
- Image URLs from mock data only

## Next Steps

Proceed to Phase 4: Order List Page (integration with hooks, state management)
