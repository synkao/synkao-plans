# Phase 01: TODO Tasks with Old Versions

## Context

- Parent: [plan.md](plan.md)
- Dependencies: Phase 00
- Docs: [codebase-summary.md](../../docs/codebase-summary.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P2 |
| Effort | 20m |
| Implementation Status | Pending |
| Review Status | Pending |

**Objective:** Thêm old design versions cho một số TODO tasks, simulating flow REVISION → TODO (restart).

## Key Insights

1. Theo workflow: REVISION → TODO khi task cần restart
2. TODO tasks có thể có rejected versions từ trước
3. Timeline cần show history: VERSION_UPLOADED → STATUS_CHANGED

## Requirements

### Add Rejected Versions for TODO Tasks

- 5 TODO tasks cuối (index 10-14) có old versions
- Version status: REJECTED
- Version có notes explaining rejection

### Timeline History

- STATUS_CHANGED events showing: TODO → IN_PROGRESS → REVIEW → REVISION → TODO
- VERSION_UPLOADED events for rejected versions

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `mocks/data/design-versions.data.ts` | Modify | Add rejected versions |
| `mocks/data/timeline-events.data.ts` | Modify | Add status change history |

## Implementation Steps

### Step 1: Update design-versions.data.ts

```typescript
// Add rejected versions for last 5 TODO tasks
const todoTasksWithVersions = mockTasks
  .filter(t => t.status === 'TODO')
  .slice(-5);

const rejectedVersions = todoTasksWithVersions.map((task, idx) => ({
  id: designVersionUUID(100 + idx),
  taskId: task.id,
  version: 1,
  status: 'REJECTED' as const,
  fileUrl: `https://picsum.photos/seed/rejected-${task.id}/800/600`,
  thumbnailUrl: `https://picsum.photos/seed/rejected-${task.id}/200/150`,
  fileName: `design-v1-rejected.png`,
  fileSize: 1024 * (300 + idx * 100),
  notes: 'Design rejected - customer requested changes',
  uploadedById: designerIds[idx % 3],
  createdAt: daysAgo(10 - idx),
}));
```

### Step 2: Update timeline-events.data.ts

Add status change events for TODO tasks with versions:
- TASK_CREATED
- STATUS_CHANGED: TODO → IN_PROGRESS
- VERSION_UPLOADED
- STATUS_CHANGED: IN_PROGRESS → REVIEW
- STATUS_CHANGED: REVIEW → REVISION
- STATUS_CHANGED: REVISION → TODO (restart)

## Todo List

- [ ] Add rejected versions for 5 TODO tasks
- [ ] Add status change history in timeline
- [ ] Verify TypeScript compiles
- [ ] Test drawer shows old versions

## Success Criteria

- [ ] 5 TODO tasks have rejected design versions
- [ ] Timeline shows complete status history
- [ ] Drawer shows version history section
- [ ] TypeScript compiles without errors

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Circular import | Medium | Check import order between files |
| ID conflicts | Low | Use separate UUID range (100+) |

## Security Considerations

None - mock data only.
