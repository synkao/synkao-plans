---
title: "Design Area Tabs & Create Task Modal"
description: "Add design area tabs per order item and create task (Design Service) modal for order detail page"
status: completed
priority: P2
effort: 4h
branch: main
tags: [orders, design, ui, modal]
created: 2026-02-08
reviewed: 2026-02-08
---

# Design Area Tabs & Create Task Modal

## Summary

Add two new UI features to the Order Detail page:
1. **Design Area Tabs** - horizontal tab row per order item showing design positions (Front, Back, etc.)
2. **Create Task Modal** - "Design Service" dialog for creating design tasks from order items

Based on reference screenshots showing tab-based design area selection and a task creation form.

## Phases

| # | Phase | Status | Effort | File |
|---|-------|--------|--------|------|
| 1 | Data Model & Mock Updates | ✅ complete | 0.5h | [phase-01](./phase-01-data-model-mock-updates.md) |
| 2 | Design Area Tabs Component | ✅ complete | 1h | [phase-02](./phase-02-design-area-tabs-component.md) |
| 3 | Create Task Modal Component | ⚠️ complete (validation needed) | 1.5h | [phase-03](./phase-03-create-task-modal-component.md) |
| 4 | OrderItemCard Integration & Modularization | ✅ complete | 1h | [phase-04-integration](./phase-04-order-item-card-integration.md) |

## Key Dependencies

- Existing: `DESIGN_POSITION_OPTIONS` (11 positions) in `constants.ts`
- Existing: `DesignUploadModal` component for upload-on-tab-click
- Existing: `MockOrderItem` type in `mocks/types.ts`
- Existing: `DatePicker` component in `components/ui/date-picker.tsx`
- Existing: shadcn Dialog, Input, Textarea, Button components

## Key Decisions

- `designAreas` field uses `Partial<Record<DesignPositionType, DesignFileInfo | null>>` - null = active but no file
- Tab click on inactive position opens `DesignUploadModal` with position context
- Active tabs shown with blue styling, inactive with gray outline
- Create Task modal title "Design Service", pre-fills from order/item data
- `OrderItemCard` modularization needed (324 lines > 200 line limit)

## Reference Screenshots

- `/tmp/comment1.png` - Overview with tabs + upload modal with position
- `/tmp/comment2.png` - Full view with Design Service modal, 3 item cards with tabs

## Code Review

**Review Date:** 2026-02-08
**Score:** 8.5/10
**Status:** Approved with minor fixes

See full report: [reports/code-reviewer-260208-1207-final-review.md](./reports/code-reviewer-260208-1207-final-review.md)

**Summary:**
- ✅ All files under 200 lines
- ✅ TypeScript strict, no `any` types
- ✅ Accessibility (18 ARIA attributes)
- ✅ No security issues
- ⚠️ Missing input validation in CreateTaskModal
- ⚠️ Minor performance optimization needed (modal rendering)

**Recommended Actions:**
1. Add title/message validation
2. Add due date past-date check
3. Fix tooltip pointer-events on disabled button

## Architecture

```
OrderDetailPage
  └── OrderItemCard (modularized)
        ├── DesignAreaTabs (new)
        │     └── DesignUploadModal (existing, updated with position)
        └── CreateTaskModal (new)
              └── Dialog with Title, Message, Image, DatePicker
```
