# Phase 03: Update UI Components

## Context

- **Parent Plan:** [plan.md](./plan.md)
- **Dependencies:** [Phase 01](./phase-01-update-types-constants.md), [Phase 02](./phase-02-update-mock-data.md)
- **Related Docs:** [Brainstorm Report](../reports/brainstorm-260105-1003-simplify-priority-to-urgent-flag.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-05 |
| Priority | P1 |
| Status | ✅ Done |
| Review | ✅ Completed |
| Effort | 1h |
| Completed | 2026-01-05 |

**Description:** Cập nhật UI components để dùng isUrgent boolean, chỉ render badge khi urgent.

## Related Files

| File | Change Type |
|------|-------------|
| `src/features/design/components/priority-badge.tsx` | REWRITE → urgent-badge.tsx |
| `src/features/design/components/index.ts` | UPDATE exports |
| `src/features/design/components/kanban/task-card.tsx` | UPDATE usage |
| `src/features/design/components/drawer/task-info-section.tsx` | UPDATE usage |
| `src/features/design/components/backlog/backlog-table.tsx` | UPDATE column |
| `src/features/design/components/backlog/backlog-filter-bar.tsx` | SIMPLIFY filter |
| `src/features/design/components/backlog/backlog-action-bar.tsx` | UPDATE bulk action |
| `src/features/design/components/backlog/backlog-stats-bar.tsx` | REMOVE priority stats |
| `src/app/design-system/page.tsx` | UPDATE demo |
| `src/app/dev-tools/page.tsx` | UPDATE demo |

## Implementation Steps

### Step 1: Rewrite priority-badge.tsx → urgent-badge.tsx

```tsx
// NEW FILE: urgent-badge.tsx
'use client';

import { Flame } from 'lucide-react';
import { Badge } from '@/components/ui/badge';
import { cn } from '@/lib/utils';
import { URGENT_CONFIG } from '../lib/constants';

export interface UrgentBadgeProps {
  size?: 'sm' | 'default';
  showLabel?: boolean;
}

export function UrgentBadge({ size = 'default', showLabel = true }: UrgentBadgeProps) {
  return (
    <Badge
      variant="outline"
      className={cn(
        'gap-1 border-0 font-medium',
        URGENT_CONFIG.bgClass,
        URGENT_CONFIG.textClass,
        size === 'sm' && 'px-1.5 py-0.5 text-xs'
      )}
    >
      <Flame className={cn('h-3 w-3', size === 'sm' && 'h-2.5 w-2.5')} />
      {showLabel && <span>{URGENT_CONFIG.label}</span>}
    </Badge>
  );
}
```

### Step 2: Update exports in index.ts

```typescript
// BEFORE
export { PriorityBadge, type PriorityBadgeProps } from './priority-badge';

// AFTER
export { UrgentBadge, type UrgentBadgeProps } from './urgent-badge';
```

### Step 3: Update task-card.tsx

```tsx
// BEFORE
<PriorityBadge priority={task.priority} size="sm" />

// AFTER
{task.isUrgent && <UrgentBadge size="sm" />}
```

### Step 4: Update task-info-section.tsx

```tsx
// BEFORE
<PriorityBadge priority={task.priority} />

// AFTER
{task.isUrgent ? <UrgentBadge /> : <span className="text-muted-foreground text-sm">Normal</span>}
```

### Step 5: Update backlog-table.tsx

```tsx
// BEFORE: Priority column with dropdown
{
  accessorKey: 'priority',
  header: 'Priority',
  cell: ({ row }) => <PriorityBadge priority={row.original.priority} />,
}

// AFTER: Urgent column with simple toggle
{
  accessorKey: 'isUrgent',
  header: 'Urgent',
  cell: ({ row }) => row.original.isUrgent && <UrgentBadge />,
}
```

### Step 6: Update backlog-filter-bar.tsx

```tsx
// BEFORE: Priority filter dropdown với 4 options

// AFTER: Simple checkbox "Show Urgent Only"
<Checkbox
  id="urgent-filter"
  checked={showUrgentOnly}
  onCheckedChange={setShowUrgentOnly}
/>
<label htmlFor="urgent-filter">Urgent only</label>
```

### Step 7: Update backlog-action-bar.tsx

```tsx
// BEFORE: "Set Priority" dropdown

// AFTER: "Toggle Urgent" button
<Button onClick={handleToggleUrgent}>
  <Flame className="h-4 w-4 mr-2" />
  Toggle Urgent
</Button>
```

### Step 8: Update backlog-stats-bar.tsx

```tsx
// BEFORE: Stats per priority level (4 items)

// AFTER: Single "Urgent" count
<div>
  <Flame className="h-4 w-4 text-red-500" />
  <span>{urgentCount} urgent</span>
</div>
```

### Step 9: Update design-system/page.tsx & dev-tools/page.tsx

Cập nhật demo sections để reflect new UrgentBadge component.

## Todo List

- [ ] Rename priority-badge.tsx → urgent-badge.tsx
- [ ] Rewrite component logic
- [ ] Update index.ts exports
- [ ] Update task-card.tsx
- [ ] Update task-info-section.tsx
- [ ] Update backlog-table.tsx
- [ ] Simplify backlog-filter-bar.tsx
- [ ] Update backlog-action-bar.tsx
- [ ] Update backlog-stats-bar.tsx
- [ ] Update design-system demo
- [ ] Update dev-tools page
- [ ] Delete old priority-badge.tsx

## Success Criteria

- [ ] UrgentBadge component hoạt động
- [ ] Badge chỉ hiển thị khi `isUrgent === true`
- [ ] Backlog table hiển thị đúng
- [ ] Filter bar hoạt động
- [ ] Bulk action hoạt động
- [ ] No TypeScript errors
- [ ] UI renders correctly

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Missed component reference | Medium | Low | Thorough grep search |
| Filter logic break | Low | Medium | Test filter functionality |

## Security Considerations

None - UI changes only.

## Next Steps

After completion → Final testing & PR review
