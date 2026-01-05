---
title: "Simplify Priority → isUrgent Boolean"
description: "Thay thế Priority enum 4 mức bằng isUrgent boolean đơn giản"
status: done
priority: P3
effort: 2h
branch: main
tags: [refactor, simplify, mvp, ui]
created: 2026-01-05
completed: 2026-01-05
---

# Plan: Simplify Priority → isUrgent Boolean

## Overview

Đơn giản hóa hệ thống priority từ enum 4 mức (LOW|NORMAL|HIGH|URGENT) xuống boolean `isUrgent`.

**Brainstorm Report:** [brainstorm-260105-1003-simplify-priority-to-urgent-flag.md](../reports/brainstorm-260105-1003-simplify-priority-to-urgent-flag.md)

## Motivation

- User ít khi dùng đủ 4 mức priority
- MVP cần đơn giản hóa scope
- Chỉ cần biết "có cần làm gấp không"
- Phù hợp nguyên tắc KISS/YAGNI

## Impact Analysis

| Category | Files Affected |
|----------|----------------|
| Types | 2 files |
| Constants | 2 files |
| Components | 6 files |
| Mock Data | 3 files |
| **Total** | **13 files** |

## Phases

| Phase | Description | Status | Progress |
|-------|-------------|--------|----------|
| [Phase 01](./phase-01-update-types-constants.md) | Update types & constants | ✅ Done | 100% |
| [Phase 02](./phase-02-update-mock-data.md) | Update mock data & handlers | ✅ Done | 100% |
| [Phase 03](./phase-03-update-components.md) | Update UI components | ✅ Done | 100% |

## Success Criteria

- [ ] Tất cả references đến `Priority` enum được xóa
- [ ] `isUrgent: boolean` hoạt động đúng
- [ ] UI chỉ hiển thị badge khi urgent
- [ ] Build thành công, không TypeScript errors
- [ ] Bulk action "Set Priority" đổi thành "Toggle Urgent"

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Breaking existing features | Low | Mock data, không có real data |
| TypeScript errors | Low | Thorough grep + IDE support |

## Next Steps

1. Review plan với user
2. Implement Phase 01: Types & Constants
3. Implement Phase 02: Mock Data
4. Implement Phase 03: Components
5. Test toàn bộ flow
