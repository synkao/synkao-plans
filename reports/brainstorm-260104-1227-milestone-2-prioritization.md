# Brainstorm Report: Milestone 2 Prioritization

**Date:** 2026-01-04
**Milestone:** [FE-2: Core Components](https://github.com/synkao/synkao/milestone/2)
**Status:** 0/8 issues completed

---

## Problem Statement

Milestone 2 chứa 8 issues (6 P0, 2 P1) cho Core Components của frontend. Tất cả đều OPEN và cần được prioritize để implement hiệu quả.

---

## Current State Analysis

| Issue | Title | Priority | Est | % Done | Notes |
|-------|-------|----------|-----|--------|-------|
| #6 | Layout: AppShell, GlassSidebar, PageHeader | P0 | 4h | ~80% | `glass-sidebar.tsx`, `page-header.tsx` ✅ |
| #7 | Glass Card variants | P0 | 3h | 0% | shadcn default, chưa glass |
| #8 | Status Badges | P0 | 2h | 0% | cần domain variants |
| #9 | Form components | P0 | 4h | ~40% | có input, select; thiếu FileUploader, DatePicker |
| #10 | Data Table | P0 | 4h | 0% | chưa có TanStack Table |
| #11 | Modal, Drawer, Toast | P0 | 3h | ~50% | có dialog, sheet; thiếu Toast |
| #63 | Login + Auth | P1 | ~4h | 0% | cần auth hooks, guards |
| #64 | MSW Mock Data Update | P1 | 4h | 0% | có plan sẵn |

---

## Dependency Analysis

```
#6 Layout (Foundation)
    ↓
#8 Badges → Kanban, Order views
    ↓
#11 Toast → User feedback
    ↓
#7 Glass Card → Visual consistency
    ↓
#9 Form → Data entry
    ↓
#10 DataTable → Order list
    ↓
#64 MSW → Dev experience
    ↓
#63 Login → Auth (defer đến BE ready)
```

---

## Recommended Priority Order

| Rank | Issue | Rationale | Remaining |
|------|-------|-----------|-----------|
| 1 | #6 | Foundation cho mọi page | ~1h |
| 2 | #8 | Quick win, high visibility | 2h |
| 3 | #11 | Toast critical cho UX | ~1.5h |
| 4 | #7 | Glass aesthetic | 3h |
| 5 | #9 | Form completion | ~2.5h |
| 6 | #10 | TanStack Table setup | 4h |
| 7 | #64 | Better mock data | 4h |
| 8 | #63 | Auth flow (P1) | ~4h |

**Total remaining effort: ~22h**

---

## Decisions Made

1. **Priority order:** #6 → #8 → #11 → #7 → #9 → #10 → #64 → #63
2. **Due date:** Removed (work by priority, not deadline)
3. **Approach:** Complete existing components trước, tạo mới sau

---

## Implementation Considerations

### Quick Wins (< 2h each)
- #6 remaining: AppShell wrapper chỉ là layout container
- #8 Badges: Extend existing `badge.tsx` với domain variants

### Complex Items
- #10 DataTable: Cần TanStack Table setup, column definitions, pagination logic
- #64 MSW: 4 phases, nhiều mock handlers

### Dependencies Ngoài Scope
- #63 Login: Phụ thuộc vào BE auth API (#33)

---

## Success Metrics

- [ ] 6/8 issues completed (P0 focus)
- [ ] All core components có Storybook stories
- [ ] Build passes, no type errors
- [ ] Components reusable across features

---

## Next Steps

1. Bắt đầu với #6 - hoàn thành AppShell wrapper
2. Tiếp #8 - Status Badges cho Kanban
3. Parallel: Document completed components

---

## Unresolved Questions

1. #63 Login: Wait for BE auth API hay mock toàn bộ?
2. Storybook: Có cần stories cho mỗi component không?
