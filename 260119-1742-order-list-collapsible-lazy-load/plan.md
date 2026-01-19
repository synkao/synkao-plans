---
title: "Order List Collapsible with Lazy Load Items"
description: "Convert order list to collapsible table with on-demand item fetching"
status: completed
priority: P2
effort: 2h
branch: main
tags: [orders, ui, performance, react-query]
created: 2026-01-19
reviewed: 2026-01-19
review_score: 8/10
---

# Order List Collapsible with Lazy Load Items

## Overview

Chuyển Order List table sang dạng collapsible rows với lazy loading items khi user expand mỗi order.

**Current:** Fetch tất cả orders + items cùng lúc (batch) → hiển thị tất cả items
**Target:** Fetch orders only → expand để fetch + hiển thị items của từng order

## Requirements

1. Cho phép nhiều orders expand cùng lúc
2. Hiển thị "No items" khi order không có items
3. Default state: tất cả orders collapsed
4. On expand: call API fetch items cho order đó

## Implementation Phases

| Phase | Description | Status | File |
|-------|-------------|--------|------|
| 01 | Update OrderRow - Add expand toggle | ✅ Completed | [phase-01](phase-01-order-row-expand-toggle.md) |
| 02 | Update OrderListTable - Expandable state & lazy fetch | ✅ Completed | [phase-02](phase-02-order-list-table-lazy-fetch.md) |
| 03 | Update OrdersPage - Use useOrders instead of useOrdersWithItems | ✅ Completed | [phase-03](phase-03-orders-page-simplify.md) |

## Architecture

```
OrdersPage
├── useOrders() ─────────────────────── Fetch orders only (no items)
└── OrderListTable
    ├── expandedIds: Set<string> ────── Track expanded orders
    └── OrderGroup (per order)
        ├── OrderRow
        │   └── ChevronRight/Down ───── Toggle expand
        └── {expanded && ...}
            ├── useOrderItems(orderId) ── Lazy fetch on expand
            ├── Loading state
            ├── Empty state ("No items")
            └── OrderItemRow (per item)
```

## Key Files

| File | Changes |
|------|---------|
| `features/orders/components/order-list/order-row.tsx` | Add expand button, expanded prop |
| `features/orders/components/order-list/order-list-table.tsx` | Add expandedIds state, lazy fetch in OrderGroup |
| `app/(main)/orders/page.tsx` | Change useOrdersWithItems → useOrders |

## Dependencies

- `useOrderItems` hook already exists in `use-orders.ts`
- React Query caches items per order
- ChevronRight/ChevronDown icons from lucide-react

## Success Criteria

- [x] Orders load without items initially
- [x] Click chevron expands/collapses order
- [x] Expanded order fetches and displays items
- [x] Loading state shown while fetching items
- [x] "No items" shown for empty orders
- [x] Multiple orders can be expanded simultaneously
- [x] React Query caches items (second expand is instant)

## Code Review Summary

**Review Date:** 2026-01-19
**Score:** 8/10
**Report:** [code-reviewer-260119-1749-order-list-collapsible-lazy-load-review.md](reports/code-reviewer-260119-1749-order-list-collapsible-lazy-load-review.md)

**Key Findings:**
- ✅ All phases completed successfully
- ✅ Clean lazy loading architecture with React Query
- ✅ Proper accessibility (ARIA labels, keyboard navigation)
- ✅ No security vulnerabilities detected
- ⚠️ Build verification blocked (Node.js version mismatch)
- ⚠️ Missing error handling in OrderGroup component
- ⚠️ orders/page.tsx exceeds 200-line guideline (252 lines)

**Recommended Actions:**
1. Upgrade Node.js to >=20.9.0
2. Add error state handling to OrderGroup
3. Modularize orders/page.tsx for better maintainability
