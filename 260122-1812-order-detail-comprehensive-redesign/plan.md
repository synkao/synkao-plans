# Order Detail Page Comprehensive Redesign

**Issue:** [#73](https://github.com/synkao/synkao/issues/73)
**Created:** 2026-01-22
**Status:** ✅ Complete (All Phases Done)
**Last Updated:** 2026-01-22 20:20

## Overview

Comprehensive redesign của trang Order Detail (`/orders/:id`) với:
- Enhanced product cards với attributes, file info, action buttons
- Order summary card với financial breakdown
- Rich timeline với order-specific events
- Header actions: Cancel Order, Edit Notes
- Billing vs Shipping addresses tách biệt

## Phases

| Phase | Description | Status | Score |
|-------|-------------|--------|-------|
| [Phase 01](phase-01-data-layer-types.md) | Data Layer - Types & Mock Data | ✅ Complete | 8.5/10 |
| [Phase 02](phase-02-api-endpoints.md) | API Layer - New Endpoints | ✅ Complete | 8.0/10 |
| [Phase 03](phase-03-order-summary-card.md) | UI - Order Summary Card | ✅ Complete | 8.5/10 |
| [Phase 04](phase-04-product-cards-enhancement.md) | UI - Product Cards Enhancement | ✅ Complete | 8.5/10 |
| [Phase 05](phase-05-timeline-enhancement.md) | UI - Timeline Enhancement | ✅ Complete | 9/10 |
| [Phase 06](phase-06-header-actions.md) | UI - Header Actions & Dialogs | ✅ Complete | 9/10 |
| [Phase 07](phase-07-customer-info-enhancement.md) | UI - Customer Info Enhancement | ✅ Complete | 9/10 |

## Key Decisions

1. **Order timeline**: Separate order events (không aggregate từ tasks)
2. **Payment fields**: External readonly (từ WooCommerce/Shopify)
3. **Attributes format**: Key-value pairs `{ key, value }`

## Architecture

```
/orders/:id (page.tsx)
├── PageHeader
│   ├── Back button
│   ├── Create Fulfillment button
│   ├── Edit Notes button (NEW)
│   └── Cancel Order button (NEW)
│
├── Left Column (2/3)
│   ├── CustomerInfoCard (ENHANCED: billing/shipping)
│   ├── OrderNotesSection (NEW)
│   └── Items Section
│       └── OrderItemCard[] (ENHANCED: attributes, actions)
│
└── Right Column (1/3)
    ├── OrderMetadataCard (NEW: timestamps, update button)
    ├── OrderStatusStepper (existing)
    ├── OrderSummaryCard (NEW: financial breakdown)
    └── OrderTimeline (ENHANCED: rich events)
```

## Acceptance Criteria (12 items)

- [x] Design status badges visible for each product ✅
- [x] Product cards display thumbnail, name, SKU, quantity, status, file info ✅
- [x] Action buttons (View, Download, Create Task, Upload, Replace) present ✅
- [x] Order Summary card with complete financial breakdown ✅
- [x] Order metadata with timestamps and Update button ✅
- [x] Status progress indicator shows current step ✅
- [x] Timeline displays 6+ event types with user attribution ✅
- [x] Order Notes section visible with edit capability ✅
- [x] Cancel Order button available in header ✅
- [x] File info (size, upload date) in product cards ✅
- [x] Color-coded badges and borders for status indicators ✅
- [x] Section labels with count indicators ✅

## Dependencies

- shadcn/ui components: Card, Button, Dialog, AlertDialog, Badge
- Existing constants: ORDER_STATUS_CONFIG, ITEM_DESIGN_STATUS_CONFIG
- MSW mock handlers for API endpoints

## Risk Assessment

| Risk | Mitigation |
|------|------------|
| Data model changes affect other components | Review all usages of MockOrder, MockOrderItem |
| Large scope (12 criteria) | Split into focused phases |
| Mobile responsive requirement | Test each phase on mobile breakpoints |
