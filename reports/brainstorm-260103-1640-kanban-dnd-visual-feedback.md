# Brainstorm: Kanban Drag-Drop Visual Feedback

## Tóm Tắt

**Vấn đề:** Kanban board hiện tại thiếu visual feedback khi drag task - không có indicator cho vị trí drop, không reorder được trong column.

**Giải pháp:** Sử dụng full dnd-kit Sortable với DragOver handler, arrayMove, và trailing placeholder.

---

## Yêu Cầu

| # | Feature | Mô tả |
|---|---------|-------|
| 1 | Drop indicator | Hiển thị placeholder mờ ở vị trí sẽ drop |
| 2 | Insert above | Drag lên task → đặt phía trên task đó |
| 3 | Trailing placeholder | Ô mờ cuối column, chỉ khi đang drag |
| 4 | Reorder trong column | Sắp xếp lại thứ tự tasks |

---

## Approach Đã Chọn: Full dnd-kit Sortable

### Rationale

- dnd-kit built-in support cho sortable với visual feedback
- Ít code, stable, well-tested
- Phù hợp với code hiện tại đã dùng SortableContext
- KISS principle - không over-engineer

---

## Implementation Plan

### Files Cần Sửa

| File | Change |
|------|--------|
| `kanban-board.tsx` | Thêm `handleDragOver`, `arrayMove` logic |
| `kanban-column.tsx` | Thêm trailing placeholder droppable |
| `sortable-task-card.tsx` | Cải thiện visual khi isDragging |

### File Mới

| File | Purpose |
|------|---------|
| `drop-placeholder.tsx` | Component placeholder mờ ở cuối column |

---

### Chi Tiết Implementation

#### 1. kanban-board.tsx

**Thay đổi chính:**

```typescript
// Thêm imports
import { arrayMove } from '@dnd-kit/sortable';
import type { DragOverEvent } from '@dnd-kit/core';

// Trong KanbanBoard component
const handleDragOver = (event: DragOverEvent) => {
  const { active, over } = event;
  if (!over) return;

  const activeId = active.id as string;
  const overId = over.id as string;

  // Tìm task đang drag và task đang hover
  const activeTask = tasks.find(t => t.id === activeId);
  const overTask = tasks.find(t => t.id === overId);

  if (!activeTask) return;

  // Case 1: Hover lên task khác (trong cùng column hoặc khác column)
  if (overTask) {
    if (activeTask.status !== overTask.status) {
      // Move sang column mới + insert trước overTask
      const updatedTasks = tasks.filter(t => t.id !== activeId);
      const overIndex = updatedTasks.findIndex(t => t.id === overId);
      const newTask = { ...activeTask, status: overTask.status };
      updatedTasks.splice(overIndex, 0, newTask);
      onTasksChange(updatedTasks);
    } else {
      // Reorder trong cùng column
      const oldIndex = tasks.findIndex(t => t.id === activeId);
      const newIndex = tasks.findIndex(t => t.id === overId);
      if (oldIndex !== newIndex) {
        onTasksChange(arrayMove(tasks, oldIndex, newIndex));
      }
    }
    return;
  }

  // Case 2: Hover lên column (empty area) hoặc trailing placeholder
  if (VALID_STATUSES.includes(overId as MockTask['status'])) {
    if (activeTask.status !== overId) {
      // Move sang column mới, append cuối
      const updatedTasks = tasks.map(t =>
        t.id === activeId ? { ...t, status: overId as MockTask['status'] } : t
      );
      onTasksChange(updatedTasks);
    }
  }

  // Case 3: Trailing placeholder (overId = `${columnId}-trailing`)
  if (overId.endsWith('-trailing')) {
    const columnId = overId.replace('-trailing', '') as MockTask['status'];
    if (activeTask.status !== columnId) {
      const updatedTasks = tasks.map(t =>
        t.id === activeId ? { ...t, status: columnId } : t
      );
      onTasksChange(updatedTasks);
    }
  }
};

// DndContext thêm onDragOver
<DndContext
  sensors={sensors}
  collisionDetection={closestCorners}
  onDragStart={handleDragStart}
  onDragOver={handleDragOver}  // ← Thêm
  onDragEnd={handleDragEnd}
>
```

