# Debug Report: TypeError in groupTasksByStatus

**Date:** 2026-01-02
**Severity:** High
**Runtime Error:** `Cannot read properties of undefined (reading 'push')`
**Location:** `src/features/design/lib/utils.ts:16:26`

---

## Executive Summary

Runtime TypeError xảy ra khi `groupTasksByStatus()` cố gắng push task vào array không tồn tại trong `grouped` object. Error trigger khi user click vào task detail trong workspace kanban board.

**Root Cause:** `grouped` object chỉ khởi tạo 5 status keys cố định (TODO, IN_PROGRESS, REVIEW, REVISION, DONE). Nếu bất kỳ task nào có `status` value ngoài 5 giá trị này, `grouped[task.status]` trả về `undefined`, dẫn đến crash khi gọi `.push()`.

**Business Impact:**
- User không thể xem task detail
- Kanban board crash khi có task status không hợp lệ
- UX bị gián đoạn nghiêm trọng

---

## Technical Analysis

### 1. Error Stack Trace

```
groupTasksByStatus (utils.ts:16:26)
  ↓
KanbanBoard component (kanban-board.tsx:28:42)
  ↓
WorkspacePage (page.tsx:71:9)
```

### 2. Root Cause Identification

**File:** `src/features/design/lib/utils.ts`

```typescript
export function groupTasksByStatus(tasks: MockTask[]): Record<MockTask['status'], MockTask[]> {
  const grouped: Record<MockTask['status'], MockTask[]> = {
    TODO: [],
    IN_PROGRESS: [],
    REVIEW: [],
    REVISION: [],
    DONE: [],
  };

  tasks.forEach((task) => {
    grouped[task.status].push(task);  // ← LINE 16: CRASH HERE
  });

  return grouped;
}
```

**Vấn đề:**
- `grouped` object literal chỉ khởi tạo 5 keys cố định
- TypeScript type annotation `Record<MockTask['status'], MockTask[]>` không enforce runtime safety
- Nếu `task.status` có giá trị ngoài 5 values → `grouped[task.status]` = `undefined` → crash

### 3. Data Flow Trace

**WorkspacePage → KanbanBoard → groupTasksByStatus**

```typescript
// page.tsx:30 - Load tasks from hook
const { tasks, setTasks } = useKanbanState(workspace.id);

// page.tsx:40-43 - Apply designer filter
const filteredTasks = useMemo(() => {
  if (designerFilter === 'ALL') return tasks;
  return tasks.filter((task) => task.assigneeId === designerFilter);
}, [tasks, designerFilter]);

// page.tsx:71 - Pass to KanbanBoard
<KanbanBoard
  tasks={filteredTasks}  // ← Tasks passed here
  onTasksChange={setTasks}
  onTaskClick={handleTaskClick}
/>

// kanban-board.tsx:28 - Group tasks
const groupedTasks = groupTasksByStatus(tasks);  // ← CRASH HERE if invalid status
```

### 4. useKanbanState Hook Analysis

**File:** `src/features/design/hooks/use-kanban-state.ts`

```typescript
export function useKanbanState(workspaceId?: string) {
  const [tasks, setTasks] = useState<MockTask[]>(() => {
    if (workspaceId) {
      return mockTasks.filter((task) => task.workspaceId === workspaceId);
    }
    return mockTasks;
  });

  // Re-filter when workspaceId changes
  useEffect(() => {
    if (workspaceId) {
      setTasks(mockTasks.filter((task) => task.workspaceId === workspaceId));
    } else {
      setTasks(mockTasks);
    }
  }, [workspaceId, mockTasks]);

  // ...
}
```

**Vấn đề tiềm ẩn:**
- Hook dependency array có `mockTasks` (module-level constant) → không cần thiết nhưng vô hại
- Filter logic đơn giản, không validate task.status

### 5. Drag & Drop Flow

**File:** `src/features/design/components/kanban/kanban-board.tsx`

