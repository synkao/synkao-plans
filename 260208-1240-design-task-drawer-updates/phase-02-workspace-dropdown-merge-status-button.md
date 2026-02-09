# Phase 02: Workspace Dropdown + Merge QuickStatusActions

## Context Links

- Issue: #80 - Fix Design Task (TaskDetailDrawer)
- [plan.md](./plan.md)
- [Phase 01](./phase-01-remove-print-specs-add-description.md) (prerequisite)
- Drawer: `apps/web/src/features/design/components/drawer/`
- AlertDialog: `apps/web/src/components/ui/alert-dialog.tsx`

## Overview

- **Priority**: P2
- **Status**: pending
- **Description**: Add workspace change dropdown with confirmation dialog next to taskCode in drawer header. Merge QuickStatusActions from 2 buttons (primary action + chevron dropdown) into 1 unified dropdown button.

## Key Insights

- AlertDialog component already exists at `components/ui/alert-dialog.tsx`
- mockWorkspaces has 2 entries: "Main Workspace" and "Archive"
- Current QuickStatusActions: Button (Start/Submit/Approve/Restart) + separate DropdownMenu trigger (ChevronDown)
- Target QuickStatusActions: single Button with label + ChevronDown endIcon, click opens dropdown with all statuses
- task-info-section.tsx already shows workspace read-only (line 136-143) -- keep that, the new dropdown is in SheetHeader
- WorkspaceChangeDropdown needs its own file due to AlertDialog state complexity (~80-100 LOC)

## Requirements

### Functional
- Workspace dropdown in SheetHeader next to task.taskCode
- Shows all mockWorkspaces, current workspace has check indicator
- Selecting different workspace triggers AlertDialog confirmation
- On confirm: console.log workspace change (demo), close dialog
- On cancel: close dialog, no change
- QuickStatusActions: 1 button replaces 2 buttons
- Button shows primary action label + ChevronDown icon
- Click opens DropdownMenu with all statuses (current marked/disabled)
- When status is APPROVED, show "Approved" disabled button (no dropdown)

### Non-functional
- WorkspaceChangeDropdown under 100 LOC
- QuickStatusActions refactored in-place (no new file)

## Architecture

### Workspace Change Dropdown

```
WorkspaceChangeDropdown
  props: { currentWorkspaceId?: string, onWorkspaceChange?: (id: string) => void }

  DropdownMenu
    trigger: Button(variant=outline, size=sm) showing current workspace name or "No Workspace"
    content: mockWorkspaces.map(ws => DropdownMenuItem)
      - Check icon for current workspace
      - onClick → if different, set pendingWorkspace state

  AlertDialog (controlled by pendingWorkspace state)
    title: "Change Workspace?"
    description: "Move this task from {current} to {selected}?"
    cancel: "Cancel"
    action: "Confirm" → onWorkspaceChange(pendingId), clear pending
```

### Merged QuickStatusActions

```
Before:
  [Start Button] [v Dropdown]

After:
  [Start v]  ← single button that opens dropdown

  DropdownMenu
    trigger: Button(size=sm) with "Start" label + ChevronDown endIcon
    content: ALL_STATUSES.map(status =>
      DropdownMenuItem(disabled if current) with icon + label
    )

  Special case APPROVED:
    render Badge("Approved") instead of dropdown
```

## Related Code Files

### Files to CREATE
- `apps/web/src/features/design/components/drawer/workspace-change-dropdown.tsx`
  - New component with DropdownMenu + AlertDialog
  - Props: currentWorkspaceId, onWorkspaceChange callback

### Files to MODIFY
1. `apps/web/src/features/design/components/drawer/task-detail-drawer.tsx`
   - Import WorkspaceChangeDropdown
   - Add next to task.taskCode in SheetHeader
   - Layout: taskCode line becomes `<div className="flex items-center gap-2"><p>{taskCode}</p><WorkspaceChangeDropdown .../></div>`

2. `apps/web/src/features/design/components/drawer/quick-status-actions.tsx`
   - Remove primary Button (lines 68-77)
   - Replace with single DropdownMenu: trigger is a Button with primary action label + ChevronDown
   - Keep ALL_STATUSES array, keep handleClick
   - Remove getPrimaryAction function, simplify to just getButtonLabel
   - Special case: APPROVED status renders static Badge, no dropdown

