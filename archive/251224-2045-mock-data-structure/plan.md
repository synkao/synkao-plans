---
title: "Mock Data Structure"
description: "Tạo TypeScript mock data với factory pattern cho MSW testing"
status: completed
priority: P2
effort: 3h
issue: 4
branch: main
tags: [mocks, testing, msw, backend]
created: 2025-12-24
updated: 2025-12-24T23:50Z
---

# Mock Data Structure Implementation Plan

## Overview

Tạo comprehensive mock data structure sử dụng TypeScript files với factory pattern. Thay thế inline data trong handlers bằng imported data files có referential integrity.

**Source:** [Brainstorm Report](../reports/brainstorm-251224-2025-mock-data-structure.md)

## Phases

| # | Phase | Status | Effort | Link |
|---|-------|--------|--------|------|
| 1 | Update Types | Done | 30m | [phase-01](./phase-01-update-types.md) |
| 2 | Create Data Files | Done | 2h | [phase-02-create-data-files.md](./phase-02-create-data-files.md) |
| 3 | Update Handlers | Done ✅ | 30m | [phase-03-update-handlers.md](./phase-03-update-handlers.md) |

## Data Model

```
User (6) → Designer assigned to Tasks
Workspace (2) → Columns (5 default)
Order (15) → OrderItems (45, ~3/order)
OrderItem → Task (1:1)
Task → DesignVersions (20, only REVIEW/DONE)
```

## Dependencies

- MSW 2.x đã setup
- Existing types: MockUser, MockTask, MockOrder, MockKanbanColumn
- Existing handlers: auth.ts, tasks.ts, orders.ts

## Success Criteria

- [ ] 3 new types: MockOrderItem, MockWorkspace, MockDesignVersion
- [ ] Factory helpers cho ID generation
- [ ] Data volume: 6 users, 2 workspaces, 15 orders, 45 items, 45 tasks
- [ ] Vietnamese names/context trong data
- [ ] Handlers dùng imported data
- [ ] MSW hoạt động với data mới
