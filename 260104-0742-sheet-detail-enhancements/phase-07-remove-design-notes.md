# Phase 07: Remove Design Notes Section

## Context

- Parent: [plan.md](plan.md)
- Dependencies: Phase 02 (Print Specs replaces Design Notes)
- Docs: [codebase-summary.md](../../docs/codebase-summary.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P2 |
| Effort | 5m |
| Implementation Status | Done |
| Review Status | Done |

**Objective:** Xóa DesignNotesSection vì Print Specs đã thay thế chức năng hiển thị thông tin design.

## Key Insights

1. DesignNotesSection chỉ hiển thị read-only notes từ order item
2. Print Specs section đã có thể chứa thông tin này dưới dạng key-value
3. Redundant UI - cần cleanup

## Requirements

### Remove DesignNotesSection

- Xóa import từ task-detail-drawer.tsx
- Xóa component usage và Separator liên quan
- Xóa file design-notes-section.tsx
- Update index.ts exports

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `drawer/task-detail-drawer.tsx` | Modify | Remove import và usage |
| `drawer/design-notes-section.tsx` | Delete | Remove file |
| `drawer/index.ts` | Modify | Remove export |

## Implementation Steps

### Step 1: Update task-detail-drawer.tsx

Remove import và component usage.

### Step 2: Delete design-notes-section.tsx

Remove file completely.

### Step 3: Update index.ts

Remove export.

## Todo List

- [x] Remove DesignNotesSection from drawer
- [x] Delete design-notes-section.tsx
- [x] Update index.ts exports
- [x] Verify TypeScript

## Success Criteria

- [x] No DesignNotesSection in drawer
- [x] File deleted
- [x] TypeScript compiles
- [x] No unused imports

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Breaking change | Low | Print Specs covers same info |

## Security Considerations

None - UI cleanup only.
