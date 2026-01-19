# Phase 01: OrderRow Expand Toggle

## Context Links
- Parent: [plan.md](plan.md)
- File: `apps/web/src/features/orders/components/order-list/order-row.tsx`

## Overview
- **Priority:** P1
- **Status:** Completed
- **Description:** Add expand/collapse toggle button to OrderRow component

## Key Insights
- Current OrderRow has no expand functionality
- Need chevron icon that rotates on expand state
- Click on chevron should NOT trigger row click (navigation)

## Requirements

### Functional
- Add chevron button (right or left side)
- Chevron rotates: ChevronRight (collapsed) → ChevronDown (expanded)
- Stop propagation on chevron click

### Non-Functional
- Smooth rotation animation
- Accessible (aria-expanded, aria-label)

## Related Code Files

**Modify:**
- `apps/web/src/features/orders/components/order-list/order-row.tsx`

## Implementation Steps

1. Add new props to `OrderRowProps`:
   ```tsx
   expanded: boolean;
   onToggleExpand: () => void;
   ```

2. Import ChevronRight, ChevronDown from lucide-react

3. Add expand button in first column (after checkbox):
   ```tsx
   <td className="w-8 px-2">
     <button
       onClick={(e) => {
         e.stopPropagation();
         onToggleExpand();
       }}
       className="p-1 hover:bg-gray-100 rounded transition-transform"
       aria-expanded={expanded}
       aria-label={expanded ? 'Collapse order' : 'Expand order'}
     >
       {expanded ? (
         <ChevronDown className="h-4 w-4 text-gray-500" />
       ) : (
         <ChevronRight className="h-4 w-4 text-gray-500" />
       )}
     </button>
   </td>
   ```

4. Update table header in OrderListTable to add column for chevron

## Todo List
- [x] Add `expanded` and `onToggleExpand` props to OrderRowProps
- [x] Import chevron icons
- [x] Add expand button column
- [x] Add corresponding th in table header
- [x] Test click behavior (doesn't navigate)

## Success Criteria
- Chevron visible on each order row
- Click toggles visual state (passed from parent)
- Row click still works for navigation
- Chevron click does NOT navigate

## Risk Assessment
- **Low risk:** Simple UI addition, no state logic yet

## Next Steps
- Phase 02 will handle the state management and item fetching
