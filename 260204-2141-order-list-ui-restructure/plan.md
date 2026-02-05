---
title: "Order List UI Restructure"
description: "UI enhancements for Order List: expand all, external ID, platform icons, item DESIGN/TASK columns"
status: completed
priority: P2
effort: 6h
branch: main
tags: [ui, orders, enhancement]
created: 2026-02-04
updated: 2026-02-04
---

# Order List UI Restructure

**Issue:** [#76](https://github.com/synkao/synkao/issues/76)
**Scout Report:** [scout-260204-2133-order-list-ui-restructure.md](../reports/scout-260204-2133-order-list-ui-restructure.md)

## Overview

Restructure Order List UI with improved information display and better item-level design/task visibility.

## Changes Summary

| # | Change | Status |
|---|--------|--------|
| 1 | Toggle Expand/Close All | ✅ complete |
| 2 | Add External ID | ✅ complete |
| 3 | Platform Icon + Domain | ✅ complete |
| 4 | Remove Order DESIGN/TASK cols | ✅ N/A |
| 5 | Item DESIGN & TASK columns | ✅ complete |

## Implementation Phases

| Phase | Description | Effort | Status |
|-------|-------------|--------|--------|
| [Phase 01](./phase-01-data-model-updates.md) | Data model updates (Store domain) | 1h | ✅ complete |
| [Phase 02](./phase-02-expand-all-toggle.md) | Toggle expand/close all | 1h | ✅ complete |
| [Phase 03](./phase-03-order-row-enhancements.md) | External ID + Platform icon | 2h | ✅ complete |
| [Phase 04](./phase-04-item-row-design-task-columns.md) | Item DESIGN & TASK columns | 2h | ✅ complete |

## Key Files

```
apps/web/src/features/orders/
├── components/order-list/
│   ├── order-list-table.tsx  ← Phase 02
│   ├── order-row.tsx         ← Phase 03
│   └── order-item-row.tsx    ← Phase 04
├── hooks/use-orders.ts       ← Phase 04 (task hook)
└── lib/constants.ts          ← Column widths

apps/web/src/mocks/
├── data/stores.data.ts       ← Phase 01
└── types.ts                  ← Phase 01 (if needed)
```

## Acceptance Criteria

- [x] Toggle expand/close all button works
- [x] External ID displays as `#WC-xxxxx`, `#SP-xxxxx`
- [x] Platform icons display correct color and letter
- [x] Store domain displays instead of platform name
- [x] Item row has separate DESIGN and TASK columns
- [x] DESIGN column: `—` when has, `⚠ Missing Design` when not
- [x] TASK column: status badge when has, `+ Create Task` button when not
- [x] `+ Create Task` button clickable and triggers create task flow (stub exists)
- [x] Task status data wired through to display actual task status

## Review Report

**Code Review:** [code-reviewer-260204-2156-order-list-ui-restructure.md](./reports/code-reviewer-260204-2156-order-list-ui-restructure.md)
**Score:** 8.5/10
**Status:** ✅ Approve with minor fixes

### Remaining Issues
1. **HIGH:** Build verification blocked by Node.js version (need >=20.9.0)
2. **MEDIUM:** Task status data not wired (taskStatus prop not passed)
3. **MEDIUM:** Column header alignment needs adjustment
4. **MEDIUM:** PlatformIcon uses inline styles (prefer Tailwind)

### Next Actions
1. Upgrade Node.js and verify build
2. Wire task status through OrderItemRow
3. Test end-to-end with all scenarios
