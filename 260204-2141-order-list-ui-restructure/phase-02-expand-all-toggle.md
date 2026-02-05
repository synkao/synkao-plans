# Phase 02: Toggle Expand/Close All

**Parent:** [plan.md](./plan.md)
**Dependencies:** None

## Overview

| Field | Value |
|-------|-------|
| Priority | P2 |
| Status | ✅ completed |
| Effort | 1h |

Add button in table header to expand/collapse all orders at once.

## Key Insights

- Current: `expandedIds: Set<string>` tracks individual expanded orders
- Need: Toggle button `▶` that switches to `▼` when all expanded
- Location: Replace empty expand column header

## Requirements

### Functional
- Add toggle button in header (first column)
- Click expands all orders → button shows `▼`
- Click again collapses all → button shows `▶`
- Visual feedback: button has hover state

### Non-functional
- Performance: Consider lazy loading impact when expanding many orders
- A11y: Button should have aria-label

## Architecture

```
┌────┬────┬─────────┐
│ ▶  │ □  │ ORDER   │  ← Button in header
└────┴────┴─────────┘

State: allExpanded = expandedIds.size === orders.length
Toggle: setExpandedIds(allExpanded ? new Set() : new Set(orders.map(o => o.id)))
```

## Related Code Files

| File | Action |
|------|--------|
| `apps/web/src/features/orders/components/order-list/order-list-table.tsx` | Modify |

## Implementation Steps

1. Add `allExpanded` derived state: `expandedIds.size === orders.length`
2. Add `handleToggleExpandAll` handler:
   ```typescript
   const handleToggleExpandAll = () => {
     if (allExpanded) {
       setExpandedIds(new Set());
     } else {
       setExpandedIds(new Set(orders.map(o => o.id)));
     }
   };
   ```
3. Replace empty `<th>` for expand column with toggle button
4. Use `ChevronDown` when all expanded, `ChevronRight` otherwise
5. Add hover styles and aria-label

## Todo List

- [x] Add allExpanded derived state
- [x] Add handleToggleExpandAll handler
- [x] Replace expand header with toggle button
- [x] Add appropriate icons and styles
- [x] Add aria-label for accessibility

## Success Criteria

- Button visible in table header
- Click expands all orders
- Click again collapses all
- Icon changes based on state
- Keyboard accessible

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Performance with 100+ orders | Medium | Medium | Lazy loading already in place |
| UI flash on expand all | Low | Low | React batches state updates |

## Next Steps

→ Phase 03: External ID + Platform icon in OrderRow
