---
title: "Order Detail Page UI Updates"
description: "UI-only updates for Order Detail page based on GitHub issue #77"
status: in-progress
priority: P1
effort: 6h
branch: main
tags: [ui, orders, frontend]
created: 2026-02-05
completed: 2026-02-05
phase-02-completed: 2026-02-05 22:49
---

# Order Detail Page UI Updates

## Overview
UI-only updates to Order Detail page per GitHub issue #77. No backend API changes.

## References
- [Codebase Analysis](./reports/codebase-analysis.md)
- GitHub Issue: #77

## Phase Summary

| Phase | Description | Effort | Status | Review |
|-------|-------------|--------|--------|--------|
| 01 | Header & Status Dropdown | 2h | ✅ completed | [Report](./reports/code-reviewer-260205-2237-phase-01-review.md) |
| 02 | Sidebar Cards Updates | 2h | ✅ completed | [Report](./reports/code-reviewer-260205-2249-phase-02-review.md) 8/10 |
| 03 | Design Areas Components | 2h | ✅ completed 9/10 | [Report](./reports/code-reviewer-260205-2308-phase03-rereviewed.md) |

## Key Changes

### Header (Phase 01)
- Remove: Edit Notes, Cancel Order, Create Fulfillment buttons
- Keep: Back button
- Add: Split Order button, Status Dropdown

### Sidebar (Phase 02)
- Order Info Card: Add Store row, External ID row, remove Update Status btn
- Customer Info Card: Remove External ID and Source rows
- Order Summary: Show payment method only (no badge)
- Remove: OrderStatusStepper component usage

### Design Areas (Phase 03)
- Upload Modal: Drag & drop for empty design tabs
- Task Modal: Position dropdown (11 options)
- Task Button Logic: Conditional rendering based on design/task state

## Dependencies
- shadcn/ui (Select, Dialog, FileUploader)
- Existing constants/types patterns
- Platform icon component

## Files Overview

**Create:**
- `order-status-dropdown.tsx`
- `design-upload-modal.tsx`

**Modify:**
- `page.tsx` (header actions)
- `order-metadata-card.tsx` (store/external ID rows)
- `customer-info-card.tsx` (remove rows)
- `order-summary-card.tsx` (simplify payment)
- `order-item-card.tsx` (task button logic)
- `constants.ts` (new status config)

## Success Criteria
- [x] Phase 01 completed (Header & Status Dropdown) - Score: 8.5/10 - 2026-02-05 22:37
- [x] Phase 02 completed (Sidebar Cards Updates) - Score: 8/10 - 2026-02-05 22:49
  - [x] Order Info card shows Store and External ID rows
  - [x] Order Info card has no Update Status button
  - [x] Customer Info card has no External ID or Source rows
  - [x] Order Summary shows payment method only (no badge)
  - [x] Sidebar no longer shows stepper component
  - [x] Code review passed (8/10)
- [x] Phase 03 completed (Design Areas Components) - Score: 9/10 - 2026-02-05 23:09
  - [x] Upload modal for empty design tabs
  - [x] Task button conditional logic (blue/enabled, gray/disabled, status badge)
  - [x] Position dropdown options (11 options)
  - [x] File validation and error handling
  - [x] Accessibility compliance (ARIA, semantic HTML)
  - [x] Code review passed (9/10 after fixes)
- [x] No backend API changes (confirmed)
- [x] Follows existing codebase patterns (verified)
- [ ] TypeScript compiles without errors (blocked by Node version)