**Lưu ý:** `handleDragOver` được gọi liên tục khi drag, nên cần optimize để tránh re-render không cần thiết.

---

#### 2. kanban-column.tsx

**Thêm trailing placeholder:**

```tsx
import { DropPlaceholder } from './drop-placeholder';

// Trong KanbanColumn
export function KanbanColumn({ column, tasks, onTaskClick, isDragging }: KanbanColumnProps) {
  const { setNodeRef, isOver } = useDroppable({ id: column.id });

  // Thêm droppable cho trailing
  const { setNodeRef: setTrailingRef, isOver: isOverTrailing } = useDroppable({
    id: `${column.id}-trailing`,
  });

  return (
    <div className="flex h-full w-80 flex-shrink-0 flex-col ...">
      {/* Column header */}
      ...

      {/* Droppable area */}
      <div ref={setNodeRef} className={cn('flex-1 space-y-3 ...', isOver && 'bg-gray-100/50')}>
        <SortableContext items={tasks.map(t => t.id)} strategy={verticalListSortingStrategy}>
          {tasks.map(task => (
            <SortableTaskCard key={task.id} task={task} onClick={() => onTaskClick?.(task)} />
          ))}
        </SortableContext>

        {/* Trailing placeholder - chỉ hiện khi đang drag */}
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

**Props mới:** `isDragging: boolean` - pass từ KanbanBoard

---

#### 3. drop-placeholder.tsx (File mới)

```tsx
import { cn } from '@/lib/utils';

interface DropPlaceholderProps {
  isActive?: boolean;
}

/**
 * Placeholder hiển thị khi drag task - đặt cuối column
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
        {isActive ? 'Thả vào đây' : 'Kéo vào đây'}
      </span>
    </div>
  );
}
```

---

#### 4. sortable-task-card.tsx

**Cải thiện visual:**

```tsx
const style = {
  transform: CSS.Transform.toString(transform),
  transition,
  opacity: isDragging ? 0.5 : 1,  // ← Đã có
};

// Thêm class cho animation khi có task đang hover phía trên
return (
  <div
    ref={setNodeRef}
    style={style}
    className={cn(
      'transition-transform',
      // Khi có item đang hover → tạo gap animation
    )}
    {...attributes}
    {...listeners}
  >
    <TaskCard task={task} onTitleClick={onClick} isDragging={isDragging} />
  </div>
);
```

---

### Visual Behavior Summary

| State | Visual |
|-------|--------|
| Đang drag task | Task gốc opacity 50%, DragOverlay hiện task preview |
| Hover lên task khác | Task đó dịch xuống tạo gap |
| Hover lên trailing placeholder | Placeholder highlight border + bg |
| Drop | Smooth animation về vị trí mới |

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Performance với nhiều tasks | Medium | Throttle `handleDragOver`, virtualize list nếu cần |
| Flickering khi drag nhanh | Low | CSS transition smooth, debounce state updates |
| Collision detection không chính xác | Low | Test với `closestCenter` nếu `closestCorners` có issues |

---

## Success Criteria

- [ ] Drag task hiển thị placeholder mờ ở vị trí drop
- [ ] Drag lên task khác → đặt phía trên (insert before)
- [ ] Trailing placeholder xuất hiện khi drag, biến mất khi không drag
- [ ] Reorder trong cùng column hoạt động
- [ ] Animation smooth, không flickering
- [ ] Performance OK với 45 tasks

---

## Next Steps

1. Implement theo plan trên
2. Test các edge cases: empty column, fast drag, cross-column
3. Polish animation nếu cần
