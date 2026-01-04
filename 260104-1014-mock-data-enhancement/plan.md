---
title: "Mock Data Enhancement cho TaskDetailDrawer"
description: "Bổ sung mock data toàn diện: TODO tasks trong workspace, reference files, comments, timeline events"
status: completed
priority: P2
effort: 3h
branch: main
tags: [mock-data, enhancement, task-detail-drawer]
created: 2026-01-04
---

# Mock Data Enhancement cho TaskDetailDrawer

## Tổng quan

Bổ sung mock data toàn diện để hỗ trợ development và testing TaskDetailDrawer. Hiện tại mock data đã có cấu trúc tốt nhưng cần mở rộng coverage cho các trường hợp sử dụng.

## Phân tích hiện trạng

### Files hiện có
| File | Records | Ghi chú |
|------|---------|---------|
| `tasks.data.ts` | 45 | 15 TODO, 10 IN_PROGRESS, 8 REVIEW, 5 REVISION, 7 DONE |
| `order-items.data.ts` | 45 | Tất cả có printSpecs theo productType |
| `design-versions.data.ts` | 20 | Chỉ cho REVIEW/DONE tasks |
| `reference-files.data.ts` | 15 | Chỉ cho 8 tasks đầu tiên |
| `comments.data.ts` | 25 | Chỉ cho tasks có design versions |
| `timeline-events.data.ts` | ~100 | Auto-generated từ tasks + versions |

### Gaps cần xử lý
1. **TODO tasks không có workspaceId**: `workspaceId: undefined` → không hiển thị trong Kanban
2. **TODO tasks từ REVISION không có old versions**: Flow REVISION → TODO nhưng không có design versions cũ
3. **Reference files**: Chỉ cover 8/45 tasks → cần mở rộng
4. **Comments**: Chỉ cho tasks có design versions → TODO tasks không có comments
5. **Timeline events**: TODO tasks chỉ có TASK_CREATED event

---

## Implementation Phases

| Phase | Name | Effort | Status | Progress |
|-------|------|--------|--------|----------|
| 00 | [Fix TODO Tasks Workspace](phase-00-fix-todo-workspace.md) | 5m | Done | 100% |
| 01 | [TODO Tasks with Old Versions](phase-01-todo-old-versions.md) | 20m | Done | 100% |
| 02 | [Reference Files Enhancement](phase-02-reference-files.md) | 30m | Done | 100% |
| 03 | [Comments for TODO Tasks](phase-03-comments-todo.md) | 30m | Done | 100% |
| 04 | [Timeline Events Enhancement](phase-04-timeline-events.md) | 30m | Done | 100% |
| 05 | [Validation & Testing](phase-05-validation.md) | 30m | Done | 100% |

---

## Phase 0: Fix TODO Tasks Workspace (5 phút)

### Vấn đề
TODO tasks có `workspaceId: undefined` → không hiển thị trong Kanban khi filter theo workspace.

### Nguyên nhân
```typescript
// tasks.data.ts line 27
const hasWorkspace = status !== 'TODO';
workspaceId: hasWorkspace ? WORKSPACE_IDS.main : undefined
```

### Giải pháp
Tất cả tasks đều thuộc main workspace:
```typescript
// Xóa logic hasWorkspace
workspaceId: WORKSPACE_IDS.main
```

### File thay đổi
| File | Action |
|------|--------|
| `tasks.data.ts` | Remove hasWorkspace condition |

---

## Phase 1: TODO Tasks with Old Versions (20 phút)

### Vấn đề
Theo flow: REVISION → TODO (restart). Nhưng hiện tại TODO tasks không có old design versions.

### Mục tiêu
Một số TODO tasks có design versions cũ (đã từng submit nhưng bị reject → restart).

### Thay đổi

**File: `design-versions.data.ts`**

```typescript
// Thêm 3-5 TODO tasks có old versions (simulating REVISION → TODO)
// Lấy TODO tasks index 10-14 (5 tasks cuối trong TODO group)
const todoTasksWithVersions = mockTasks
  .filter(t => t.status === 'TODO')
  .slice(-5); // Last 5 TODO tasks

// Generate 1-2 versions cho mỗi task (rejected versions)
todoTasksWithVersions.forEach((task, idx) => {
  mockDesignVersions.push({
    id: designVersionUUID(100 + idx),
    taskId: task.id,
    version: 1,
    status: 'REJECTED',
    fileUrl: `https://picsum.photos/seed/rejected-${idx}/800/600`,
    thumbnailUrl: `https://picsum.photos/seed/rejected-${idx}/200/150`,
    fileName: `design-v1-rejected.png`,
    fileSize: 1024 * 500, // 500KB
    notes: 'Design rejected - needs revision',
    uploadedById: designerIds[idx % 3],
    createdAt: daysAgo(7 - idx),
  });
});
```

**File: `timeline-events.data.ts`**

```typescript
// Thêm events cho TODO tasks có old versions:
// 1. VERSION_UPLOADED
// 2. STATUS_CHANGED (TODO → IN_PROGRESS → REVIEW → REVISION → TODO)
```

### File thay đổi
| File | Action |
|------|--------|
| `design-versions.data.ts` | Add rejected versions for TODO tasks |
| `timeline-events.data.ts` | Add status change history |

---

## Phase 2: Mở rộng Reference Files (30 phút)

### Mục tiêu
Đảm bảo TODO tasks có reference files để hiển thị trong drawer.

### Thay đổi

**File: `reference-files.data.ts`**

```typescript
// Thêm reference files cho TODO tasks (task index 1-15)
// Mở rộng từ 8 tasks → 20 tasks (bao gồm tất cả 15 TODO tasks)
const tasksForRefs = mockTasks.slice(0, 20);

