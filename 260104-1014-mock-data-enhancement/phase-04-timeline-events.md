# Phase 04: Timeline Events Enhancement

## Context

- Parent: [plan.md](plan.md)
- Dependencies: Phase 02 (needs comments data)
- Docs: [codebase-summary.md](../../docs/codebase-summary.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P2 |
| Effort | 30m |
| Implementation Status | Pending |
| Review Status | Pending |

**Objective:** Thêm COMMENT_ADDED events vào timeline từ mockComments data.

## Key Insights

1. Timeline hiện có: TASK_CREATED, STATUS_CHANGED, VERSION_UPLOADED
2. Thiếu: COMMENT_ADDED events
3. Import mockComments và generate events từ đó

## Requirements

### Add COMMENT_ADDED Events

- Generate from mockComments
- Include comment preview in description
- Link to commentId in metadata

### Event Type (if not exists)

```typescript
type TimelineEventType =
  | 'TASK_CREATED'
  | 'STATUS_CHANGED'
  | 'VERSION_UPLOADED'
  | 'COMMENT_ADDED'; // Add this
```

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `mocks/data/timeline-events.data.ts` | Modify | Add COMMENT_ADDED generation |
| `mocks/types.ts` | Check | Verify COMMENT_ADDED in type |

## Implementation Steps

### Step 1: Import mockComments

```typescript
import { mockComments } from './comments.data';
```

### Step 2: Generate COMMENT_ADDED events

```typescript
// After existing event generation
const commentEvents = mockComments.map((comment, idx) => ({
  id: eventUUID(200 + idx),
  taskId: comment.taskId,
  type: 'COMMENT_ADDED' as const,
  userId: comment.userId,
  description: `Added comment: "${comment.content.slice(0, 40)}..."`,
  metadata: { commentId: comment.id },
  createdAt: comment.createdAt,
}));
```

### Step 3: Merge and sort

```typescript
export const mockTimelineEvents = [
  ...existingEvents,
  ...commentEvents,
].sort((a, b) =>
  new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime()
);
```

## Todo List

- [ ] Verify COMMENT_ADDED in TimelineEventType
- [ ] Import mockComments
- [ ] Generate COMMENT_ADDED events
- [ ] Merge and sort events
- [ ] Verify total ~120-130 events

## Success Criteria

- [ ] COMMENT_ADDED events appear in timeline
- [ ] Events sorted by date (newest first)
- [ ] TODO tasks have timeline events
- [ ] TypeScript compiles

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Circular import | High | Check import order |
| Duplicate event IDs | Medium | Use separate UUID range (200+) |