```typescript
const handleDragEnd = (event: DragEndEvent) => {
  const { active, over } = event;
  if (!over) return;

  const taskId = active.id as string;
  const newStatus = over.id as MockTask['status'];  // ← Potential issue

  const taskIndex = tasks.findIndex((t) => t.id === taskId);
  if (taskIndex === -1) return;

  const task = tasks[taskIndex];
  if (!task || task.status === newStatus) return;

  // Update task status
  const updatedTasks = [...tasks];
  updatedTasks[taskIndex] = { ...task, status: newStatus };  // ← Status update
  onTasksChange(updatedTasks);
};
```

**Potential vulnerability:**
- `over.id` được cast thành `MockTask['status']` không có runtime validation
- Nếu DnD library trả về invalid ID → task.status có thể bị set giá trị không hợp lệ

### 6. Mock Data Validation

**File:** `src/mocks/data/tasks.data.ts`

```typescript
const statusDistribution: MockTask['status'][] = [
  ...Array(15).fill('TODO'),
  ...Array(10).fill('IN_PROGRESS'),
  ...Array(8).fill('REVIEW'),
  ...Array(5).fill('REVISION'),
  ...Array(7).fill('DONE'),
];

export const mockTasks: MockTask[] = mockOrderItems.map((item, idx) => {
  const status = statusDistribution[idx]!;
  // ...
  return {
    // ...
    status,
    // ...
  };
});
```

**Analysis:**
- Mock data generation looks correct
- Status values hardcoded as strings trong `statusDistribution`
- Array access `statusDistribution[idx]!` với non-null assertion

**Type Definition:** `src/mocks/types.ts`

```typescript
export interface MockTask {
  // ...
  status: 'TODO' | 'IN_PROGRESS' | 'REVIEW' | 'REVISION' | 'DONE';
  // ...
}
```

TypeScript union type đúng với 5 values trong `grouped` object.

---

## Evidence & Scenarios

### Scenario 1: Runtime Status Mutation
Nếu task.status bị mutate ở runtime (vd: qua localStorage, API response, hoặc drag-drop bug):

```typescript
// Example: Invalid status từ external source
const task = {
  id: '...',
  status: 'PENDING',  // ← Not in grouped keys!
  // ...
};

groupTasksByStatus([task]);  // ← CRASH
```

### Scenario 2: Drag-Drop Type Casting Issue
```typescript
// kanban-board.tsx:42
const newStatus = over.id as MockTask['status'];  // ← Force cast

// Nếu over.id = 'invalid-column-id'
// → task.status = 'invalid-column-id'
// → groupTasksByStatus crash on next render
```

### Scenario 3: State Persistence Corruption
Nếu `setTasks` được gọi với tasks có status không hợp lệ:

```typescript
// Example: Load from localStorage
const savedTasks = JSON.parse(localStorage.getItem('tasks') || '[]');
setTasks(savedTasks);  // ← Không validate status

// Nếu savedTasks chứa outdated/corrupted status values → crash
```

---

## Constants Validation

**File:** `src/features/design/lib/constants.ts`

```typescript
export const KANBAN_COLUMNS: KanbanColumnConfig[] = [
  { id: 'TODO', ... },
  { id: 'IN_PROGRESS', ... },
  { id: 'REVIEW', ... },
  { id: 'REVISION', ... },
  { id: 'DONE', ... },
];
```

KANBAN_COLUMNS IDs match với `grouped` keys → Drag-drop giữa valid columns không gây lỗi.

---

## Suggested Fix (Not Implemented)

### Option 1: Dynamic Initialization (Safest)

```typescript
export function groupTasksByStatus(tasks: MockTask[]): Record<MockTask['status'], MockTask[]> {
  const grouped: Record<string, MockTask[]> = {};

  tasks.forEach((task) => {
    if (!grouped[task.status]) {
      grouped[task.status] = [];
    }
    grouped[task.status].push(task);
  });

  return grouped as Record<MockTask['status'], MockTask[]>;
}
```

