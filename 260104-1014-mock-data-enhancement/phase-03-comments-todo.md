# Phase 03: Comments for TODO Tasks

## Context

- Parent: [plan.md](plan.md)
- Dependencies: Phase 00
- Docs: [codebase-summary.md](../../docs/codebase-summary.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P2 |
| Effort | 30m |
| Implementation Status | Pending |
| Review Status | Pending |

**Objective:** Thêm comments cho TODO tasks. Comments không cần link designVersionId vì TODO chưa có design.

## Key Insights

1. TODO tasks có thể có comments từ staff/customer trước khi assign
2. Comments không cần designVersionId (optional field)
3. Diverse comment contents cho realistic demo

## Requirements

### Comment Contents for TODO

- Customer requirements
- Rush order notes
- Reference clarifications
- Brand guidelines
- Priority explanations

### Users for Comments

- Staff users (not designers)
- Use USER_IDS.staff1, staff2

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `mocks/data/comments.data.ts` | Modify | Add TODO task comments |

## Implementation Steps

### Step 1: Define TODO comment contents

```typescript
const todoCommentContents = [
  'Customer prefers minimalist style',
  'Rush order - please prioritize',
  'Reference images attached above',
  'Use brand colors: #2563EB, #1E40AF',
  'Keep text simple and readable',
  'Customer confirmed artwork brief via email',
  'See attached brand guidelines',
  'Double-check spelling before design',
  'Product dimensions in order notes',
  'Reprint from last year - see reference',
];
```

### Step 2: Generate comments for TODO tasks

```typescript
const todoTasks = mockTasks.filter(t => t.status === 'TODO');

const todoComments = todoTasks.slice(0, 10).flatMap((task, idx) => {
  const numComments = 1 + (idx % 2); // 1-2 comments per task
  return Array.from({ length: numComments }, (_, i) => ({
    id: commentUUID(100 + idx * 2 + i),
    taskId: task.id,
    userId: USER_IDS[`staff${1 + (i % 2)}`],
    content: todoCommentContents[(idx + i) % todoCommentContents.length],
    // No designVersionId for TODO tasks
    createdAt: daysAgo(7 - idx),
  }));
});
```

### Step 3: Merge with existing comments

```typescript
export const mockComments = [
  ...existingComments,
  ...todoComments,
];
```

## Todo List

- [ ] Add todoCommentContents array
- [ ] Generate 15-20 comments for TODO tasks
- [ ] Merge with existing comments
- [ ] Sort by createdAt

## Success Criteria

- [ ] 10+ TODO tasks have comments
- [ ] No designVersionId on TODO comments
- [ ] Realistic content
- [ ] TypeScript compiles

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Duplicate comment IDs | Medium | Use separate UUID range (100+) |
