# Phase 01: Update BacklogStatsBar Component

## Context

- **Parent Plan:** [plan.md](./plan.md)
- **Brainstorm:** [brainstorm report](../reports/brainstorm-260105-1059-backlog-stats-cards.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-05 |
| Priority | P3 |
| Effort | 30m |
| Status | ⏳ Pending |

## Requirements

1. Thêm 2 cards mới: Redesign, Avg. Wait
2. Update layout từ 2 cols → 4 cols (responsive)
3. Avg. Wait hiển thị format giờ (e.g., "36h")
4. Giữ nguyên styling hiện tại

## Related Files

| File | Action |
|------|--------|
| `apps/web/src/features/design/components/backlog/backlog-stats-bar.tsx` | Edit |
| `apps/web/src/features/design/lib/constants.ts` | Read (SOURCE_CONFIG) |

## Implementation Steps

### Step 1: Add imports

```typescript
// Thêm icons
import { Flame, FileStack, RefreshCw, Clock } from 'lucide-react';

// Thêm constants
import { URGENT_CONFIG, SOURCE_CONFIG } from '../../lib/constants';
import { TaskSource } from '@/mocks/types';
```

### Step 2: Add calculations

```typescript
const totalCount = tasks.length;
const urgentCount = tasks.filter((t) => t.isUrgent).length;
const redesignCount = tasks.filter((t) => t.source === TaskSource.REDESIGN).length;

// Avg wait in hours
const avgWaitHours = tasks.length > 0
  ? Math.round(
      tasks.reduce((sum, t) => {
        const waitMs = Date.now() - new Date(t.createdAt).getTime();
        return sum + waitMs;
      }, 0) / tasks.length / (1000 * 60 * 60)
    )
  : null;
```

### Step 3: Update layout

```typescript
// From:
<div className="grid grid-cols-2 gap-4 max-w-md">

// To:
<div className="grid grid-cols-2 md:grid-cols-4 gap-4 max-w-2xl">
```

### Step 4: Add Redesign card

```tsx
{/* Redesign tasks */}
<Card className="border border-gray-200/50 bg-white/60 p-4 backdrop-blur-md">
  <div className="flex items-center gap-3">
    <div className={`rounded-lg ${SOURCE_CONFIG.REDESIGN.bgClass} p-2`}>
      <RefreshCw className={`h-5 w-5 ${SOURCE_CONFIG.REDESIGN.textClass}`} />
    </div>
    <div>
      <p className="text-xs text-gray-500">Redesign</p>
      <p className="text-2xl font-semibold text-gray-900">{redesignCount}</p>
    </div>
  </div>
</Card>
```

### Step 5: Add Avg. Wait card

```tsx
{/* Avg wait time */}
<Card className="border border-gray-200/50 bg-white/60 p-4 backdrop-blur-md">
  <div className="flex items-center gap-3">
    <div className="rounded-lg bg-gray-50 p-2">
      <Clock className="h-5 w-5 text-gray-600" />
    </div>
    <div>
      <p className="text-xs text-gray-500">Avg. Wait</p>
      <p className="text-2xl font-semibold text-gray-900">
        {avgWaitHours !== null ? `${avgWaitHours}h` : '-'}
      </p>
    </div>
  </div>
</Card>
```

## Todo List

- [ ] Add new icon imports (RefreshCw, Clock)
- [ ] Add TaskSource import
- [ ] Add SOURCE_CONFIG import
- [ ] Calculate redesignCount
- [ ] Calculate avgWaitHours
- [ ] Update grid layout
- [ ] Add Redesign card JSX
- [ ] Add Avg. Wait card JSX
- [ ] Test responsive layout

## Success Criteria

- [ ] 4 cards render correctly
- [ ] Redesign count matches filter
- [ ] Avg. Wait shows hours format
- [ ] Responsive: 2x2 on mobile, 4 cols on desktop
- [ ] No TypeScript errors
- [ ] Existing styling preserved

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| createdAt timezone issues | Low | Low | Ensure UTC |
| Empty tasks array | Low | Low | Handle null case |

## Security Considerations

- No user input handling
- No API calls
- Pure UI component

## Next Steps

After completion:
1. Visual testing on `/design/backlog`
2. Check responsive breakpoints
3. Update plan status to completed