3. `apps/web/src/features/design/components/drawer/index.ts`
   - Add export for WorkspaceChangeDropdown

## Implementation Steps

### Step 1: Create workspace-change-dropdown.tsx

```tsx
'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/button';
import {
  DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import {
  AlertDialog, AlertDialogAction, AlertDialogCancel, AlertDialogContent,
  AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle,
} from '@/components/ui/alert-dialog';
import { Building2, Check, ChevronDown } from 'lucide-react';
import { mockWorkspaces } from '@/mocks/data';

export interface WorkspaceChangeDropdownProps {
  currentWorkspaceId?: string;
  onWorkspaceChange?: (workspaceId: string) => void;
}

export function WorkspaceChangeDropdown({
  currentWorkspaceId,
  onWorkspaceChange,
}: WorkspaceChangeDropdownProps) {
  const [pendingWorkspaceId, setPendingWorkspaceId] = useState<string | null>(null);

  const currentWorkspace = currentWorkspaceId
    ? mockWorkspaces.find((w) => w.id === currentWorkspaceId)
    : null;
  const pendingWorkspace = pendingWorkspaceId
    ? mockWorkspaces.find((w) => w.id === pendingWorkspaceId)
    : null;

  const handleSelect = (wsId: string) => {
    if (wsId === currentWorkspaceId) return;
    setPendingWorkspaceId(wsId);
  };

  const handleConfirm = () => {
    if (pendingWorkspaceId) {
      console.log(`Workspace: ${currentWorkspaceId} -> ${pendingWorkspaceId}`);
      onWorkspaceChange?.(pendingWorkspaceId);
    }
    setPendingWorkspaceId(null);
  };

  return (
    <>
      <DropdownMenu>
        <DropdownMenuTrigger asChild>
          <Button variant="outline" size="sm" className="h-6 text-xs gap-1 px-2">
            <Building2 className="h-3 w-3" />
            {currentWorkspace?.name ?? 'No Workspace'}
            <ChevronDown className="h-3 w-3" />
          </Button>
        </DropdownMenuTrigger>
        <DropdownMenuContent align="start">
          {mockWorkspaces.map((ws) => (
            <DropdownMenuItem
              key={ws.id}
              onClick={() => handleSelect(ws.id)}
              className="gap-2 text-xs"
            >
              {ws.id === currentWorkspaceId && <Check className="h-3 w-3" />}
              {ws.id !== currentWorkspaceId && <span className="w-3" />}
              {ws.name}
            </DropdownMenuItem>
          ))}
        </DropdownMenuContent>
      </DropdownMenu>

      <AlertDialog open={!!pendingWorkspaceId} onOpenChange={(open) => !open && setPendingWorkspaceId(null)}>
        <AlertDialogContent>
          <AlertDialogHeader>
            <AlertDialogTitle>Change Workspace?</AlertDialogTitle>
            <AlertDialogDescription>
              Move this task from &quot;{currentWorkspace?.name ?? 'No Workspace'}&quot; to &quot;{pendingWorkspace?.name}&quot;?
            </AlertDialogDescription>
          </AlertDialogHeader>
          <AlertDialogFooter>
            <AlertDialogCancel>Cancel</AlertDialogCancel>
            <AlertDialogAction onClick={handleConfirm}>Confirm</AlertDialogAction>
          </AlertDialogFooter>
        </AlertDialogContent>
      </AlertDialog>
    </>
  );
}
```

### Step 2: Update task-detail-drawer.tsx header

Add WorkspaceChangeDropdown import and place next to taskCode:

```tsx
import { WorkspaceChangeDropdown } from './workspace-change-dropdown';
```

Replace taskCode line:
```tsx
// Before:
<p className="text-sm font-medium text-muted-foreground">{task.taskCode}</p>

// After:
<div className="flex items-center gap-2">
  <p className="text-sm font-medium text-muted-foreground">{task.taskCode}</p>
  <WorkspaceChangeDropdown currentWorkspaceId={task.workspaceId} />
</div>
```

### Step 3: Refactor quick-status-actions.tsx

Merge 2 buttons into 1 dropdown button:

