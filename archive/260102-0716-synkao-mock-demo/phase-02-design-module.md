# Phase 2: Design Module (Kanban, Backlog, Task Drawer)

## Context Links
- **Parent Plan:** [plan.md](./plan.md)
- **Dependencies:** [Phase 1](./phase-01-foundation-layout.md)
- **Design System:** [synkao-design-system.md](../../docs/design/synkao-design-system.md)
- **UI Spec:** Section 7 - Design Module

---

## Overview

| Property | Value |
|----------|-------|
| Priority | P1 |
| Status | pending |
| Effort | 3.5 days |
| Description | Build Design module - Kanban board với dnd-kit drag-drop, Backlog table, Task detail drawer |

---

## Key Insights (from Research)

1. **dnd-kit Library** - `@dnd-kit/core` + `@dnd-kit/sortable` cho drag-drop
2. **Kanban NOT in shadcn** - Build custom với Card components + dnd-kit
3. **Task Data** - 45 tasks trong `mockTasks` (15 TODO, 10 IN_PROGRESS, 8 REVIEW, 5 REVISION, 7 DONE)
4. **Sheet Component** - Use for Task Detail Drawer (slide from right)
5. **Column Colors** - TODO (blue), IN_PROGRESS (amber), REVIEW (violet), REVISION (red), DONE (green)
6. **Local State Only** - Drag-drop changes local state, không persist

---

## Requirements

### Functional
- F1: Kanban board with 5 columns (TODO, IN_PROGRESS, REVIEW, REVISION, DONE)
- F2: Task cards display order, product name, priority, assignee
- F3: Click task card → open Task Detail Drawer
- F4: **Drag-drop** task between columns (dnd-kit)
- F5: Status dropdown as fallback (accessibility)
- F6: Backlog page with data table (filterable, sortable)
- F7: Workspace list page (cards for each workspace)
- F8: Filter tasks by assignee

### Non-Functional
- NF1: Kanban horizontal scrollable on smaller screens
- NF2: Column has subtle background color
- NF3: Card hover shows elevation

---

## Architecture

### Component Tree - Kanban
```
WorkspacePage
├── PageHeader (title, filter)
└── KanbanBoard
    ├── KanbanColumn (TODO)
    │   ├── ColumnHeader (name, count)
    │   └── TaskCard[]
    ├── KanbanColumn (IN_PROGRESS)
    ├── KanbanColumn (REVIEW)
    ├── KanbanColumn (REVISION)
    └── KanbanColumn (DONE)

TaskDetailDrawer (Sheet)
├── TaskHeader (order, item, status badge)
├── TaskInfo (assignee, deadline, workspace)
├── DesignNotes
├── DesignVersions (list)
├── Timeline
└── Actions (status dropdown)
```

### Component Tree - Backlog
```
BacklogPage
├── PageHeader (title, create button)
├── FilterBar (search, priority, source)
├── StatsBar (counts by priority)
├── BacklogTable (data-table)
│   └── columns: checkbox, order, item, customer, priority, source, age
└── BulkActionBar (when selected)
```

---

## Related Code Files

### Install (FIRST)
```bash
cd apps/web
pnpm add @dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities
```

### Create - Pages
| File Path | Description |
|-----------|-------------|
| `apps/web/src/app/(main)/design/backlog/page.tsx` | Backlog page - TODO tasks table |
| `apps/web/src/app/(main)/design/workspace/page.tsx` | Workspace list - cards grid |
| `apps/web/src/app/(main)/design/workspace/[id]/page.tsx` | Kanban board page |

