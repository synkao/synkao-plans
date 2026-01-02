# Phase 3: Components

## Context
- Parent: [plan.md](plan.md)
- Dependencies: Phase 1, Phase 2

## Overview
| Field | Value |
|-------|-------|
| Date | 2026-01-02 |
| Priority | P2 |
| Effort | 30m |
| Status | pending |

Update hardcoded Vietnamese strings in React components.

## Target Files

### 1. user-menu.tsx
Path: `apps/web/src/components/layout/user-menu.tsx`

```typescript
// mockUser object
name: 'Nguyễn Văn A' → 'Alex Chen'
email: 'nguyenvana@synkao.com' → 'alex.chen@synkao.com'
```

### 2. task-info-section.tsx
Path: `apps/web/src/features/design/components/drawer/task-info-section.tsx`

```typescript
'Thông tin task' → 'Task Info'
'Chưa giao'      → 'Unassigned'
```

### 3. Other Components (search required)
Files detected with VN text:
- `timeline-section.tsx`
- `backlog-table.tsx`
- `workspace-card.tsx`
- `designer-filter.tsx`
- `backlog-filter-bar.tsx`
- `design-versions-section.tsx`
- `design-notes-section.tsx`
- `design-system/page.tsx`
- `lib/utils.ts`

## Implementation Steps
- [ ] Update user-menu.tsx
- [ ] Update task-info-section.tsx
- [ ] Search and update remaining component files
- [ ] Check each file for VN characters

## Success Criteria
- [ ] All component strings in English
- [ ] No Vietnamese characters in component files
- [ ] UI renders correctly
