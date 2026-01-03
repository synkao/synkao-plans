# Phase 01: DnD Visual Feedback Implementation

**Effort:** 2h
**Status:** pending

## Overview

Implement visual feedback cho Kanban drag-drop: drop indicator, trailing placeholder, reorder trong column.

---

## Task 1: Tao drop-placeholder.tsx

**File:** `apps/web/src/features/design/components/kanban/drop-placeholder.tsx`

### Implementation

```tsx
import { cn } from '@/lib/utils';

interface DropPlaceholderProps {
  isActive?: boolean;
}

/**
 * Placeholder hien thi khi drag task - dat cuoi column
 */
export function DropPlaceholder({ isActive = false }: DropPlaceholderProps) {
  return (
    <div
      className={cn(
        'h-20 rounded-lg border-2 border-dashed border-gray-300 bg-gray-100/30',
        'flex items-center justify-center transition-all',
        isActive && 'border-primary bg-primary/10'
      )}
    >
      <span className="text-sm text-gray-400">
        {isActive ? 'Drop here' : 'Drag here'}
      </span>
    </div>
  );
}
```

### Acceptance
- [ ] Component render dung
- [ ] isActive thay doi style

---

## Task 2: Update kanban-board.tsx

**File:** `apps/web/src/features/design/components/kanban/kanban-board.tsx`

### 2.1 Them imports

```typescript
// Them vao dong 13
import { arrayMove } from '@dnd-kit/sortable';
import type { DragOverEvent } from '@dnd-kit/core';
```

### 2.2 Them handleDragOver function

Them sau `handleDragStart`:

```typescript
const handleDragOver = (event: DragOverEvent) => {
  const { active, over } = event;
  if (!over) return;

  const activeId = active.id as string;
  const overId = over.id as string;

  const activeTask = tasks.find((t) => t.id === activeId);
  const overTask = tasks.find((t) => t.id === overId);

  if (!activeTask) return;

  // Case 1: Hover len task khac
  if (overTask) {
    if (activeTask.status !== overTask.status) {
      // Move sang column moi + insert truoc overTask
      const updatedTasks = tasks.filter((t) => t.id !== activeId);
      const overIndex = updatedTasks.findIndex((t) => t.id === overId);
      const newTask = { ...activeTask, status: overTask.status };
      updatedTasks.splice(overIndex, 0, newTask);
      onTasksChange(updatedTasks);
    } else {
      // Reorder trong cung column
      const oldIndex = tasks.findIndex((t) => t.id === activeId);
      const newIndex = tasks.findIndex((t) => t.id === overId);
      if (oldIndex !== newIndex) {
        onTasksChange(arrayMove(tasks, oldIndex, newIndex));
      }
    }
    return;
  }

  // Case 2: Hover len column (empty area)
  if (VALID_STATUSES.includes(overId as MockTask['status'])) {
    if (activeTask.status !== overId) {
      const updatedTasks = tasks.map((t) =>
        t.id === activeId ? { ...t, status: overId as MockTask['status'] } : t
      );
      onTasksChange(updatedTasks);
    }
    return;
  }

  // Case 3: Trailing placeholder (overId = `${columnId}-trailing`)
  if (overId.endsWith('-trailing')) {
    const columnId = overId.replace('-trailing', '') as MockTask['status'];
    if (activeTask.status !== columnId) {
      const updatedTasks = tasks.map((t) =>
        t.id === activeId ? { ...t, status: columnId } : t
      );
      onTasksChange(updatedTasks);
    }
  }
};
```

### 2.3 Update DndContext

```tsx
<DndContext
  sensors={sensors}
  collisionDetection={closestCorners}
  onDragStart={handleDragStart}
  onDragOver={handleDragOver}  // <-- Them
  onDragEnd={handleDragEnd}
>
```

### 2.4 Pass isDragging prop to KanbanColumn

```tsx
{KANBAN_COLUMNS.map((column) => (
  <KanbanColumn
    key={column.id}
    column={column}
    tasks={groupedTasks[column.id] ?? []}
    onTaskClick={onTaskClick}
    isDragging={!!activeTask}  // <-- Them
  />
))}
```

### 2.5 Simplify handleDragEnd

