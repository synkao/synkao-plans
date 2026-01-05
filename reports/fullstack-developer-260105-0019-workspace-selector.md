# Phase Implementation Report: Workspace Selector + Create Modal

## Executed Phase
- **Phase:** phase-02-workspace-selector
- **Plan:** `/home/tuntran/application/github.com/synkao/synkao/plans/260104-2348-fe3-remaining-issues/`
- **Status:** ✅ completed

## Files Modified

### Created (4 files)
1. `apps/web/src/features/design/components/workspace/workspace-selector.tsx` (67 lines)
   - Dropdown selector component với workspace icon + name
   - Hiển thị check mark (✓) cho current workspace
   - "+ Create Workspace" option với separator
   - Navigation sử dụng `useRouter`

2. `apps/web/src/features/design/components/workspace/create-workspace-modal.tsx` (156 lines)
   - Dialog form với react-hook-form + zod validation
   - Fields: name (required, max 50), description (optional, max 200)
   - POST /api/workspaces on submit
   - Toast success message
   - Auto navigate to new workspace
   - Loading state với spinner

3. `apps/web/src/features/design/components/workspace/index.ts` (2 lines)
   - Export WorkspaceSelector + CreateWorkspaceModal

### Modified (2 files)
1. `apps/web/src/features/design/components/index.ts`
   - Added `export * from './workspace'`

2. `apps/web/src/components/layout/app-header.tsx`
   - Added `children?: ReactNode` prop
   - Render children với separator khi có
   - Cho phép inject WorkspaceSelector vào header

## Tasks Completed

- ✅ Create workspace folder structure
- ✅ Create workspace-selector.tsx component
- ✅ Create create-workspace-modal.tsx component
- ✅ Create workspace/index.ts exports
- ✅ Update features/design/components/index.ts
- ✅ Update app-header.tsx with children support
- ✅ Run typecheck (via build)

## Tests Status

- **Type check:** ✅ pass (via `pnpm build`)
- **Unit tests:** N/A (no test script configured)
- **Integration tests:** N/A

Build output:
```
✓ Compiled successfully in 16.5s
✓ Running TypeScript ...
✓ Generating static pages (14/14) in 1037.1ms
```

## Implementation Details

### WorkspaceSelector Component
- Sử dụng `DropdownMenu` từ shadcn/ui
- Trigger: Button variant="outline" với Briefcase icon
- Hiển thị current workspace name (max-width 150px, truncate)
- List workspaces với Check icon cho current selection
- Separator trước "Create Workspace" option
- Click workspace → navigate `/design/workspace/{id}`

### CreateWorkspaceModal Component
- Sử dụng `Dialog` từ shadcn/ui
- Form validation với zod schema:
  - name: min 1, max 50 chars
  - description: optional, max 200 chars
- Submit flow:
  1. POST /api/workspaces với name, description, ownerId
  2. Show loading state (Loader2 spinner)
  3. On success: toast, reset form, close modal, navigate
  4. On error: toast error message
- Lấy ownerId từ `useUser()` hook (fallback: 'usr_admin')

### AppHeader Update
- Thay đổi minimal: thêm children prop
- Pattern: `<AppHeader>{children}</AppHeader>`
- Children được wrap với Separator để consistent spacing
- Backward compatible: existing usage không bị break

## MSW Handlers

MSW handlers đã được tạo trong Phase 1:
- ✅ GET /api/workspaces - trả về list workspaces
- ✅ POST /api/workspaces - tạo workspace mới với auto ID generation

## Usage Example

```tsx
// In workspace page
import { WorkspaceSelector, CreateWorkspaceModal } from '@/features/design/components';

function WorkspacePage() {
  const [isCreateModalOpen, setIsCreateModalOpen] = useState(false);
  const { data: workspaces } = useWorkspaces();

  return (
    <>
      <AppHeader title="Workspace">
        <WorkspaceSelector
          currentWorkspaceId={workspaceId}
          workspaces={workspaces}
          onCreateClick={() => setIsCreateModalOpen(true)}
        />
      </AppHeader>

      <CreateWorkspaceModal
        open={isCreateModalOpen}
        onOpenChange={setIsCreateModalOpen}
      />
    </>
  );
}
```

## Issues Encountered

Không có blocking issues.

**Minor notes:**
- Không có `form.tsx` và `textarea.tsx` components → sử dụng plain `label` + `Input` như login page
- Không có test script → validation qua build process

## Next Steps

### Dependencies Unblocked
Phase này độc lập, không block phases khác.

### Integration Required
Để hoàn thành end-to-end flow, cần:

1. **Update Workspace Page** (`apps/web/src/app/design/workspace/[id]/page.tsx`):
   - Fetch workspaces list
   - Pass currentWorkspaceId từ URL params
   - Integrate WorkspaceSelector vào AppHeader
   - Manage CreateWorkspaceModal state

2. **Example implementation:**
```tsx
'use client';

import { useState } from 'react';
import { useParams } from 'next/navigation';
import { useQuery } from '@tanstack/react-query';
import { AppHeader } from '@/components/layout';
import { WorkspaceSelector, CreateWorkspaceModal } from '@/features/design/components';

export default function WorkspacePage() {
  const params = useParams();
  const workspaceId = params.id as string;
  const [isCreateModalOpen, setIsCreateModalOpen] = useState(false);

  const { data } = useQuery({
    queryKey: ['workspaces'],
    queryFn: async () => {
      const res = await fetch('/api/workspaces');
      return res.json();
    },
  });

  return (
    <>
      <AppHeader title="Kanban Board">
        {data?.workspaces && (
          <WorkspaceSelector
            currentWorkspaceId={workspaceId}
            workspaces={data.workspaces}
            onCreateClick={() => setIsCreateModalOpen(true)}
          />
        )}
      </AppHeader>

      <CreateWorkspaceModal
        open={isCreateModalOpen}
        onOpenChange={setIsCreateModalOpen}
      />

      {/* Rest of workspace content */}
    </>
  );
}
```

### Follow-up Tasks
- Integrate vào workspace page để test end-to-end
- Optional: Add workspace color picker trong create modal
- Optional: Add workspace edit/delete functionality

## Unresolved Questions

Không có.