### Create - Feature Components
| File Path | Description |
|-----------|-------------|
| `apps/web/src/features/design/components/kanban/kanban-board.tsx` | Main kanban với DndContext |
| `apps/web/src/features/design/components/kanban/kanban-column.tsx` | Droppable column container |
| `apps/web/src/features/design/components/kanban/sortable-task-card.tsx` | Draggable task wrapper |
| `apps/web/src/features/design/components/kanban/task-card.tsx` | Task card UI (no drag logic) |
| `apps/web/src/features/design/components/kanban/drag-overlay-card.tsx` | Card preview khi dragging |
| `apps/web/src/features/design/components/kanban/index.ts` | Kanban barrel exports |
| `apps/web/src/features/design/components/drawer/task-detail-drawer.tsx` | Sheet slide from right |
| `apps/web/src/features/design/components/drawer/task-info-section.tsx` | Assignee, deadline, workspace |
| `apps/web/src/features/design/components/drawer/design-notes-section.tsx` | Design notes display |
| `apps/web/src/features/design/components/drawer/design-versions-section.tsx` | Version list |
| `apps/web/src/features/design/components/drawer/timeline-section.tsx` | Activity timeline |
| `apps/web/src/features/design/components/drawer/index.ts` | Drawer barrel exports |
| `apps/web/src/features/design/components/backlog/backlog-table.tsx` | Data table component |
| `apps/web/src/features/design/components/backlog/backlog-columns.tsx` | TanStack column defs |
| `apps/web/src/features/design/components/backlog/backlog-filter-bar.tsx` | Search, priority filter |
| `apps/web/src/features/design/components/backlog/backlog-stats-bar.tsx` | Priority counts |
| `apps/web/src/features/design/components/backlog/index.ts` | Backlog barrel exports |
| `apps/web/src/features/design/components/workspace-card.tsx` | Workspace card with stats |
| `apps/web/src/features/design/components/priority-badge.tsx` | Priority với icon + color |
| `apps/web/src/features/design/components/status-badge.tsx` | Status với color |
| `apps/web/src/features/design/components/source-badge.tsx` | Task source (NEW, REDO, COMPLAINT) |
| `apps/web/src/features/design/components/status-dropdown.tsx` | Status change dropdown |
| `apps/web/src/features/design/components/designer-filter.tsx` | Filter by assignee dropdown |
| `apps/web/src/features/design/components/index.ts` | Feature barrel exports |

### Create - Hooks & Utils
| File Path | Description |
|-----------|-------------|
| `apps/web/src/features/design/hooks/use-kanban-state.ts` | Local state for drag-drop |
| `apps/web/src/features/design/hooks/use-tasks-by-status.ts` | Group tasks by column |
| `apps/web/src/features/design/hooks/index.ts` | Hooks barrel exports |
| `apps/web/src/features/design/lib/constants.ts` | Column configs, colors, priority configs |
| `apps/web/src/features/design/lib/utils.ts` | Helper functions |
| `apps/web/src/features/design/index.ts` | Feature barrel exports |

### Create - Mock Data (Enhanced)
| File Path | Description |
|-----------|-------------|
| `apps/web/src/mocks/data/design-versions.data.ts` | 20+ design versions cho tasks |
| `apps/web/src/mocks/data/timeline-events.data.ts` | Activity events cho drawer |
| `apps/web/src/mocks/data/workspaces.data.ts` | Workspace data nếu chưa có |

### Modify
| File Path | Change Description |
|-----------|-------------------|
| `apps/web/src/mocks/data/tasks.data.ts` | Add workspaceId field, customer name join |
| `apps/web/src/mocks/types.ts` | Add MockTimelineEvent interface |
| `apps/web/src/mocks/index.ts` | Export new mock data |
| `apps/web/src/lib/navigation.ts` | Add Design routes with children |

### Delete
| File Path | Reason |
|-----------|--------|
| `apps/web/src/features/design/.gitkeep` | Folder sẽ có files |

---

## Implementation Steps

### 0. Install dnd-kit (10 min)
```bash
cd apps/web
pnpm add @dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities
```

### 1. Create Design Constants (20 min)
```typescript
// apps/web/src/features/design/lib/constants.ts
export const KANBAN_COLUMNS = [
  { id: 'TODO', name: 'Cần làm', color: 'blue', bgClass: 'bg-blue-50/50' },
  { id: 'IN_PROGRESS', name: 'Đang làm', color: 'amber', bgClass: 'bg-amber-50/50' },
  { id: 'REVIEW', name: 'Review', color: 'violet', bgClass: 'bg-violet-50/50' },
  { id: 'REVISION', name: 'Cần sửa', color: 'red', bgClass: 'bg-red-50/50' },
  { id: 'DONE', name: 'Hoàn thành', color: 'emerald', bgClass: 'bg-emerald-50/50' },
];

export const PRIORITY_CONFIG = {
  URGENT: { label: 'Gấp', color: 'red', icon: Flame },
  HIGH: { label: 'Cao', color: 'amber', icon: ArrowUp },
  NORMAL: { label: 'Bình thường', color: 'blue', icon: Minus },
  LOW: { label: 'Thấp', color: 'gray', icon: ArrowDown },
};
```

### 2. Create Badge Components (30 min)
- `priority-badge.tsx` - Display priority with color + icon
- `status-badge.tsx` - Display task status with color