`handleDragEnd` chi can reset activeTask, vi logic move da xu ly trong `handleDragOver`:

```typescript
const handleDragEnd = () => {
  setActiveTask(null);
};
```

### Acceptance
- [ ] handleDragOver hoat dong
- [ ] Reorder trong column OK
- [ ] Cross-column move OK
- [ ] isDragging prop duoc pass

---

## Task 3: Update kanban-column.tsx

**File:** `apps/web/src/features/design/components/kanban/kanban-column.tsx`

### 3.1 Update interface

```typescript
export interface KanbanColumnProps {
  column: KanbanColumnConfig;
  tasks: MockTask[];
  onTaskClick?: (task: MockTask) => void;
  isDragging?: boolean;  // <-- Them
}
```

### 3.2 Them import

```typescript
import { DropPlaceholder } from './drop-placeholder';
```

### 3.3 Them trailing droppable

```typescript
export function KanbanColumn({ column, tasks, onTaskClick, isDragging = false }: KanbanColumnProps) {
  const { setNodeRef, isOver } = useDroppable({ id: column.id });

  // Them droppable cho trailing placeholder
  const { setNodeRef: setTrailingRef, isOver: isOverTrailing } = useDroppable({
    id: `${column.id}-trailing`,
  });

  return (
    <div className="flex h-full w-80 flex-shrink-0 flex-col rounded-lg bg-white/40 p-4 backdrop-blur-sm">
      {/* Column header - giu nguyen */}
      ...

      {/* Droppable area */}
      <div
        ref={setNodeRef}
        className={cn(
          'flex-1 space-y-3 overflow-y-auto rounded-lg p-2 transition-colors',
          isOver && 'bg-gray-100/50'
        )}
      >
        <SortableContext items={tasks.map((t) => t.id)} strategy={verticalListSortingStrategy}>
          {tasks.map((task) => (
            <SortableTaskCard
              key={task.id}
              task={task}
              onClick={() => onTaskClick?.(task)}
            />
          ))}
        </SortableContext>

        {/* Trailing placeholder - chi hien khi dang drag */}
        {isDragging && (
          <div ref={setTrailingRef}>
            <DropPlaceholder isActive={isOverTrailing} />
          </div>
        )}
      </div>
    </div>
  );
}
```

### Acceptance
- [ ] Trailing placeholder hien khi drag
- [ ] An khi khong drag
- [ ] isOverTrailing highlight dung

---

## Task 4: Update sortable-task-card.tsx

**File:** `apps/web/src/features/design/components/kanban/sortable-task-card.tsx`

### 4.1 Them cn import

```typescript
import { cn } from '@/lib/utils';
```

### 4.2 Update style va className

```tsx
const style = {
  transform: CSS.Transform.toString(transform),
  transition,
};

return (
  <div
    ref={setNodeRef}
    style={style}
    className={cn(
      'transition-transform duration-200',
      isDragging && 'opacity-50 scale-[0.98]'
    )}
    {...attributes}
    {...listeners}
  >
    <TaskCard task={task} onTitleClick={onClick} isDragging={isDragging} />
  </div>
);
```

### Acceptance
- [ ] Dragging state co visual feedback
- [ ] Transition smooth

---

## Task 5: Testing

### Manual Test Cases

| # | Test Case | Expected |
|---|-----------|----------|
| 1 | Drag task trong column | Reorder smooth, gap animation |
| 2 | Drag task sang column khac | Insert truoc task dang hover |
| 3 | Drag vao trailing placeholder | Append cuoi column |
| 4 | Drag nhanh qua nhieu tasks | Khong flickering |
| 5 | Column rong | Trailing placeholder hoat dong |

### Performance Check

- Drag voi 45 tasks khong lag
- No unnecessary re-renders (React DevTools)

---

## Files Summary

| File | Action |
|------|--------|
| `drop-placeholder.tsx` | CREATE |
| `kanban-board.tsx` | UPDATE |
| `kanban-column.tsx` | UPDATE |
| `sortable-task-card.tsx` | UPDATE |

---

## Notes

- `handleDragOver` goi lien tuc khi drag -> can de y performance
- Neu flickering xay ra, can them throttle hoac debounce
- Neu collision detection khong chinh xac, thu `closestCenter` thay vi `closestCorners`
