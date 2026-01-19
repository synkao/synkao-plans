# Phase 03: Orders Page Simplify

## Context Links
- Parent: [plan.md](plan.md)
- Depends on: [phase-02](phase-02-order-list-table-lazy-fetch.md)
- File: `apps/web/src/app/(main)/orders/page.tsx`

## Overview
- **Priority:** P1
- **Status:** Completed
- **Description:** Simplify page to use useOrders instead of useOrdersWithItems

## Key Insights
- Current page uses useOrdersWithItems which batch fetches all items
- With lazy loading, only need useOrders for order list
- selectedItemCount calculation needs adjustment (can't count without expand)

## Requirements

### Functional
- Replace useOrdersWithItems with useOrders
- Pass orders (without items) to OrderListTable
- Handle selectedItemCount (remove or show "—")

### Non-Functional
- Faster initial page load (no batch item fetch)
- Cleaner code

## Related Code Files

**Modify:**
- `apps/web/src/app/(main)/orders/page.tsx`

## Implementation Steps

1. Change import and hook usage:
   ```tsx
   // Before
   import { useOrdersWithItems, ... } from '@/features/orders';
   const { data: orders, pagination, isLoading, error } = useOrdersWithItems({...});

   // After
   import { useOrders, ... } from '@/features/orders';
   const { data, pagination, isLoading, error } = useOrders({...});
   const orders = data?.data ?? [];
   ```

2. Update OrderListTable props:
   ```tsx
   // Now passes MockOrder[] instead of OrderWithItems[]
   <OrderListTable
     orders={orders}
     onOrderClick={handleOrderClick}
     onViewItemTask={handleViewItemTask}
     onSelectionChange={handleSelectionChange}
   />
   ```

3. Handle selectedItemCount:
   ```tsx
   // Option A: Remove from BulkActionBar (can't know without expand)
   // Option B: Show "—" or hide itemCount prop

   // Current:
   const selectedItemCount = orders
     .filter((o) => selectedOrderIds.has(o.id))
     .reduce((sum, o) => sum + o.items.length, 0);

   // After: Remove or set to undefined
   // BulkActionBar can show just order count
   ```

4. Update BulkActionBar usage:
   ```tsx
   <BulkActionBar
     selectedCount={selectedOrderIds.size}
     // itemCount removed or set to undefined
     onExport={handleExport}
     onCreateFulfillment={handleCreateFulfillment}
     onAssignDesigner={handleAssignDesigner}
     onClearSelection={handleClearSelection}
   />
   ```

## Todo List
- [x] Change useOrdersWithItems to useOrders
- [x] Update orders variable extraction
- [x] Remove or adjust selectedItemCount calculation
- [x] Update BulkActionBar props (remove itemCount or make optional)
- [x] Verify types compile correctly

## Success Criteria
- Page loads faster (no batch item fetch)
- Order list displays correctly
- Selection works
- Pagination works
- No TypeScript errors

## Risk Assessment
- **Low risk:** Simplification, mostly removing code
- BulkActionBar may need minor adjustment for optional itemCount

## Next Steps
- Test full flow: load → select → expand → view items
- Verify React Query caching works as expected
