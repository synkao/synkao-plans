# Phase 02: Workspace Selector + Create Modal

## Context
- Parent: [plan.md](./plan.md)
- Issue: [#18](https://github.com/synkao/synkao/issues/18)
- Depends: Phase 01 (optional, can parallel)

## Overview
- **Priority:** P1
- **Effort:** 3h
- **Status:** Pending

Thêm workspace selector dropdown trong header và modal tạo workspace mới.

## Key Insights

1. `AppHeader` hiện tại chỉ có title + notification bell
2. `mockWorkspaces` có 2 workspaces: Main + Archive
3. User confirmed: chỉ cần name + description, columns auto-create
4. UI: `DropdownMenu` cho selector, `Dialog` cho modal
5. Cần detect current workspace từ URL `/design/workspace/[id]`

## Requirements

### Functional
- [ ] Dropdown trong header showing current workspace name
- [ ] List workspaces với click to navigate
- [ ] "+ Create Workspace" option
- [ ] Modal với form: name (required), description (optional)
- [ ] On create: add workspace + navigate to new workspace
- [ ] Default columns auto-created: TODO, IN_PROGRESS, REVIEW, DONE

### Non-Functional
- Dropdown trigger shows workspace icon + name
- Match glassmorphism design system
- Form validation: name required, max 50 chars

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│ AppHeader                                                 │
│ ┌──────────────────────┐ ┌────────────────────────────┐  │
│ │ SidebarTrigger       │ │ WorkspaceSelector          │  │
│ └──────────────────────┘ │ ┌────────────────────────┐ │  │
│                          │ │ Current: Main Workspace│ │  │
│                          │ │ ────────────────────── │ │  │
│                          │ │ ○ Main Workspace       │ │  │
│                          │ │ ○ Archive              │ │  │
│                          │ │ ────────────────────── │ │  │
│                          │ │ + Create Workspace     │ │  │
│                          │ └────────────────────────┘ │  │
│                          └────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘

CreateWorkspaceModal:
┌─────────────────────────────────────┐
│ Create Workspace              [×]   │
├─────────────────────────────────────┤
│ Name *                              │
│ ┌─────────────────────────────────┐ │
│ │ My New Workspace                │ │
│ └─────────────────────────────────┘ │
│                                     │
│ Description                         │
│ ┌─────────────────────────────────┐ │
│ │ Optional description...         │ │
│ └─────────────────────────────────┘ │
│                                     │
│           [Cancel] [Create]         │
└─────────────────────────────────────┘
```

## Related Code Files

### Create
| File | Description |
|------|-------------|
| `features/design/components/workspace/workspace-selector.tsx` | Dropdown selector component |
| `features/design/components/workspace/create-workspace-modal.tsx` | Create modal with form |
| `features/design/components/workspace/index.ts` | Exports |

### Modify
| File | Changes |
|------|---------|
| `components/layout/app-header.tsx` | Add WorkspaceSelector |
| `components/layout/index.ts` | May need update |
| `features/design/components/index.ts` | Export workspace components |
| `mocks/handlers/workspaces.ts` | Add GET/POST workspace handlers |
| `mocks/data/workspaces.data.ts` | Add helper functions |

## Implementation Steps

### Step 1: Create workspace folder structure
```
features/design/components/workspace/
├── workspace-selector.tsx
├── create-workspace-modal.tsx
└── index.ts
```

### Step 2: Create WorkspaceSelector Component
```tsx
interface WorkspaceSelectorProps {
  currentWorkspaceId?: string;
  workspaces: MockWorkspace[];
  onSelect: (workspaceId: string) => void;
  onCreateClick: () => void;
}
```

Implementation notes:
- Use `DropdownMenu` from shadcn/ui
- Trigger: `Button variant="outline"` với workspace icon + name
- Check icon (✓) next to current workspace
- Separator before "Create Workspace"
- `useRouter` for navigation on select

### Step 3: Create CreateWorkspaceModal Component
```tsx
interface CreateWorkspaceModalProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  onCreated: (workspace: MockWorkspace) => void;
}

// Form schema
const createWorkspaceSchema = z.object({
  name: z.string().min(1, 'Name required').max(50),
  description: z.string().max(200).optional(),
});
```

Implementation notes:
- Use `Dialog` from shadcn/ui
- `react-hook-form` + `zod` for validation
- Call POST /api/workspaces on submit
- Navigate to new workspace on success

### Step 4: Update AppHeader
```tsx
// Add workspace selector after title
<AppHeader title={title}>
  {showWorkspaceSelector && (
    <WorkspaceSelector
      currentWorkspaceId={workspaceId}
      workspaces={workspaces}
      onSelect={handleSelect}
      onCreateClick={() => setCreateModalOpen(true)}
    />
  )}
</AppHeader>
```

Need to:
- Pass `currentWorkspaceId` from page
- Fetch workspaces list
- Handle create modal state

### Step 5: Add MSW Handlers
```ts
// GET /api/workspaces
http.get('/api/workspaces', () => {
  return HttpResponse.json({ workspaces: mockWorkspaces });
});

// POST /api/workspaces
http.post('/api/workspaces', async ({ request }) => {
  const body = await request.json();
  const newWorkspace = {
    id: `ws-${Date.now()}`,
    ...body,
    columns: ['TODO', 'IN_PROGRESS', 'REVIEW', 'DONE'],
  };
  return HttpResponse.json(newWorkspace, { status: 201 });
});
```

### Step 6: Integrate with Workspace Page
- Pass `workspaceId` prop to header
- Detect from URL params: `useParams()`

## Todo List

- [ ] Create `workspace/` folder structure
- [ ] Create `workspace-selector.tsx`
- [ ] Create `create-workspace-modal.tsx`
- [ ] Create `workspace/index.ts`
- [ ] Update `features/design/components/index.ts`
- [ ] Add MSW handlers GET/POST workspaces
- [ ] Update `app-header.tsx` với optional selector
- [ ] Update workspace pages to pass context
- [ ] Test: navigation, create flow

## Success Criteria

- [ ] Dropdown shows current workspace when on workspace page
- [ ] Click workspace navigates to that workspace
- [ ] Create modal opens, validates, submits
- [ ] New workspace appears in dropdown after create
- [ ] Navigates to new workspace after create

## Risk Assessment

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| URL context detection | Low | Medium | Use `usePathname` + `useParams` |
| Modal focus trap issues | Low | Low | shadcn Dialog handles this |
| State sync after create | Medium | Medium | Refetch or local update |

## Security Considerations

- Validate workspace name input
- Sanitize description (XSS prevention)
- Check user permission for workspace creation (future)

## Next Steps

After completion:
1. Close issue #18
2. Close FE-3 milestone
3. End-to-end test: Backlog → Add to Workspace → Kanban board
