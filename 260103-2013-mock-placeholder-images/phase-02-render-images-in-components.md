# Phase 02: Render Images in UI Components

## Context

- **Parent Plan:** [plan.md](plan.md)
- **Dependency:** Phase 01 (completed)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-03 |
| Priority | P3 |
| Implementation | ✅ completed |
| Review | ✅ completed |

## Key Insights

- Mock data đã có image URLs từ Phase 01
- Components chưa render các URLs này
- Cần update task-card và user-menu

## Requirements

1. Render `task.designUrl` trong task-card thumbnail
2. Render `assignee.avatar` trong task-card
3. Update user-menu để import từ mockUsers

## Related Files

- `apps/web/src/features/design/components/kanban/task-card.tsx`
- `apps/web/src/components/layout/user-menu.tsx`

## Implementation Steps

### Step 1: Update task-card.tsx

**Before:**
```tsx
{/* Thumbnail placeholder */}
<div className="mb-3 flex h-24 items-center justify-center rounded-md bg-gray-100/50">
  <Image className="h-8 w-8 text-gray-400" />
</div>
```

**After:**
```tsx
{/* Design thumbnail */}
<div className="mb-3 h-24 rounded-md bg-gray-100/50 overflow-hidden">
  {task.designUrl ? (
    <img src={task.designUrl} alt={task.productName} className="h-full w-full object-cover" />
  ) : (
    <div className="flex h-full items-center justify-center">
      <ImageIcon className="h-8 w-8 text-gray-400" />
    </div>
  )}
</div>
```

### Step 2: Update Avatar in task-card.tsx

**Before:**
```tsx
<Avatar className="h-5 w-5">
  <AvatarFallback className="text-[10px]">
    {getInitials(assignee.name)}
  </AvatarFallback>
</Avatar>
```

**After:**
```tsx
<Avatar className="h-5 w-5">
  <AvatarImage src={assignee.avatar} alt={assignee.name} />
  <AvatarFallback className="text-[10px]">
    {getInitials(assignee.name)}
  </AvatarFallback>
</Avatar>
```

### Step 3: Update user-menu.tsx

Import from mockUsers và use avatar URL.

## Todo List

- [x] Render designUrl in task-card thumbnail
- [x] Add AvatarImage in task-card assignee
- [x] Update user-menu to use mockUsers
- [x] Verify images load correctly

## Success Criteria

- [x] Task cards show design thumbnails
- [x] Assignee avatars show DiceBear images
- [x] User menu shows avatar
- [x] No broken images

## Code Review

- **Report:** [code-reviewer-260103-2143-phase02-images.md](../reports/code-reviewer-260103-2143-phase02-images.md)
- **Status:** ✅ APPROVED với minor recommendations
- **Key Findings:**
  - Next.js Image optimization warning (acceptable for mock phase)
  - Duplicate getInitials logic (low priority)
  - Build passes successfully

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| CORS issues | Low | External services support CORS |
| Slow loading | Low | Images are small, services have CDN |

## Security Considerations

None - rendering external placeholder images only.

## Next Steps

1. Implement changes
2. Visual test in browser
3. Verify all images load