### 3. Create TaskCard Component (1 hour)
```tsx
// Task card in kanban column
<Card className="p-3 hover:shadow-lg transition cursor-pointer">
  <div className="flex gap-3">
    <Thumbnail />
    <div>
      <p className="text-xs text-muted">{orderNumber}</p>
      <p className="font-medium">{productName}</p>
      <p className="text-sm text-muted">{customerName}</p>
    </div>
  </div>
  <div className="flex justify-between mt-2">
    <PriorityBadge priority={priority} />
    <Avatar assignee={assignee} />
  </div>
</Card>
```

### 4. Create KanbanColumn Component (45 min)
```tsx
// Droppable column với useDroppable
import { useDroppable } from '@dnd-kit/core';
import { SortableContext, verticalListSortingStrategy } from '@dnd-kit/sortable';

export function KanbanColumn({ column, tasks }) {
  const { setNodeRef, isOver } = useDroppable({ id: column.id });

  return (
    <div
      ref={setNodeRef}
      className={cn(
        "min-w-[280px] rounded-lg p-2 transition-colors",
        column.bgClass,
        isOver && "ring-2 ring-primary/50"
      )}
    >
      <ColumnHeader name={column.name} count={tasks.length} />
      <SortableContext items={tasks.map(t => t.id)} strategy={verticalListSortingStrategy}>
        <div className="space-y-2 mt-3">
          {tasks.map(task => <SortableTaskCard key={task.id} task={task} />)}
        </div>
      </SortableContext>
    </div>
  );
}
```

### 5. Create KanbanBoard Component with DndContext (1 hour)
```tsx
import { DndContext, DragOverlay, closestCorners } from '@dnd-kit/core';

export function KanbanBoard({ workspaceId }) {
  const [tasks, setTasks] = useKanbanState(workspaceId);
  const [activeTask, setActiveTask] = useState<MockTask | null>(null);

  const handleDragStart = (event) => {
    setActiveTask(findTask(event.active.id));
  };

  const handleDragEnd = (event) => {
    const { active, over } = event;
    if (!over) return;

    const activeStatus = findTask(active.id)?.status;
    const overStatus = over.id; // Column id = status

    if (activeStatus !== overStatus) {
      setTasks(prev => prev.map(t =>
        t.id === active.id ? { ...t, status: overStatus } : t
      ));
    }
    setActiveTask(null);
  };

  return (
    <DndContext
      collisionDetection={closestCorners}
      onDragStart={handleDragStart}
      onDragEnd={handleDragEnd}
    >
      <div className="flex gap-4 overflow-x-auto pb-4">
        {KANBAN_COLUMNS.map(col => (
          <KanbanColumn
            key={col.id}
            column={col}
            tasks={tasks.filter(t => t.status === col.id)}
          />
        ))}
      </div>
      <DragOverlay>
        {activeTask && <DragOverlayCard task={activeTask} />}
      </DragOverlay>
    </DndContext>
  );
}
```

### 6. Create useKanbanState Hook (30 min)
```typescript
// Local state for drag-drop, persists in session only
export function useKanbanState(workspaceId: string) {
  const [tasks, setTasks] = useState<MockTask[]>(() => {
    // Filter tasks for this workspace
    return mockTasks.filter(t => t.workspaceId === workspaceId);
  });

  return [tasks, setTasks] as const;
}
```

### 7. Create SortableTaskCard (30 min)
```tsx
import { useSortable } from '@dnd-kit/sortable';
import { CSS } from '@dnd-kit/utilities';

export function SortableTaskCard({ task, onClick }) {
  const { attributes, listeners, setNodeRef, transform, transition, isDragging } = useSortable({
    id: task.id,
  });

  const style = {
    transform: CSS.Transform.toString(transform),
    transition,
    opacity: isDragging ? 0.5 : 1,
  };

  return (
    <div ref={setNodeRef} style={style} {...attributes} {...listeners}>
      <TaskCard task={task} onClick={onClick} />
    </div>
  );
}
```

### 8. Create useTasksByStatus Hook (20 min)
```typescript
// Group mock tasks by status (readonly, for initial render)
export function useTasksByStatus() {
  const grouped = useMemo(() => {
    return mockTasks.reduce((acc, task) => {
      if (!acc[task.status]) acc[task.status] = [];
      acc[task.status].push(task);
      return acc;
    }, {} as Record<string, MockTask[]>);
  }, []);
  return grouped;
}
```