```tsx
'use client';

import type { MockTask, DesignTaskStatusType } from '@/mocks/types';
import { DesignTaskStatus } from '@/mocks/types';
import { Button } from '@/components/ui/button';
import {
  DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { Badge } from '@/components/ui/badge';
import { Play, Send, Check, RotateCcw, ChevronDown, Circle } from 'lucide-react';

type TaskStatus = DesignTaskStatusType;

const ALL_STATUSES: { status: TaskStatus; label: string; icon: React.ReactNode }[] = [
  { status: DesignTaskStatus.TODO, label: 'To Do', icon: <Circle className="h-3 w-3" /> },
  { status: DesignTaskStatus.IN_PROGRESS, label: 'In Progress', icon: <Play className="h-3 w-3" /> },
  { status: DesignTaskStatus.REVIEW, label: 'Review', icon: <Send className="h-3 w-3" /> },
  { status: DesignTaskStatus.REVISION, label: 'Revision', icon: <RotateCcw className="h-3 w-3" /> },
  { status: DesignTaskStatus.APPROVED, label: 'Approved', icon: <Check className="h-3 w-3" /> },
];

// Get display label for current primary action
function getActionLabel(status: TaskStatus): string {
  switch (status) {
    case DesignTaskStatus.TODO: return 'Start';
    case DesignTaskStatus.IN_PROGRESS: return 'Submit';
    case DesignTaskStatus.REVIEW: return 'Approve';
    case DesignTaskStatus.REVISION: return 'Restart';
    default: return '';
  }
}

export interface QuickStatusActionsProps {
  task: MockTask;
  onStatusChange?: (newStatus: TaskStatus) => void;
}

export function QuickStatusActions({ task, onStatusChange }: QuickStatusActionsProps) {
  const handleClick = (nextStatus: TaskStatus) => {
    console.log(`Status: ${task.status} -> ${nextStatus}`);
    onStatusChange?.(nextStatus);
  };

  // APPROVED: show static badge, no dropdown
  if (task.status === DesignTaskStatus.APPROVED) {
    return <Badge variant="outline" className="text-xs text-green-600">Approved</Badge>;
  }

  const label = getActionLabel(task.status);

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button size="sm" className="h-7 text-xs gap-1">
          {label}
          <ChevronDown className="h-3 w-3" />
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end">
        {ALL_STATUSES.filter((s) => s.status !== task.status).map((item) => (
          <DropdownMenuItem
            key={item.status}
            onClick={() => handleClick(item.status)}
            className="gap-2 text-xs"
          >
            {item.icon}
            {item.label}
          </DropdownMenuItem>
        ))}
      </DropdownMenuContent>
    </DropdownMenu>
  );
}
```

### Step 4: Update drawer/index.ts

Add export:
```ts
export { WorkspaceChangeDropdown, type WorkspaceChangeDropdownProps } from './workspace-change-dropdown';
```

### Step 5: Compile check
```bash
cd apps/web && pnpm tsc --noEmit
```

## Todo List

- [ ] Create workspace-change-dropdown.tsx with DropdownMenu + AlertDialog
- [ ] Update task-detail-drawer.tsx header to include WorkspaceChangeDropdown
- [ ] Refactor quick-status-actions.tsx: merge 2 buttons into 1 dropdown button
- [ ] Add WorkspaceChangeDropdown export to drawer/index.ts
- [ ] Run tsc --noEmit to verify no compile errors
- [ ] Visual verification: drawer header shows workspace dropdown, single status button

## Success Criteria

- `pnpm tsc --noEmit` passes
- Workspace dropdown shows next to taskCode in drawer header
- Selecting different workspace triggers confirm dialog
- QuickStatusActions renders as single button with dropdown
- APPROVED status shows static badge instead of dropdown
- workspace-change-dropdown.tsx under 100 LOC

## Risk Assessment

- **Low**: AlertDialog component already exists and is well-tested
- **Low**: DropdownMenu pattern already used in quick-status-actions.tsx
- **Medium**: WorkspaceChangeDropdown needs proper controlled state for AlertDialog -- reference code provided uses `pendingWorkspaceId` pattern to avoid stale closures

## Security Considerations

- Workspace names rendered via React JSX (auto-escaped, no XSS risk)
- No user text input in either component
- Demo-only changes (console.log), no API calls

## Next Steps

- Run code-reviewer agent after implementation
- Run tester agent to verify compile + visual check
- Update docs if needed (codebase-summary.md for drawer section)
