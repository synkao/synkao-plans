# Phase 4: Order List Page

## Context Links

- Spec: `docs/frontend/synkao-frontend-spec.md` section 3.1
- Pattern: `apps/web/src/app/(main)/design/backlog/page.tsx`
- Existing page: `apps/web/src/app/(main)/orders/page.tsx` (placeholder)
- Components: Phase 3 order-list components
- Hooks: Phase 1 `useOrdersWithItems()`

## Overview

| Field | Value |
|-------|-------|
| Priority | P1 |
| Status | pending |
| Effort | 1h |
| Date | 2026-01-19 |

Integrate order list components with React Query hooks, manage page state, add pagination.

## Key Insights

- Follow backlog page pattern for state management
- Use `useOrdersWithItems()` to fetch orders with nested items
- Debounce search input
- Reset page/selection on filter change
- Navigate to detail page on order click

## Requirements

### Functional
- Fetch orders with pagination
- Filter by search, status, design status
- Display grouped table
- Show bulk action bar on selection
- Navigate to `/orders/:id` on order click
- Loading and error states

### Non-functional
- Server-side pagination
- Debounced search (300ms)
- URL state sync (optional, can add later)

## Architecture

```
app/(main)/orders/
├── page.tsx             # Update existing placeholder
└── [id]/
    └── page.tsx         # Created in Phase 6
```

## Related Code Files

**To Update:**
- `apps/web/src/app/(main)/orders/page.tsx`

**To Use:**
- Phase 1 hooks: `useOrdersWithItems()`
- Phase 3 components: `OrderListTable`, `OrderListFilters`, `BulkActionBar`
- `features/orders/index.ts` (main export)

## Implementation Steps

### 1. Create features/orders/index.ts

**File:** `features/orders/index.ts`

```typescript
// Types
export * from './types';

// Lib
export * from './lib';

// Hooks
export * from './hooks';

// Components
export * from './components/shared';
export * from './components/order-list';
```

### 2. Update Orders Page

**File:** `apps/web/src/app/(main)/orders/page.tsx`