### 9. Create TaskDetailDrawer (1.5 hours)
Using shadcn Sheet:
```tsx
<Sheet>
  <SheetContent className="w-[480px]">
    <SheetHeader>
      <SheetTitle>{orderNumber} / {itemCode}</SheetTitle>
    </SheetHeader>
    <div className="space-y-6">
      <TaskInfo task={task} />
      <Separator />
      <DesignNotes notes={notes} />
      <Separator />
      <DesignVersionsList versions={versions} />
      <Separator />
      <TaskTimeline events={events} />
    </div>
    <SheetFooter>
      <StatusDropdown currentStatus={status} />
    </SheetFooter>
  </SheetContent>
</Sheet>
```

### 10. Create StatusDropdown (30 min)
Dropdown as fallback for accessibility:
```tsx
<DropdownMenu>
  <DropdownMenuTrigger asChild>
    <Button variant="outline">Đổi trạng thái</Button>
  </DropdownMenuTrigger>
  <DropdownMenuContent>
    {KANBAN_COLUMNS.map(col => (
      <DropdownMenuItem key={col.id}>
        {col.name}
      </DropdownMenuItem>
    ))}
  </DropdownMenuContent>
</DropdownMenu>
```

### 11. Create DesignerFilter Component (30 min)
Filter tasks by assignee:
```tsx
// apps/web/src/features/design/components/designer-filter.tsx
export function DesignerFilter({ designers, value, onChange }) {
  return (
    <Select value={value} onValueChange={onChange}>
      <SelectTrigger className="w-[180px]">
        <SelectValue placeholder="Tất cả designers" />
      </SelectTrigger>
      <SelectContent>
        <SelectItem value="all">Tất cả designers</SelectItem>
        {designers.map(d => (
          <SelectItem key={d.id} value={d.id}>
            <div className="flex items-center gap-2">
              <Avatar className="h-5 w-5">{d.name[0]}</Avatar>
              {d.name}
            </div>
          </SelectItem>
        ))}
      </SelectContent>
    </Select>
  );
}
```

### 12. Create Workspace Page (Kanban) (45 min)
```tsx
// apps/web/src/app/(main)/design/workspace/[id]/page.tsx
export function generateStaticParams() {
  return mockWorkspaces.map(w => ({ id: w.id }));
}

export default function WorkspacePage({ params }) {
  const workspace = findWorkspace(params.id);
  const [selectedDesigner, setSelectedDesigner] = useState('all');
  const designers = mockUsers.filter(u => u.role === 'DESIGNER');

  return (
    <>
      <PageHeader
        title={`Workspace: ${workspace.name}`}
        actions={
          <DesignerFilter
            designers={designers}
            value={selectedDesigner}
            onChange={setSelectedDesigner}
          />
        }
      />
      <KanbanBoard workspaceId={params.id} filterDesigner={selectedDesigner} />
      <TaskDetailDrawer />
    </>
  );
}
```

### 13. Create Backlog Table (1.5 hours)
Using shadcn data-table:
- Define columns: checkbox, order, item, customer, priority, source, age
- Add filter bar: search, priority, source
- Add stats bar: counts by priority
- Add bulk action bar (visible when selected)

### 14. Create Backlog Page (30 min)
```tsx
// apps/web/src/app/(main)/design/backlog/page.tsx
export default function BacklogPage() {
  // Filter TODO tasks (not yet assigned to workspace)
  const backlogTasks = mockTasks.filter(t => t.status === 'TODO');

  return (
    <>
      <PageHeader title="Design Backlog" />
      <StatsBar tasks={backlogTasks} />
      <BacklogTable tasks={backlogTasks} />
    </>
  );
}
```

### 15. Create Workspace List Page (45 min)
```tsx
// apps/web/src/app/(main)/design/workspace/page.tsx
export default function WorkspaceListPage() {
  return (
    <>
      <PageHeader title="Workspaces" />
      <div className="grid grid-cols-3 gap-4">
        {mockWorkspaces.map(w => (
          <WorkspaceCard key={w.id} workspace={w} />
        ))}
      </div>
    </>
  );
}
```

---

## Todo List

### Setup
- [ ] Install `@dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities`
- [ ] Create `apps/web/src/features/design/lib/constants.ts`
- [ ] Create `apps/web/src/features/design/lib/utils.ts`

### Kanban Components
- [ ] Create `apps/web/src/features/design/components/kanban/task-card.tsx`
- [ ] Create `apps/web/src/features/design/components/kanban/sortable-task-card.tsx`
- [ ] Create `apps/web/src/features/design/components/kanban/drag-overlay-card.tsx`
- [ ] Create `apps/web/src/features/design/components/kanban/kanban-column.tsx`
- [ ] Create `apps/web/src/features/design/components/kanban/kanban-board.tsx`
- [ ] Create `apps/web/src/features/design/components/kanban/index.ts`

