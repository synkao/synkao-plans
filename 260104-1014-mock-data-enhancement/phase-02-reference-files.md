# Phase 02: Reference Files Enhancement

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

**Objective:** Mở rộng reference files coverage từ 8 tasks → 20+ tasks, đảm bảo TODO tasks có reference files.

## Key Insights

1. Hiện có 15 reference files cho 8 tasks đầu tiên
2. TODO tasks (index 0-14) cần có reference files
3. picsum.photos cho placeholder thumbnails

## Requirements

### Expand Coverage

- Cover tất cả 15 TODO tasks
- Thêm 5 tasks khác (IN_PROGRESS, REVIEW)
- Tổng: 20 tasks có reference files

### Add More File Types

- Thêm các file types phổ biến: .ai, .psd, .docx, .zip
- File names realistic cho print industry

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `mocks/data/reference-files.data.ts` | Modify | Expand records |

## Implementation Steps

### Step 1: Update tasksForRefs slice

```typescript
// Mở rộng từ 8 → 20 tasks
const tasksForRefs = mockTasks.slice(0, 20);
```

### Step 2: Add file types

```typescript
const fileNames = [
  // Existing
  'brand-guidelines.pdf',
  'logo-reference.png',
  'color-palette.png',
  'product-photo.jpg',
  // New
  'customer-mockup.jpg',
  'print-template.ai',
  'artwork-brief.docx',
  'design-assets.zip',
  'font-files.zip',
  'vector-logo.svg',
];
```

### Step 3: Increase records per task

```typescript
// 1-3 files per task (random)
const filesPerTask = 1 + (idx % 3);
```

## Todo List

- [ ] Expand tasksForRefs to 20 tasks
- [ ] Add new file types
- [ ] Increase files per task
- [ ] Verify ~30-40 total reference files

## Success Criteria

- [ ] All 15 TODO tasks have at least 1 reference file
- [ ] Total reference files: 30-40
- [ ] TypeScript compiles
- [ ] UI renders correctly in drawer

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Too many files slow down | Low | Limit to 3 per task |