// Tăng tổng số reference files từ 15 → 30
// Đảm bảo mỗi TODO task có ít nhất 1 reference file
```

**Thêm file names mới:**
```typescript
const fileNames = [
  // Existing
  'brand-guidelines.pdf',
  'logo-reference.png',
  // Thêm mới
  'customer-mockup.jpg',
  'print-template.ai',
  'artwork-brief.docx',
  'design-assets.zip',
];
```

---

## Phase 2: Mở rộng Comments cho TODO Tasks (30 phút)

### Mục tiêu
TODO tasks có thể có comments từ staff/customer trước khi assign cho designer.

### Thay đổi

**File: `comments.data.ts`**

```typescript
// Thêm comments cho TODO tasks (không cần designVersionId)
const todoTasks = mockTasks.filter(t => t.status === 'TODO');

// Thêm 15-20 comments cho TODO tasks
const todoCommentContents = [
  'Customer prefers minimalist style',
  'Rush order - high priority',
  'Reference images attached',
  'Use brand colors: #2563EB, #1E40AF',
  'Text: Keep it simple',
  'Customer confirmed artwork brief',
];

// Tách logic:
// 1. Comments cho tasks với versions (existing)
// 2. Comments cho TODO tasks (new)
```

---

## Phase 3: Cải thiện Timeline Events (30 phút)

### Mục tiêu
Timeline events phong phú hơn, bao gồm COMMENT_ADDED events.

### Thay đổi

**File: `timeline-events.data.ts`**

```typescript
// Thêm COMMENT_ADDED events từ mockComments
import { mockComments } from './comments.data';

mockComments.forEach((comment) => {
  events.push({
    id: generateEventId(eventIndex++),
    taskId: comment.taskId,
    type: 'COMMENT_ADDED',
    userId: comment.userId,
    description: comment.content.slice(0, 50) + '...',
    metadata: { commentId: comment.id },
    createdAt: comment.createdAt,
  });
});
```

**Thêm event types mới (nếu cần):**
- `REFERENCE_UPLOADED`: Khi staff upload reference file
- `PRIORITY_CHANGED`: Thay đổi priority

---

## Phase 4: Validation & Testing (30 phút)

### Checklist

1. **TypeScript compilation**: `pnpm build:web` không lỗi
2. **Data relationships**:
   - Tất cả `taskId` trong comments/refs trỏ đến task hợp lệ
   - Tất cả `userId` trỏ đến user hợp lệ
3. **Coverage verification**:
   ```typescript
   // Helper để verify coverage
   const todoTasks = mockTasks.filter(t => t.status === 'TODO');
   const todoWithRefs = todoTasks.filter(t =>
     mockReferenceFiles.some(r => r.taskId === t.id)
   );
   console.assert(todoWithRefs.length >= 10, 'TODO tasks cần có ref files');
   ```
4. **UI rendering**: Mở TaskDetailDrawer với TODO task, verify hiển thị đúng

---

## Summary thay đổi

| File | Thay đổi |
|------|----------|
| `tasks.data.ts` | Fix workspaceId cho TODO tasks |
| `design-versions.data.ts` | +5 rejected versions cho TODO tasks (REVISION → TODO flow) |
| `reference-files.data.ts` | +15 records, cover TODO tasks |
| `comments.data.ts` | +15-20 records cho TODO tasks |
| `timeline-events.data.ts` | +COMMENT_ADDED, +STATUS_CHANGED history |
| `mock-factory.ts` | Không đổi (existing helpers đủ dùng) |
| `types.ts` | Không đổi |

## Không cần thay đổi

- **order-items.data.ts**: Đã có printSpecs cho tất cả items

---

## Câu hỏi chưa giải quyết

1. Có cần thêm mock data cho file upload flow không? (preview → upload → confirm)
2. Timeline events có cần paginate không? (hiện tại ~100 events)
