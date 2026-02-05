# Phase 04: Item Row DESIGN & TASK Columns

**Parent:** [plan.md](./plan.md)
**Dependencies:** Phase 01, 02, 03

## Overview

| Field | Value |
|-------|-------|
| Priority | P1 |
| Status | ✅ completed |
| Effort | 2h |

Add separate DESIGN and TASK columns to order item rows with proper status display and actions.

## Key Insights

- Current `OrderItemRow` has: thumbnail, product info, qty, fulfiller, action
- Need to add: DESIGN column, TASK column
- `designStatus` exists in MockOrderItem: `NOT_REQUIRED | PENDING | IN_PROGRESS | APPROVED`
- Task data is separate entity, linked via `orderItemId`
- Need to fetch task status per item or pass from parent

## Requirements

### Functional

**DESIGN Column:**
| Condition | Display |
|-----------|---------|
| `designStatus` = APPROVED or IN_PROGRESS | `—` (dash) |
| `designStatus` = PENDING | `⚠ Missing Design` (red badge) |
| `designStatus` = NOT_REQUIRED | `—` (dash) |

**TASK Column:**
| Condition | Display |
|-----------|---------|
| Has task - IN_PROGRESS | `In Progress` (purple badge) |
| Has task - REVIEW | `Review` (amber badge) |
| Has task - APPROVED | `✓ Approved` (green badge) |
| Has task - other status | Status badge |
| No task | `+ Create Task` (dashed button) |

**Actions:**
- `+ Create Task` button triggers task creation (TBD: modal or navigation)

### Non-functional
- Task fetching should be efficient (batch with items or separate endpoint)

## Architecture

**Item Row Structure:**
```
| Tree | Image | Name/SKU | Qty | DESIGN | TASK | FF | Action |
| ├─   | [📷]  | Product  | x2  | —      | ✓    |    | 👁     |
```

**Task Status Badge Component:**
```typescript
interface TaskStatusBadgeProps {
  status?: DesignTaskStatusType;
  onCreateTask?: () => void;
}
```

**Task Status Colors:**
```typescript
const TASK_STATUS_COLORS = {
  IN_PROGRESS: 'bg-purple-100 text-purple-700',
  REVIEW: 'bg-amber-100 text-amber-700',
  APPROVED: 'bg-green-100 text-green-700',
  TODO: 'bg-gray-100 text-gray-600',
  BLOCKED: 'bg-red-100 text-red-700',
  REVISION: 'bg-orange-100 text-orange-700',
};
```

## Related Code Files

| File | Action |
|------|--------|
| `apps/web/src/features/orders/components/order-list/order-item-row.tsx` | Modify |
| `apps/web/src/features/orders/components/order-list/design-status-cell.tsx` | Create |
| `apps/web/src/features/orders/components/order-list/task-status-cell.tsx` | Create |
| `apps/web/src/features/orders/lib/constants.ts` | Modify (add task colors) |
| `apps/web/src/features/orders/types/order.types.ts` | Modify (extend item with task) |

## Implementation Steps

1. **Create DesignStatusCell component:**
   - Props: `designStatus: ItemDesignStatusType`
   - Render `—` or `⚠ Missing Design` badge based on status

2. **Create TaskStatusCell component:**
   - Props: `task?: { status: DesignTaskStatusType }`, `onCreateTask?: () => void`
   - Render status badge or `+ Create Task` button
   - Style button with dashed border

3. **Add task status colors to constants:**
   - Add `TASK_STATUS_COLORS` mapping
   - Add `DESIGN_STATUS_CONFIG` if needed

4. **Update OrderItemRow:**
   - Add props: `task?: MockTask` (or just status)
   - Add DESIGN column after Qty
   - Add TASK column after DESIGN
   - Adjust column spans

5. **Update OrderListTable:**
   - Update item rendering to pass task data
   - Handle `onCreateTask` callback

6. **Update table header:**
   - Add DESIGN and TASK column headers in item section (or adjust layout)

## Todo List

- [x] Create DesignStatusCell component
- [x] Create TaskStatusCell component
- [x] Add task status colors to constants
- [x] Update OrderItemRow with new columns
- [ ] **BLOCKED:** Update OrderListTable to pass task data
- [x] Implement `+ Create Task` click handler (stub)
- [x] Adjust column widths/spans
- [ ] **BLOCKED:** Test all status variants (waiting for task data)

## Success Criteria

- DESIGN column shows correct status
- TASK column shows correct badges
- `+ Create Task` button visible when no task
- `+ Create Task` button is clickable
- Column alignment is correct

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Task data not available | Medium | High | Fetch with items or add endpoint |
| Column width overflow | Low | Medium | Test with long product names |
| Create task flow undefined | Medium | Medium | Stub handler, implement later |

## Security Considerations

- Task status is display-only
- Create task action should validate permissions

## Open Questions

1. **Task fetching strategy:** Include task in order items API or separate fetch?
   - Recommendation: Include task status in order items response (1:1 relationship exists)

2. **Create Task flow:** Modal or navigation?
   - Recommendation: Modal for quick creation, consistent with existing patterns

## Next Steps

→ Testing and polish
→ Implement Create Task modal (separate issue/phase if complex)
