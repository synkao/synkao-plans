---
title: "Kanban Drag-Drop Visual Feedback"
description: "Theem visual feedback khi drag task: drop indicator, insert above, trailing placeholder, reorder trong column"
status: completed
priority: P2
effort: 2h
branch: main
tags: [kanban, dnd-kit, ux, design-feature]
created: 2026-01-03
reviewed: 2026-01-03
completed: 2026-01-03T18:44:00Z
---

# Kanban Drag-Drop Visual Feedback

## Tong Quan

**Van de:** Kanban board hien tai thieu visual feedback khi drag task - khong co indicator cho vi tri drop, khong reorder duoc trong column.

**Giai phap:** Su dung full dnd-kit Sortable voi `handleDragOver`, `arrayMove`, va trailing placeholder.

## Yeu Cau

| # | Feature | Mo ta |
|---|---------|-------|
| 1 | Drop indicator | Hien thi placeholder mo o vi tri se drop |
| 2 | Insert above | Drag len task -> dat phia tren task do |
| 3 | Trailing placeholder | O mo cuoi column, chi khi dang drag |
| 4 | Reorder trong column | Sap xep lai thu tu tasks |

## Approach

**Chon:** Full dnd-kit Sortable

**Ly do:**
- dnd-kit built-in support cho sortable voi visual feedback
- It code, stable, well-tested
- Phu hop voi code hien tai da dung SortableContext
- KISS principle

## Files Can Sua

| File | Change |
|------|--------|
| `kanban-board.tsx` | Them `handleDragOver`, `arrayMove` logic, pass `isDragging` prop |
| `kanban-column.tsx` | Them trailing placeholder droppable, nhan `isDragging` prop |
| `sortable-task-card.tsx` | Cai thien visual khi isDragging |

## File Moi

| File | Purpose |
|------|---------|
| `drop-placeholder.tsx` | Component placeholder mo o cuoi column |

## Phases

| Phase | Mo ta | Effort | Status |
|-------|-------|--------|--------|
| [Phase 01](./phase-01-dnd-visual-feedback.md) | Implement visual feedback | 2h | ✅ DONE |

## Success Criteria

- [x] Drag task hien thi placeholder mo o vi tri drop
- [x] Drag len task khac -> dat phia tren (insert before)
- [x] Trailing placeholder xuat hien khi drag, bien mat khi khong drag
- [x] Reorder trong cung column hoat dong
- [x] Animation smooth, khong flickering
- [x] Performance OK voi 45 tasks (throttle applied + tested)

## Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Performance voi nhieu tasks | Medium | Throttle `handleDragOver` |
| Flickering khi drag nhanh | Low | CSS transition smooth |
| Collision detection khong chinh xac | Low | Test voi `closestCenter` neu can |

## Code Review Results

**Status:** ✅ APPROVED WITH CONDITIONS

**Score:** 8.5/10

**Critical Issues:** 0
**Required Fixes:** 2
1. Throttle `handleDragOver` (performance)
2. Add image alt text (accessibility)

**Report:** `plans/reports/code-reviewer-260103-1753-kanban-drag-feedback.md`

## Next Steps

1. ✅ Apply throttle to `handleDragOver` (16ms)
2. ✅ Fix image alt text warning
3. ✅ Manual test voi 45 tasks
4. ✅ Code review passed (8.5/10, 0 critical issues)
5. ✅ User approved
6. [OPTIONAL] Create comprehensive test suite (unit + integration) for regression prevention

## References

- Code Review: `plans/reports/code-reviewer-260103-1753-kanban-drag-feedback.md`
- Test Report: `plans/reports/tester-260103-1751-kanban-drag-drop.md`
- Brainstorm: `plans/reports/brainstorm-260103-1640-kanban-dnd-visual-feedback.md`
- dnd-kit docs: https://docs.dndkit.com/presets/sortable