```typescript
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { toast } from 'sonner';
import { PageHeader } from '@/components/layout';
import { Button } from '@/components/ui/button';
import { Plus, Upload } from 'lucide-react';
import Link from 'next/link';
import {
  useOrdersWithItems,
  OrderListTable,
  OrderListFilters,
  BulkActionBar,
  type MockOrder,
  type MockOrderItem,
  type OrderStatusType,
  type DesignPhaseStatusType,
} from '@/features/orders';
import { useDebouncedValue } from '@/hooks/use-debounced-value';
import { DataTablePagination } from '@/components/data-table';

export default function OrdersPage() {
  const router = useRouter();

  // Pagination state
  const [page, setPage] = useState(1);
  const [pageSize, setPageSize] = useState(10);

  // Filter state
  const [search, setSearch] = useState('');
  const [status, setStatus] = useState<OrderStatusType | 'ALL'>('ALL');
  const [designStatus, setDesignStatus] = useState<DesignPhaseStatusType | 'ALL'>('ALL');

  // Selection state
  const [selectedOrderIds, setSelectedOrderIds] = useState<Set<string>>(new Set());

  // Debounced search
  const debouncedSearch = useDebouncedValue(search, 300);

  // Fetch orders with items
  const { data: orders, pagination, isLoading, error } = useOrdersWithItems({
    page,
    limit: pageSize,
    search: debouncedSearch || undefined,
    status: status !== 'ALL' ? status : undefined,
    design_status: designStatus !== 'ALL' ? designStatus : undefined,
  });

  // Handle filter changes - reset page and selection
  const handleSearchChange = (value: string) => {
    setSearch(value);
    setPage(1);
    setSelectedOrderIds(new Set());
  };

  const handleStatusChange = (value: OrderStatusType | 'ALL') => {
    setStatus(value);
    setPage(1);
    setSelectedOrderIds(new Set());
  };

  const handleDesignStatusChange = (value: DesignPhaseStatusType | 'ALL') => {
    setDesignStatus(value);
    setPage(1);
    setSelectedOrderIds(new Set());
  };

  // Handle pagination
  const handlePageChange = (newPage: number) => {
    setPage(newPage);
    setSelectedOrderIds(new Set());
  };

  const handlePageSizeChange = (size: number) => {
    setPageSize(size);
    setPage(1);
    setSelectedOrderIds(new Set());
  };

  // Handle order click - navigate to detail
  const handleOrderClick = (order: MockOrder) => {
    router.push(`/orders/${order.id}`);
  };

  // Handle view item task - open task drawer (placeholder)
  const handleViewItemTask = (item: MockOrderItem) => {
    toast.info(`View task for ${item.productName} (coming soon)`);
  };

  // Handle selection
  const handleSelectionChange = (ids: Set<string>) => {
    setSelectedOrderIds(ids);
  };

  // Bulk actions
  const handleExport = () => {
    toast.info(`Export ${selectedOrderIds.size} orders (coming soon)`);
  };

  const handleCreateFulfillment = () => {
    toast.info(`Create fulfillment for ${selectedOrderIds.size} orders (coming soon)`);
  };

  const handleAssignDesigner = () => {
    toast.info(`Assign designer to ${selectedOrderIds.size} orders (coming soon)`);
  };

  const handleClearSelection = () => {
    setSelectedOrderIds(new Set());
  };

  // Calculate item count for selected orders
  const selectedItemCount = orders
    .filter((o) => selectedOrderIds.has(o.id))
    .reduce((sum, o) => sum + o.items.length, 0);

  // Loading state
  if (isLoading) {
    return (
      <>
        <OrdersPageHeader />
        <div className="flex items-center justify-center py-12">
          <p className="text-muted-foreground">Loading orders...</p>
        </div>
      </>
    );
  }

  // Error state
  if (error) {
    return (
      <>
        <OrdersPageHeader />
        <div className="flex items-center justify-center py-12">
          <p className="text-destructive">Failed to load orders: {error.message}</p>
        </div>
      </>
    );
  }

  return (
    <>
      <OrdersPageHeader />

      <div className="space-y-4">
        {/* Filters */}
        <OrderListFilters
          search={search}
          onSearchChange={handleSearchChange}
          status={status}
          onStatusChange={handleStatusChange}
          designStatus={designStatus}
          onDesignStatusChange={handleDesignStatusChange}
        />

        {/* Order Table */}
        <OrderListTable
          orders={orders}
          onOrderClick={handleOrderClick}
          onViewItemTask={handleViewItemTask}
          onSelectionChange={handleSelectionChange}
        />

        {/* Pagination */}
        {pagination && (
          <OrdersPagination
            page={page}
            pageSize={pageSize}
            total={pagination.total}
            totalPages={pagination.totalPages}
            onPageChange={handlePageChange}
            onPageSizeChange={handlePageSizeChange}
          />
        )}
      </div>

      {/* Bulk Action Bar */}
      <BulkActionBar
        selectedCount={selectedOrderIds.size}
        itemCount={selectedItemCount}
        onExport={handleExport}
        onCreateFulfillment={handleCreateFulfillment}
        onAssignDesigner={handleAssignDesigner}
        onClearSelection={handleClearSelection}
      />
    </>
  );
}

// Page header component
function OrdersPageHeader() {
  return (
    <PageHeader
      title="Orders"
      description="Manage your print-on-demand orders"
      breadcrumbs={[
        { label: 'Dashboard', href: '/' },
        { label: 'Orders' },
      ]}
      actions={
        <div className="flex gap-2">
          <Button variant="outline" asChild>
            <Link href="/orders/import">
              <Upload className="mr-2 h-4 w-4" />
              Import
            </Link>
          </Button>
          <Button>
            <Plus className="mr-2 h-4 w-4" />
            New Order
          </Button>
        </div>
      }
    />
  );
}

// Simple pagination component
interface OrdersPaginationProps {
  page: number;
  pageSize: number;
  total: number;
  totalPages: number;
  onPageChange: (page: number) => void;
  onPageSizeChange: (size: number) => void;
}

function OrdersPagination({
  page,
  pageSize,
  total,
  totalPages,
  onPageChange,
  onPageSizeChange,
}: OrdersPaginationProps) {
  return (
    <div className="flex items-center justify-between px-2">
      <p className="text-sm text-muted-foreground">
        Showing {(page - 1) * pageSize + 1} to {Math.min(page * pageSize, total)} of {total} orders
      </p>
      <div className="flex items-center gap-2">
        <Button
          variant="outline"
          size="sm"
          onClick={() => onPageChange(page - 1)}
          disabled={page <= 1}
        >
          Previous
        </Button>
        <span className="text-sm">
          Page {page} of {totalPages}
        </span>
        <Button
          variant="outline"
          size="sm"
          onClick={() => onPageChange(page + 1)}
          disabled={page >= totalPages}
        >
          Next
        </Button>
      </div>
    </div>
  );
}
```

## Todo List

- [ ] Create `features/orders/index.ts` main export file
- [ ] Update `apps/web/src/app/(main)/orders/page.tsx`
- [ ] Test loading state
- [ ] Test error state
- [ ] Test pagination
- [ ] Test filter changes reset pagination
- [ ] Test selection and bulk action bar
- [ ] Test navigation to order detail

## Success Criteria

- Page loads orders from MSW
- Filters work and reset pagination
- Grouped rows display correctly
- Selection shows bulk action bar
- Click order navigates to detail
- Loading/error states display

## Risk Assessment

| Risk | Mitigation |
|------|------------|
| Multiple queries for items cause waterfall | `useOrdersWithItems` handles parallel queries |
| Debounce not working | Reuse existing `useDebouncedValue` hook |
| Router navigation issues | Use `useRouter` from next/navigation |

## Security Considerations

- No sensitive data handling
- Navigation uses Next.js router (safe)

## Next Steps

Proceed to Phase 5: Order Detail Components (CustomerInfoCard, OrderStatusStepper, etc.)