### Drawer Components
- [ ] Create `apps/web/src/features/design/components/drawer/task-detail-drawer.tsx`
- [ ] Create `apps/web/src/features/design/components/drawer/task-info-section.tsx`
- [ ] Create `apps/web/src/features/design/components/drawer/design-notes-section.tsx`
- [ ] Create `apps/web/src/features/design/components/drawer/design-versions-section.tsx`
- [ ] Create `apps/web/src/features/design/components/drawer/timeline-section.tsx`
- [ ] Create `apps/web/src/features/design/components/drawer/index.ts`

### Backlog Components
- [ ] Create `apps/web/src/features/design/components/backlog/backlog-columns.tsx`
- [ ] Create `apps/web/src/features/design/components/backlog/backlog-table.tsx`
- [ ] Create `apps/web/src/features/design/components/backlog/backlog-filter-bar.tsx`
- [ ] Create `apps/web/src/features/design/components/backlog/backlog-stats-bar.tsx`
- [ ] Create `apps/web/src/features/design/components/backlog/index.ts`

### Shared Components
- [ ] Create `apps/web/src/features/design/components/priority-badge.tsx`
- [ ] Create `apps/web/src/features/design/components/status-badge.tsx`
- [ ] Create `apps/web/src/features/design/components/source-badge.tsx`
- [ ] Create `apps/web/src/features/design/components/status-dropdown.tsx`
- [ ] Create `apps/web/src/features/design/components/designer-filter.tsx`
- [ ] Create `apps/web/src/features/design/components/workspace-card.tsx`
- [ ] Create `apps/web/src/features/design/components/index.ts`

### Hooks
- [ ] Create `apps/web/src/features/design/hooks/use-kanban-state.ts`
- [ ] Create `apps/web/src/features/design/hooks/use-tasks-by-status.ts`
- [ ] Create `apps/web/src/features/design/hooks/index.ts`

### Mock Data
- [ ] Create `apps/web/src/mocks/data/design-versions.data.ts`
- [ ] Create `apps/web/src/mocks/data/timeline-events.data.ts`
- [ ] Update `apps/web/src/mocks/data/tasks.data.ts` - add workspaceId
- [ ] Update `apps/web/src/mocks/types.ts` - add MockTimelineEvent

### Pages
- [ ] Create `apps/web/src/app/(main)/design/backlog/page.tsx`
- [ ] Create `apps/web/src/app/(main)/design/workspace/page.tsx`
- [ ] Create `apps/web/src/app/(main)/design/workspace/[id]/page.tsx`

### Integration
- [ ] Update `apps/web/src/lib/navigation.ts` - Design routes
- [ ] Create `apps/web/src/features/design/index.ts` - barrel export

### Testing
- [ ] Test drag-drop between columns
- [ ] Test drag overlay shows card preview
- [ ] Test task drawer opens on card click
- [ ] Test status dropdown as fallback
- [ ] Test designer filter filters kanban tasks
- [ ] Test backlog table filters work

---

## Success Criteria

### Kanban Board
- [ ] 5 columns render with correct colors (blue, amber, violet, red, emerald)
- [ ] Tasks grouped by status correctly
- [ ] Task cards show: order number, product name, priority, assignee
- [ ] **Drag-drop** moves task between columns (local state)
- [ ] Drag overlay shows card preview while dragging
- [ ] Column highlights when dragging over (ring effect)
- [ ] Designer filter filters tasks by assignee

### Task Drawer
- [ ] Click task card opens Sheet from right
- [ ] Drawer shows task info, design notes, versions, timeline
- [ ] Status dropdown changes task status (alternative to drag)

### Backlog
- [ ] Data table renders with sortable columns
- [ ] Filter by priority, source works
- [ ] Stats bar shows priority counts
- [ ] Workspace list shows cards with task counts

---

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| dnd-kit learning curve | Medium | Follow official examples, simple setup first |
| Drag conflicts with click | Medium | Use `useSortable` delay option, separate drag handles if needed |
| Complex drawer layout | Medium | Start simple, iterate |
| Data table performance | Low | Pagination for large lists |
| Kanban scroll on mobile | Low | Desktop only, skip mobile |
| State management complexity | Low | Keep local useState, no global store for demo |

---

## Next Steps

After completion, proceed to [Phase 3: Orders Module](./phase-03-orders-module.md)
