# Order List UI Enhancement - Implementation Plan

**Issue**: [#74](https://github.com/synkao/synkao/issues/74)
**Priority**: High (Customer request)
**Branch**: `feat/order-list-ui-enhancement`

## Overview

Enhance the `/orders` page to display more information inline, reducing clicks to view order details. Reference: PodOrder style.

## Phases

| Phase | Description | Status | Dependencies |
|-------|-------------|--------|--------------|
| 1 | [Mock Data Preparation](./phase-01-mock-data-preparation.md) | Complete | None |
| 2 | [UI Implementation](./phase-02-ui-implementation.md) | Complete | Phase 1 |

## Key Changes Summary

### Phase 1: Mock Data
- Add new fields to `MockOrder` type (storeName, revenue, fulfillDeadline)
- Create mock stores data
- Populate existing optional fields (fulfillerName, trackingNumber, carrier, notes)
- Add `/api/v1/orders/counts` endpoint for status tabs

### Phase 2: UI Implementation
- Restructure table columns (BUYER, COST, FF, NOTE)
- Add store badge to order row
- Implement inline status edit (popover)
- Add status tabs with counts
- Enhance item row (56px thumbnail, design tags)

## Success Criteria

- [x] All mock data fields populated correctly
- [x] New API endpoint returns status counts
- [x] Table displays all new columns
- [x] Inline status edit works without page navigation
- [x] Status tabs filter orders correctly
- [x] Item thumbnails display at 56x56
- [x] Design tags show only for items needing attention

## Latest Update (2026-01-25)

**Phase 2 Complete** - Final Score: 9.5/10

**All Critical & High Priority Issues Fixed**:
- C1: Added onStatusChange handler chain from page → table → row
- C2: XSS prevention via sanitizeForAttribute utility
- H1: Status transition validation with allowedTransitions map
- H3: useOrderCounts retry/placeholderData for graceful degradation
- H4: CopyButton memory leak fix with useRef cleanup
- H5: isUpdating loading state for StatusEditPopover

**Cosmetic Improvements**:
- M2: TABLE_COLUMN_WIDTHS constants extracted
- M6: Scroll fade indicator for mobile
- L5: Tooltip component for notes
- L6: Revenue label clarity ("Rev:" prefix)

## Files Impact Summary

### Phase 1 (Mock Data)
- `apps/web/src/mocks/types.ts` - Modify
- `apps/web/src/mocks/data/stores.data.ts` - Create
- `apps/web/src/mocks/data/orders.data.ts` - Modify
- `apps/web/src/mocks/data/index.ts` - Modify
- `apps/web/src/mocks/handlers/orders.ts` - Modify

### Phase 2 (UI)
- `apps/web/src/features/orders/components/order-list/order-list-table.tsx` - Modify
- `apps/web/src/features/orders/components/order-list/order-row.tsx` - Modify (major)
- `apps/web/src/features/orders/components/order-list/order-item-row.tsx` - Modify
- `apps/web/src/features/orders/components/order-list/status-edit-popover.tsx` - Create
- `apps/web/src/features/orders/components/order-list/order-status-tabs.tsx` - Create
- `apps/web/src/app/(main)/orders/orders-page-header.tsx` - Modify
- `apps/web/src/features/orders/hooks/use-order-counts.ts` - Create

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Table width overflow | Medium | Use responsive design, truncate text |
| Performance with counts | Low | Mock data only, no real API impact |
| Status edit conflicts | Low | Optimistic update with rollback |

## Notes

- No backend API changes required - all mock data
- Revenue calculation: `total - (subtotal * 0.3)` as placeholder
- Fulfill deadline: `createdAt + 3 days`
