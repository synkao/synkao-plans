---
title: "Design Area Tabs Refactor & Image Preview"
description: "Filter tabs to designAreas map, add image preview on click, seed all items with designAreas"
status: completed
priority: P2
effort: 1.5h
branch: main
tags: [orders, design-areas, ui, refactor]
created: 2026-02-09
completed: 2026-02-09
---

# Design Area Tabs Refactor & Image Preview

## Goal
1. All order items have `designAreas` defined (no item without it)
2. DesignAreaTabs only shows positions present in `designAreas` map (remove gray inactive tabs)
3. Tab with `fileInfo=null` → DesignUploadModal (upload allowed)
4. Tab with `fileInfo=DesignFileInfo` → tooltip + click to preview via ImageLightboxDialog
5. Remove inactive tab logic entirely

## Phases

| # | Phase | Status | Effort | File |
|---|-------|--------|--------|------|
| 1 | Mock data: seed all items with designAreas | ✅ completed | 30m | [phase-01](phase-01-seed-design-areas-mock-data.md) |
| 2 | Refactor DesignAreaTabs: filter + upload + preview | ✅ completed | 45m | [phase-02](phase-02-refactor-design-area-tabs.md) |
| 3 | Code review | ✅ completed | 15m | [reports/code-reviewer-260209-1343-design-tabs-refactor.md](reports/code-reviewer-260209-1343-design-tabs-refactor.md) |

## Key Files
- `apps/web/src/mocks/data/order-items.data.ts` — mock data
- `apps/web/src/features/orders/components/order-detail/design-area-tabs.tsx` — main component
- `apps/web/src/features/orders/components/order-list/image-lightbox-dialog.tsx` — reuse for preview
- `apps/web/src/mocks/types.ts` — types (no change needed)

## Dependencies
- ImageLightboxDialog already exists in `order-list/`
- DesignUploadModal already accepts `position` prop
- DesignFileInfo type has `url` field for preview

## Architecture Decision
- **Approach 1 (chosen):** Filter `DESIGN_POSITION_OPTIONS` by `designAreas` keys. Reuse `ImageLightboxDialog` for preview.
- Rationale: KISS, ~20 lines changed, reuses existing components

## Review Summary
**Score:** 8.5/10 | **Status:** Approved with minor fixes

**Strengths:** Clean refactor, YAGNI/KISS/DRY compliant, reuses components, under 200 LOC
**Issues:** Build blocked (Node 18 vs 20+), upload state update missing, non-image file handling

**Critical Actions:**
1. Update Node.js to 20+ and verify build
2. Fix mock image URLs (use picsum.photos)
3. Add file type check before preview
4. Implement upload success state update

See [detailed review report](reports/code-reviewer-260209-1343-design-tabs-refactor.md)
