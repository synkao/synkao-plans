---
title: Order List UI Enhancements
description: Implement image lightbox, CSV import dialog, and note edit popover for order management
status: completed
priority: medium
effort: 8
branch: feat/order-list-ui-enhancements
tags: [orders, ui, features, v1.2]
created: 2026-01-25
completed: 2026-01-25T15:01:00Z
---

# Order List UI Enhancements

## Overview
- **Priority:** Medium
- **Status:** ✅ Completed
- **Completion:** 2026-01-25 15:01 UTC
- **Complexity:** Low-Medium
- **Files:** 3 new + 5 modified
- **Review Date:** 2026-01-25
- **Review Score:** 8.5/10

## Features

| # | Feature | New Files | Modified Files |
|---|---------|-----------|----------------|
| 1 | Image Lightbox | `image-lightbox-dialog.tsx` | `order-item-row.tsx`, `order-list-table.tsx` |
| 2 | CSV Import Dialog | `import-orders-dialog.tsx` | `orders-page-header.tsx` |
| 3 | Note Edit Popover | `note-edit-popover.tsx` | `order-row.tsx` |

## Phases

- ✅ [Phase 1: Image Lightbox](phase-01-image-lightbox.md) - Click thumbnail to view large image
- ✅ [Phase 2: CSV Import Dialog](phase-02-csv-import-dialog.md) - Drag-drop CSV upload UI
- ✅ [Phase 3: Note Edit Popover](phase-03-note-edit-popover.md) - Inline note editing with 255 char limit

## Dependencies

Existing components to reuse:
- `@/components/ui/dialog` - Radix Dialog
- `@/components/ui/popover` - Radix Popover
- `@/components/ui/file-uploader` - react-dropzone wrapper

Existing hooks:
- `useUpdateOrderNotes` - Already exists in `use-orders.ts`

## Success Criteria

- [x] Click product image opens fullscreen lightbox
- [x] Import button opens dialog with CSV dropzone
- [x] Note pencil icon opens popover with save/cancel
- [x] All components pass TypeScript compilation
- [x] UI consistent with existing design patterns

## Required Fixes Before Merge

1. **CRITICAL:** Upgrade Node.js to 20.x (blocks production build)
2. **HIGH:** Add input sanitization in `note-edit-popover.tsx`
3. **HIGH:** Fix CopyButton useEffect dependencies in `order-row.tsx`
4. **MEDIUM:** Remove console.log in `orders-page-header.tsx`

## Recommended Improvements

5. Remove/configure `unoptimized` in `image-lightbox-dialog.tsx`
6. Add file extension validation in `import-orders-dialog.tsx`
7. Document callback memoization in `OrderListTable` JSDoc
8. Add error boundaries around dialogs

## Code Review Report

See [reports/code-reviewer-260125-1456-phase-1-2-3-review.md](reports/code-reviewer-260125-1456-phase-1-2-3-review.md)

## Constraints

- YAGNI/KISS/DRY principles
- Files under 200 lines
- No API integration for CSV (UI only)
- Reuse existing component patterns
