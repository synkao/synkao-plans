# Brainstorm: Kanban Drag & Title Click

## Problem Statement

| Issue | Description |
|-------|-------------|
| Drag delay | PointerSensor có `delay: 150ms` → cảm giác laggy |
| Click target | Hiện tại click cả card → mở drawer |
| Mong muốn | Drag tức thì, chỉ click title mở drawer |

## Evaluated Approaches

### 1. Delay-based (Hiện tại) ❌
- `activationConstraint: { delay: 150, tolerance: 5 }`
- Pros: Simple implementation
- Cons: Laggy UX, không responsive

### 2. Thumbnail as Drag Handle
- Drag từ thumbnail, click text mở drawer
- Pros: Rõ ràng
- Cons: Không intuitive, user phải biết

### 3. Grip Icon
- Thêm icon ⠿⠿ làm drag handle
- Pros: Rõ ràng, visual cue
- Cons: Thêm clutter UI

### 4. Card Drag + Title Click ✅ (Đã chọn)
- Toàn bộ card draggable (trừ title)
- Title có `stopPropagation` → click mở drawer
- Pros: Natural UX, no visual clutter
- Cons: Cần hiểu card không mở drawer khi click random

## Final Solution

### Architecture
```
┌─────────────────────────────────┐
│  SortableTaskCard               │
│  ├─ listeners trên wrapper      │
│  └─ TaskCard                    │
│      ├─ Thumbnail (draggable)   │
│      ├─ Title (clickable only)  │← stopPropagation
│      ├─ Customer (draggable)    │
│      ├─ Priority (draggable)    │
│      └─ Assignee (draggable)    │
└─────────────────────────────────┘
```

### Code Changes

#### 1. `kanban-board.tsx` - Thay delay → distance

```diff
  const sensors = useSensors(
    useSensor(PointerSensor, {
-     activationConstraint: { delay: 150, tolerance: 5 },
+     activationConstraint: { distance: 5 },
    })
  );
```

**Lý do:** `distance: 5` = drag sau khi di chuyển 5px. Click thường không di chuyển > 5px.

#### 2. `task-card.tsx` - Title riêng biệt

```diff
  export interface TaskCardProps {
    task: MockTask;
-   onClick?: () => void;
+   onTitleClick?: () => void;
    isDragging?: boolean;
  }

- export function TaskCard({ task, onClick, isDragging }: TaskCardProps) {
+ export function TaskCard({ task, onTitleClick, isDragging }: TaskCardProps) {
```

```diff
  <Card
    className={cn(
-     'cursor-pointer border border-gray-200/50 bg-white/60 p-3...',
+     'border border-gray-200/50 bg-white/60 p-3...',
      isDragging && 'opacity-50'
    )}
-   onClick={onClick}
  >
    ...
-   <h4 className="truncate text-sm font-semibold text-gray-900">
+   <h4
+     className="truncate text-sm font-semibold text-gray-900 cursor-pointer hover:underline hover:text-primary"
+     onPointerDown={(e) => e.stopPropagation()}
+     onClick={(e) => {
+       e.stopPropagation();
+       onTitleClick?.();
+     }}
+   >
      {task.productName}
    </h4>
```

#### 3. `sortable-task-card.tsx` - Đổi prop name

```diff
- <TaskCard task={task} onClick={onClick} isDragging={isDragging} />
+ <TaskCard task={task} onTitleClick={onClick} isDragging={isDragging} />
```

### UX Behavior After Change

| User Action | Result |
|-------------|--------|
| Click title (productName) | Drawer opens |
| Click thumbnail | Nothing (drag starts if moved) |
| Click priority badge | Nothing |
| Click assignee | Nothing |
| Pointer down + move 5px+ | Drag starts immediately |
| Quick click anywhere (except title) | Nothing |

## Implementation Considerations

### Risks
| Risk | Impact | Mitigation |
|------|--------|------------|
| User confusion | Low | Title có hover underline, visual cue |
| Accidental drag | Low | 5px threshold đủ để filter |
| Touch devices | Medium | Test trên mobile, có thể cần `TouchSensor` |

### Testing Checklist
- [ ] Drag từ thumbnail → works tức thì
- [ ] Drag từ priority/assignee → works
- [ ] Click title → drawer opens
- [ ] Click nhanh thumbnail → không mở drawer
- [ ] Touch drag (mobile) → works

## Success Metrics

- ✅ Drag không có delay (immediate after 5px)
- ✅ Title click mở drawer
- ✅ Các vùng khác không mở drawer
- ✅ Visual hover effect trên title

## Next Steps

1. Implement code changes (3 files)
2. Test drag/click behavior
3. Test touch devices
4. Update plan nếu cần

---

*Report generated: 2026-01-03*
