# Phase 06: Quick Status & Links

## Context

- Parent: [plan.md](plan.md)
- Dependencies: None
- Docs: [codebase-summary.md](../../docs/codebase-summary.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P2 |
| Effort | 0.5h |
| Implementation Status | Pending |
| Review Status | Pending |

**Objective:** Add quick status action buttons và enhance TaskInfoSection với order/item links và deadline warnings.

## Key Insights

1. Quick Status: Buttons below header thay vì dropdown
2. Flow: TODO → IN_PROGRESS → REVIEW → DONE/REVISION
3. Links: Order code và item code clickable
4. Deadline: Warning badges for approaching/overdue

## Requirements

### QuickStatusActions Component

- Context-aware buttons based on current status
- Transition flow:
  - TODO: "Start Work" → IN_PROGRESS
  - IN_PROGRESS: "Submit for Review" → REVIEW
  - REVIEW: "Approve" → DONE, "Request Revision" → REVISION
  - REVISION: "Submit Again" → REVIEW
  - DONE: No actions (final state)

### Enhanced TaskInfoSection

- Order code: Clickable text link
- Item code: Clickable text link
- Deadline warning badges:
  - < 2 days: Yellow "Due Soon"
  - Overdue: Red "Overdue"

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `drawer/quick-status-actions.tsx` | Create | Status buttons |
| `drawer/task-info-section.tsx` | Modify | Add links, deadline warning |
| `task-detail-drawer.tsx` | Modify | Add QuickStatusActions |

## Implementation Steps

### Step 1: Create quick-status-actions.tsx

```tsx
export function QuickStatusActions({ task, onStatusChange }: Props) {
  const getAvailableActions = (status: string) => {
    switch (status) {
      case 'TODO': return [{ label: 'Start Work', nextStatus: 'IN_PROGRESS' }];
      case 'IN_PROGRESS': return [{ label: 'Submit', nextStatus: 'REVIEW' }];
      case 'REVIEW': return [
        { label: 'Approve', nextStatus: 'DONE', variant: 'default' },
        { label: 'Request Revision', nextStatus: 'REVISION', variant: 'outline' }
      ];
      case 'REVISION': return [{ label: 'Submit Again', nextStatus: 'REVIEW' }];
      default: return [];
    }
  };

  return (
    <div className="flex gap-2">
      {getAvailableActions(task.status).map(action => (
        <Button key={action.label} variant={action.variant} size="sm">
          {action.label}
        </Button>
      ))}
    </div>
  );
}
```

### Step 2: Update task-info-section.tsx

Add order/item links và deadline warning logic.

```tsx
// Deadline warning logic
const getDeadlineWarning = (dueDate: string | undefined) => {
  if (!dueDate) return null;
  const due = new Date(dueDate);
  const now = new Date();
  const daysLeft = Math.ceil((due.getTime() - now.getTime()) / (1000 * 60 * 60 * 24));

  if (daysLeft < 0) return { type: 'error', text: 'Overdue' };
  if (daysLeft < 2) return { type: 'warning', text: 'Due Soon' };
  return null;
};
```

### Step 3: Add to drawer header

Insert QuickStatusActions below header content.

## Todo List

- [ ] Create quick-status-actions.tsx
- [ ] Add deadline warning logic
- [ ] Add order/item links
- [ ] Style warning badges
- [ ] Add to drawer
- [ ] Test status transitions

## Success Criteria

- [ ] Status buttons display correctly
- [ ] Buttons match current status context
- [ ] Order/item codes clickable
- [ ] Deadline warnings show correctly
- [ ] Overdue styling visible

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Status mismatch | Low | Validate transitions |
| Link navigation | Low | Use mock data lookup |

## Security Considerations

None - demo UI only.

## Next Steps

→ Implementation complete