**Pros:**
- Không crash với invalid status
- Linh hoạt với status values mới

**Cons:**
- Invalid statuses vẫn được group (có thể không mong muốn)
- Type casting cần thiết

### Option 2: Defensive with Fallback

```typescript
export function groupTasksByStatus(tasks: MockTask[]): Record<MockTask['status'], MockTask[]> {
  const grouped: Record<MockTask['status'], MockTask[]> = {
    TODO: [],
    IN_PROGRESS: [],
    REVIEW: [],
    REVISION: [],
    DONE: [],
  };

  tasks.forEach((task) => {
    const statusArray = grouped[task.status];
    if (statusArray) {
      statusArray.push(task);
    } else {
      console.error(`Invalid task status: ${task.status}`, task);
      // Fallback: push to TODO hoặc skip
      grouped.TODO.push(task);
    }
  });

  return grouped;
}
```

**Pros:**
- Safe với invalid status
- Log errors để debug
- Fallback behavior rõ ràng

**Cons:**
- Thêm runtime overhead (nhỏ)

### Option 3: Runtime Validation

```typescript
const VALID_STATUSES = ['TODO', 'IN_PROGRESS', 'REVIEW', 'REVISION', 'DONE'] as const;

export function groupTasksByStatus(tasks: MockTask[]): Record<MockTask['status'], MockTask[]> {
  const grouped: Record<MockTask['status'], MockTask[]> = {
    TODO: [],
    IN_PROGRESS: [],
    REVIEW: [],
    REVISION: [],
    DONE: [],
  };

  tasks.forEach((task) => {
    if (VALID_STATUSES.includes(task.status as any)) {
      grouped[task.status].push(task);
    } else {
      throw new Error(`Invalid task status: ${task.status} for task ${task.id}`);
    }
  });

  return grouped;
}
```

**Pros:**
- Fail-fast với clear error message
- Bắt bugs sớm trong development

**Cons:**
- Vẫn crash (nhưng với better error message)

### Option 4: Fix Drag-Drop Type Safety

```typescript
// kanban-board.tsx
const handleDragEnd = (event: DragEndEvent) => {
  const { active, over } = event;
  if (!over) return;

  const taskId = active.id as string;
  const newStatus = over.id as string;

  // Validate status
  const validStatuses: MockTask['status'][] = ['TODO', 'IN_PROGRESS', 'REVIEW', 'REVISION', 'DONE'];
  if (!validStatuses.includes(newStatus as MockTask['status'])) {
    console.error('Invalid drop target:', newStatus);
    return;
  }

  // ... rest of handler
};
```

---

## Prevention Measures

1. **Runtime validation** trong drag-drop handler
2. **Defensive coding** trong `groupTasksByStatus`
3. **Add unit tests** cho edge cases:
   - Empty tasks array
   - Tasks với invalid status
   - Tasks với undefined/null status
4. **Zod schema validation** nếu tasks load từ external sources
5. **Error boundary** ở component level để catch runtime errors

---

## Unresolved Questions

1. **User action trigger:** Tại sao click vào task detail lại trigger error trong `groupTasksByStatus`? Component có re-render với corrupted state?
2. **Persistence layer:** App có persist tasks state vào localStorage/sessionStorage không? Nếu có, schema migration strategy?
3. **Actual runtime data:** Task nào trigger error? Status value thực tế là gì?
4. **Error frequency:** Lỗi này xảy ra consistently hay intermittent?
5. **Recent changes:** Có deploy/migration nào làm thay đổi status enum values không?

---

## Recommended Next Steps

1. Add console.log trong `groupTasksByStatus` để log task.status values
2. Add error boundary với better error reporting
3. Implement Option 2 (Defensive with Fallback) cho immediate fix
4. Add Zod validation cho task data persistence
5. Investigate localStorage/state persistence corruption
