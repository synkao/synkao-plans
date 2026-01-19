# Phase 4: Update BacklogPage

**Priority:** High
**Status:** ✅ Complete (2026-01-16)
**Depends on:** Phase 3

---

## Context

- [page.tsx](../../apps/web/src/app/(main)/design/backlog/page.tsx) - Current implementation
- [use-tasks.ts](../../apps/web/src/features/design/hooks/use-tasks.ts) - New hooks

## Overview

Replace mock imports and raw fetch with React Query hooks. Keep client-side filters for snappy UX.

## Current Issues (to fix)

1. Line 13: Direct import of `mockTasks` - replace with `useBacklogTasks()`
2. Lines 72-90, 93-112, 115-137: Raw fetch calls - replace with mutation hooks
3. No loading/error states

## Implementation

### File: `apps/web/src/app/(main)/design/backlog/page.tsx` (UPDATE)

```typescript
'use client';

import { useState, useMemo } from 'react';
import { toast } from 'sonner';
import { PageHeader } from '@/components/layout';
import {
  BacklogStatsBar,
  BacklogFilterBar,
  BacklogTable,
  BacklogActionBar,
  TaskDetailDrawer,
  useBacklogTasks,
  useBulkAssignWorkspace,
  useBulkAssignDesigner,
  useBulkToggleUrgent,
} from '@/features/design';
import { mockWorkspaces, getDesigners } from '@/mocks/data';
import type { MockTask, TaskSourceType } from '@/mocks/types';

export default function BacklogPage() {
  // React Query - server data
  const { data: taskResponse, isLoading, error } = useBacklogTasks();
  const bulkAssignWorkspace = useBulkAssignWorkspace();
  const bulkAssignDesigner = useBulkAssignDesigner();
  const bulkToggleUrgent = useBulkToggleUrgent();

  // Client-side filter state (for snappy UX)
  const [search, setSearch] = useState('');
  const [urgentOnly, setUrgentOnly] = useState(false);
  const [source, setSource] = useState<TaskSourceType | 'ALL'>('ALL');

  // Selection state
  const [selectedIds, setSelectedIds] = useState<Set<string>>(new Set());

  // Drawer state
  const [selectedTask, setSelectedTask] = useState<MockTask | null>(null);
  const [drawerOpen, setDrawerOpen] = useState(false);

  // Server-fetched backlog tasks
  const backlogTasks = taskResponse?.data ?? [];

  // Apply client-side filters
  const filteredTasks = useMemo(() => {
    return backlogTasks.filter((task) => {
      // Search filter
      if (search) {
        const searchLower = search.toLowerCase();
        const matchesSearch =
          task.taskCode.toLowerCase().includes(searchLower) ||
          task.productName.toLowerCase().includes(searchLower) ||
          task.customerName.toLowerCase().includes(searchLower);
        if (!matchesSearch) return false;
      }

      // Urgent only filter
      if (urgentOnly && !task.isUrgent) return false;

      // Source filter
      if (source !== 'ALL' && task.source !== source) return false;

      return true;
    });
  }, [backlogTasks, search, urgentOnly, source]);

  const handleTaskClick = (task: MockTask) => {
    setSelectedTask(task);
    setDrawerOpen(true);
  };

  const handleSelectionChange = (ids: Set<string>) => {
    setSelectedIds(ids);
  };

  const handleClearSelection = () => {
    setSelectedIds(new Set());
  };

  const handleAddToWorkspace = async (workspaceId: string) => {
    try {
      await bulkAssignWorkspace.mutateAsync({
        taskIds: Array.from(selectedIds),
        workspaceId,
      });
      const workspace = mockWorkspaces.find((w) => w.id === workspaceId);
      toast.success(`${selectedIds.size} tasks added to ${workspace?.name || 'workspace'}`);
      setSelectedIds(new Set());
    } catch (error) {
      console.error('Assign workspace error:', error);
      toast.error('Failed to assign tasks to workspace');
    }
  };

  const handleAssignDesigner = async (designerId: string) => {
    try {
      await bulkAssignDesigner.mutateAsync({
        taskIds: Array.from(selectedIds),
        assigneeId: designerId,
      });
      const designer = getDesigners().find((d) => d.id === designerId);
      toast.success(`${selectedIds.size} tasks assigned to ${designer?.name || 'designer'}`);
      setSelectedIds(new Set());
    } catch (error) {
      console.error('Assign designer error:', error);
      toast.error('Failed to assign tasks to designer');
    }
  };

  const handleToggleUrgent = async (isUrgent: boolean) => {
    try {
      await bulkToggleUrgent.mutateAsync({
        taskIds: Array.from(selectedIds),
        isUrgent,
      });
      toast.success(
        isUrgent
          ? `${selectedIds.size} tasks marked as urgent`
          : `${selectedIds.size} tasks cleared urgent status`
      );
      setSelectedIds(new Set());
    } catch (error) {
      console.error('Toggle urgent error:', error);
      toast.error('Failed to update urgent status');
    }
  };

  // Loading state
  if (isLoading) {
    return (
      <>
        <PageHeader
          title="Design Backlog"
          description="Tasks waiting to be assigned to designers"
          breadcrumbs={[
            { label: 'Dashboard', href: '/' },
            { label: 'Design', href: '/design' },
            { label: 'Backlog' },
          ]}
        />
        <div className="flex items-center justify-center py-12">
          <div className="text-muted-foreground">Loading tasks...</div>
        </div>
      </>
    );
  }

  // Error state
  if (error) {
    return (
      <>
        <PageHeader
          title="Design Backlog"
          description="Tasks waiting to be assigned to designers"
          breadcrumbs={[
            { label: 'Dashboard', href: '/' },
            { label: 'Design', href: '/design' },
            { label: 'Backlog' },
          ]}
        />
        <div className="flex items-center justify-center py-12">
          <div className="text-destructive">Failed to load tasks: {error.message}</div>
        </div>
      </>
    );
  }

  return (
    <>
      <PageHeader
        title="Design Backlog"
        description="Tasks waiting to be assigned to designers"
        breadcrumbs={[
          { label: 'Dashboard', href: '/' },
          { label: 'Design', href: '/design' },
          { label: 'Backlog' },
        ]}
      />

      <div className="space-y-4">
        {/* Stats Bar */}
        <BacklogStatsBar tasks={backlogTasks} />

        {/* Filter Bar */}
        <BacklogFilterBar
          search={search}
          onSearchChange={setSearch}
          urgentOnly={urgentOnly}
          onUrgentOnlyChange={setUrgentOnly}
          source={source}
          onSourceChange={setSource}
        />

        {/* Tasks Table */}
        <BacklogTable
          tasks={filteredTasks}
          onTaskClick={handleTaskClick}
          onSelectionChange={handleSelectionChange}
        />

        {/* Action Bar */}
        {selectedIds.size > 0 && (
          <BacklogActionBar
            selectedCount={selectedIds.size}
            workspaces={mockWorkspaces}
            designers={getDesigners()}
            onAddToWorkspace={handleAddToWorkspace}
            onAssignDesigner={handleAssignDesigner}
            onToggleUrgent={handleToggleUrgent}
            onClearSelection={handleClearSelection}
          />
        )}
      </div>

      {/* Task Detail Drawer */}
      <TaskDetailDrawer
        task={selectedTask}
        open={drawerOpen}
        onOpenChange={setDrawerOpen}
      />
    </>
  );
}
```

## Key Changes

1. **Imports**: Remove `mockTasks`, add hooks from `@/features/design`
2. **Data fetching**: `useBacklogTasks()` replaces direct mock import
3. **Mutations**: Three mutation hooks replace raw fetch calls
4. **States**: Added loading/error handling
5. **Preserved**: Client-side filters remain for snappy UX

## Todo

- [ ] Update imports
- [ ] Replace mockTasks with useBacklogTasks()
- [ ] Replace raw fetch with mutation hooks
- [ ] Add loading/error states
- [ ] Run compile check
- [ ] Test page functionality

## Success Criteria

- Page loads tasks from API via React Query
- Bulk actions use mutations with auto-refetch
- Loading spinner shown during fetch
- Error message shown on failure
- Client-side filters remain instant
