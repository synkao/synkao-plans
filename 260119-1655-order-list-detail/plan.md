---
title: "Order List & Detail Implementation"
description: "Implement Order List (grouped rows) and Order Detail pages per spec 3.1/3.2"
status: completed
priority: P1
effort: 8h
branch: main
tags: [frontend, orders, react-query, msw]
created: 2026-01-19
reviewed: 2026-01-19
review_score: 7.5/10
---

# Order List & Detail Implementation Plan

## Overview

Implement Order List and Order Detail pages following **Approach B (Custom Grouped Rows)** based on `docs/frontend/synkao-frontend-spec.md` sections 3.1 and 3.2.

## Phases

| # | Phase | Status | Effort | Description |
|---|-------|--------|--------|-------------|
| 1 | [Types & Hooks](./phase-01-types-and-hooks.md) | ✅ completed | 1h | Order types, API client, React Query hooks |
| 2 | [Shared Components](./phase-02-shared-components.md) | ✅ completed | 1h | OrderStatusBadge, DesignPhaseBadge, FulfillerCell |
| 3 | [Order List Components](./phase-03-order-list-components.md) | ✅ completed | 2h | OrderRow, OrderItemRow, OrderListTable, Filters |
| 4 | [Order List Page](./phase-04-order-list-page.md) | ✅ completed | 1h | Page integration, state management |
| 5 | [Order Detail Components](./phase-05-order-detail-components.md) | ✅ completed | 1.5h | CustomerInfoCard, OrderStatusStepper, OrderItemCard, Timeline |
| 6 | [Order Detail Page](./phase-06-order-detail-page.md) | ✅ completed | 1h | Detail page integration, routing |
| 7 | [MSW Mock Enhancement](./phase-07-msw-mock-enhancement.md) | ✅ completed | 0.5h | Enhance orders handlers with timeline endpoint |

## Key Dependencies

- Existing: `DataTable`, `StatusBadge` patterns from `features/design/`
- Existing: MSW handlers for orders (`mocks/handlers/orders.ts`)
- Existing: Mock data (`mocks/data/orders.data.ts`, `mocks/data/order-items.data.ts`)
- Pattern: React Query hooks from `features/design/hooks/use-tasks.ts`

## File Structure

```
features/orders/
├── components/
│   ├── order-list/
│   │   ├── order-list-table.tsx
│   │   ├── order-row.tsx
│   │   ├── order-item-row.tsx
│   │   ├── order-list-filters.tsx
│   │   ├── bulk-action-bar.tsx
│   │   └── index.ts
│   ├── order-detail/
│   │   ├── customer-info-card.tsx
│   │   ├── order-status-stepper.tsx
│   │   ├── order-item-card.tsx
│   │   ├── order-timeline.tsx
│   │   └── index.ts
│   └── shared/
│       ├── order-status-badge.tsx
│       ├── design-phase-badge.tsx
│       ├── fulfiller-cell.tsx
│       └── index.ts
├── hooks/
│   ├── use-orders.ts
│   └── use-order-detail.ts
├── lib/
│   ├── orders-client.ts
│   └── constants.ts
├── types/
│   └── order.types.ts
└── index.ts
```

## Success Criteria

- [x] Order List displays grouped rows (order header + inline items)
- [x] Checkbox selection works at order level
- [x] Filters work: search, status, design, fulfiller, time range
- [x] Bulk action bar appears when orders selected
- [x] Server-side pagination functional
- [x] Order Detail shows customer info, status stepper, items, timeline
- [x] Navigation between list and detail works

## Code Review Results

**Date:** 2026-01-19
**Score:** 7.5/10
**Report:** [code-reviewer-260119-1720-order-implementation-review.md](./reports/code-reviewer-260119-1720-order-implementation-review.md)

### Critical Issues (FIXED ✅)
1. ~~**N+1 Query Problem**~~ → Added batch endpoint `POST /api/v1/orders/items/batch`
2. ~~**Missing Keyboard Navigation**~~ → Added onKeyDown, tabIndex, ARIA labels to OrderRow
3. ~~**Unsafe Type Assertion**~~ → Uses Map lookup for O(1) type-safe access

### High Priority
4. No error boundaries
5. Missing loading skeletons
6. Checkbox accessibility (indeterminate state)
7. Image optimization disabled
8. React Query cache invalidation missing
9. Timeline generation in page component (66 lines, SRP violation)
10. Memory leak risk - selectedOrderIds cleanup

### Recommended Next Steps (Future Improvements)
1. ~~Implement batch items endpoint~~ ✅ Done
2. ~~Add keyboard navigation~~ ✅ Done
3. Add error boundaries (error.tsx files)
4. Extract timeline generator to lib/
5. Add loading skeleton components
6. Implement mutation hooks with cache invalidation
