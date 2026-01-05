# Phase 01: Backlog Bulk Actions

## Context
- Parent: [plan.md](./plan.md)
- Issue: [#17](https://github.com/synkao/synkao/issues/17)
- Brainstorm: [fe3-milestone-review](../reports/brainstorm-260104-2326-fe3-milestone-review.md)

## Overview
- **Priority:** P0
- **Effort:** 3-4h
- **Status:** Pending

Thêm action bar hiện khi user select tasks trong backlog table. Actions: Add to Workspace, Assign Designer, Set Priority.

## Key Insights

1. `BacklogTable` đã có `selectedIds` state - cần lift lên parent hoặc expose via callback
2. User đã confirm: direct add (no confirmation modal), show toast
3. Mock data có sẵn: `mockWorkspaces`, `mockUsers` với `getDesigners()`
4. UI components available: `DropdownMenu`, `Select`, `Button`

## Requirements

### Functional
- [ ] Show action bar khi `selectedIds.size > 0`
- [ ] Display selected count: "X selected"
- [ ] Add to Workspace dropdown - list workspaces
- [ ] Assign Designer dropdown - list designers
- [ ] Set Priority dropdown - LOW/NORMAL/HIGH/URGENT
- [ ] Clear selection button
- [ ] Toast notification on success

### Non-Functional
- Sticky positioning (bottom of table container)
- Animate in/out với opacity transition
- Match design system (glass effect)

## Architecture

```
┌─────────────────────────────────────────────────────┐
│ BacklogPage                                         │
│ ┌─────────────────────────────────────────────────┐ │
│ │ BacklogTable                                    │ │
│ │ - selectedIds ← lift to parent                  │ │
│ │ - onSelectionChange(ids)                        │ │
│ └─────────────────────────────────────────────────┘ │
│ ┌─────────────────────────────────────────────────┐ │
│ │ BacklogActionBar (conditional render)           │ │
│ │ - selectedCount                                 │ │
│ │ - onAddToWorkspace(workspaceId)                 │ │
│ │ - onAssignDesigner(designerId)                  │ │
│ │ - onSetPriority(priority)                       │ │
│ │ - onClearSelection()                            │ │
│ └─────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

## Related Code Files

### Modify
| File | Changes |
|------|---------|
| `features/design/components/backlog/backlog-table.tsx` | Add `onSelectionChange` prop, expose `selectedIds` |
| `features/design/components/backlog/index.ts` | Export `BacklogActionBar` |
| `app/(main)/design/backlog/page.tsx` | Add selection state, render action bar |

### Create
| File | Description |
|------|-------------|
| `features/design/components/backlog/backlog-action-bar.tsx` | Action bar component |
| `mocks/handlers/workspaces.ts` | MSW handlers for bulk operations |

## Implementation Steps

### Step 1: Create BacklogActionBar Component
```tsx
// backlog-action-bar.tsx
interface BacklogActionBarProps {
  selectedCount: number;
  workspaces: MockWorkspace[];
  designers: MockUser[];
  onAddToWorkspace: (workspaceId: string) => void;
  onAssignDesigner: (designerId: string) => void;
  onSetPriority: (priority: PriorityType) => void;
  onClearSelection: () => void;
}
```

Layout:
```
┌─────────────────────────────────────────────────────────┐
│ ✓ 3 selected  [Add to Workspace ▼] [Assign ▼] [Priority ▼]  [× Clear] │
└─────────────────────────────────────────────────────────┘
```

Sử dụng:
- `DropdownMenu` cho các action dropdowns
- `Button variant="ghost"` cho Clear
- Glass effect: `bg-white/80 backdrop-blur-md border`

### Step 2: Update BacklogTable
- Add `onSelectionChange?: (ids: Set<string>) => void` prop
- Call `onSelectionChange` trong `handleSelectAll` và `handleSelectOne`
- Keep internal state cho checkbox UI, sync via callback

### Step 3: Update BacklogPage
- Lift `selectedIds` state lên page level
- Conditionally render `BacklogActionBar` when `selectedIds.size > 0`
- Implement bulk action handlers (call MSW endpoints)
- Show toast on success

### Step 4: Add MSW Handlers
```ts
// mocks/handlers/workspaces.ts
POST /api/tasks/bulk/assign-workspace
POST /api/tasks/bulk/assign-designer
POST /api/tasks/bulk/set-priority
```

### Step 5: Wire up Toast
- Use existing toast system hoặc add `sonner` if not present
- Success: "X tasks updated"

## Todo List

- [ ] Create `backlog-action-bar.tsx` component
- [ ] Update `BacklogTable` với `onSelectionChange` callback
- [ ] Update `index.ts` exports
- [ ] Update `BacklogPage` với selection state + action bar
- [ ] Create MSW handlers `workspaces.ts`
- [ ] Add toast notifications
- [ ] Test: select → action → verify toast

## Success Criteria

- [ ] Action bar appears when 1+ tasks selected
- [ ] All 3 action dropdowns work correctly
- [ ] Clear selection clears all checkboxes
- [ ] Toast shows after each action
- [ ] Smooth enter/exit animation

## Risk Assessment

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Selection state sync issues | Low | Medium | Use controlled pattern |
| DropdownMenu z-index conflicts | Low | Low | Check stacking context |

## Security Considerations

- Validate task IDs on backend (MSW mock)
- Sanitize user input in designer/workspace IDs

## Next Steps

After completion:
1. Close issue #17
2. Proceed to Phase 02: Workspace Selector
