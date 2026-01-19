# Phase 02: OrderListTable Expandable State & Lazy Fetch

## Context Links
- Parent: [plan.md](plan.md)
- Depends on: [phase-01](phase-01-order-row-expand-toggle.md)
- File: `apps/web/src/features/orders/components/order-list/order-list-table.tsx`

## Overview
- **Priority:** P1
- **Status:** Completed
- **Description:** Add expand state management and lazy item fetching

## Key Insights
- Use Set<string> for tracking multiple expanded orders
- useOrderItems hook exists, use `enabled: expanded` for lazy fetch
- Need loading and empty states for items

## Requirements

### Functional
- Track expanded order IDs in Set
- Fetch items only when order is expanded
- Show loading spinner while fetching
- Show "No items" for empty orders
- Items hidden when collapsed

### Non-Functional
- React Query caches items (instant on second expand)
- Smooth expand/collapse transition

## Architecture

```tsx
OrderListTable
├── expandedIds: Set<string>
└── OrderGroup
    ├── order: MockOrder (not OrderWithItems)
    ├── expanded: boolean
    ├── onToggleExpand()
    └── Conditional render:
        └── {expanded && <ItemsSection />}
            ├── useOrderItems(orderId, { enabled: expanded })
            ├── isLoading → <LoadingItemsRow />
            ├── items.length === 0 → <EmptyItemsRow />
            └── items.map() → <OrderItemRow />
```

## Related Code Files

**Modify:**
- `apps/web/src/features/orders/components/order-list/order-list-table.tsx`

**Reference:**
- `apps/web/src/features/orders/hooks/use-orders.ts` (useOrderItems hook)

## Implementation Steps

1. Change props type from `OrderWithItems[]` to `MockOrder[]`:
   ```tsx
   interface OrderListTableProps {
     orders: MockOrder[];  // Changed from OrderWithItems[]
     // ... rest same
   }
   ```

2. Add expandedIds state:
   ```tsx
   const [expandedIds, setExpandedIds] = useState<Set<string>>(new Set());

   const toggleExpand = useCallback((orderId: string) => {
     setExpandedIds(prev => {
       const next = new Set(prev);
       if (next.has(orderId)) next.delete(orderId);
       else next.add(orderId);
       return next;
     });
   }, []);
   ```

3. Update OrderGroupProps:
   ```tsx
   interface OrderGroupProps {
     order: MockOrder;  // Changed from OrderWithItems
     expanded: boolean;
     onToggleExpand: () => void;
     isSelected: boolean;
     onSelect: (orderId: string, selected: boolean) => void;
     onOrderClick?: (order: MockOrder) => void;
     onViewItemTask?: (item: MockOrderItem) => void;
   }
   ```

4. Update OrderGroup to lazy fetch:
   ```tsx
   function OrderGroup({ order, expanded, onToggleExpand, ... }) {
     const { data, isLoading } = useOrderItems(order.id);
     const items = data?.items ?? [];
     const showItems = expanded;

     return (
       <>
         <OrderRow
           order={order}
           expanded={expanded}
           onToggleExpand={onToggleExpand}
           {...otherProps}
         />
         {showItems && (
           isLoading ? (
             <LoadingItemsRow />
           ) : items.length === 0 ? (
             <EmptyItemsRow />
           ) : (
             items.map((item, idx) => (
               <OrderItemRow
                 key={item.id}
                 item={item}
                 fulfillerName={order.fulfillerName}
                 isShipped={order.status === 'SHIPPED'}
                 isDelivered={order.status === 'COMPLETED'}
                 onViewTask={onViewItemTask}
                 isLastItem={idx === items.length - 1}
               />
             ))
           )
         )}
       </>
     );
   }
   ```

5. Add helper components:
   ```tsx
   function LoadingItemsRow() {
     return (
       <tr className="bg-white">
         <td colSpan={7} className="px-4 py-3 text-center">
           <span className="text-sm text-gray-500">Loading items...</span>
         </td>
       </tr>
     );
   }

   function EmptyItemsRow() {
     return (
       <tr className="bg-white border-b border-gray-100">
         <td colSpan={7} className="px-4 py-3 text-center">
           <span className="text-sm text-gray-400 italic">No items</span>
         </td>
       </tr>
     );
   }
   ```

6. Pass expanded state to OrderGroup:
   ```tsx
   {orders.map((order) => (
     <OrderGroup
       key={order.id}
       order={order}
       expanded={expandedIds.has(order.id)}
       onToggleExpand={() => toggleExpand(order.id)}
       isSelected={selectedIds.has(order.id)}
       onSelect={handleSelectOrder}
       onOrderClick={onOrderClick}
       onViewItemTask={onViewItemTask}
     />
   ))}
   ```

## Todo List
- [x] Change OrderListTableProps.orders type to MockOrder[]
- [x] Add expandedIds state and toggleExpand function
- [x] Update OrderGroupProps interface
- [x] Import useOrderItems hook
- [x] Update OrderGroup with lazy fetch logic
- [x] Add LoadingItemsRow component
- [x] Add EmptyItemsRow component
- [x] Update OrderGroup render in map

## Success Criteria
- Items only fetched when order expanded
- Loading state visible while fetching
- Empty state shown for orders without items
- Items cached by React Query (instant on re-expand)
- Multiple orders can be expanded

## Risk Assessment
- **Medium risk:** Core logic change, need careful testing
- Ensure useOrderItems hook works correctly with enabled flag

## Security Considerations
- No sensitive data exposed
- API calls go through existing auth middleware

## Next Steps
- Phase 03 will update the page to use simpler useOrders hook
