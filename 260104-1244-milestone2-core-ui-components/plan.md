---
title: "Milestone 2 Core UI Components"
description: "Implement Layout, Status Badges, and Overlay components for Milestone 2"
status: completed
priority: P0
effort: 5h
branch: main
tags: [frontend, ui, components, milestone-2]
created: 2026-01-04
completed: 2026-01-04
reviewed: 2026-01-04
---

# Milestone 2: Core UI Components

## Overview

Implementation plan cho 3 issues top priority trong Milestone 2:
- **#6** Layout Components (20% remaining)
- **#8** Status Badges (0% done)
- **#11** Overlay Components (50% remaining)

## Reference

- [Research Report](../reports/research-260104-1227-milestone2-top3-plan.md)
- [Milestone 2](https://github.com/synkao/synkao/milestone/2)

## Current State Analysis

| Component | Location | Status |
|-----------|----------|--------|
| GlassSidebar | `components/layout/` | ✅ Done |
| PageHeader | `components/layout/` | ✅ Done |
| AppHeader | `components/layout/` | ✅ Done |
| ContentArea | `components/layout/` | ✅ Done |
| PriorityBadge | `components/ui/status-badge.tsx` | ✅ Done |
| StatusBadge | `components/ui/status-badge.tsx` | ✅ Done |
| Dialog/Sheet | `components/ui/` | ✅ Done |
| Toast | `components/ui/sonner.tsx` | ✅ Done |
| ConfirmDialog | `components/ui/confirm-dialog.tsx` | ✅ Done |

## Implementation Phases

| Phase | Title | Effort | Status |
|-------|-------|--------|--------|
| [01](phase-01-toast-setup.md) | Toast Setup (Sonner) | 45 min | ✅ Done |
| [02](phase-02-status-badges.md) | Status Badges | 2h | ✅ Done |
| [03](phase-03-layout-wrappers.md) | Layout Wrappers | 1h | ✅ Done |
| [04](phase-04-confirm-dialog.md) | Confirm Dialog | 45 min | ✅ Done |
| [05](phase-05-test-page.md) | UI Test Page | 30 min | ✅ Done |

## Key Decisions

1. **Toast:** Use `sonner` - shadcn-compatible, popular, simple API
2. **Badges:** Create shared `components/ui/status-badge.tsx` using cva pattern
3. **Types:** Add shared types in `types/status.ts`
4. **Layout:** ContentArea minimal wrapper (YAGNI - skip AppShell unless needed)
5. **Existing code:** Keep `features/design/priority-badge.tsx` as-is, new shared badges independent

## Files Created/Modified

```
apps/web/src/
├── components/
│   ├── layout/
│   │   ├── content-area.tsx      # Phase 03 ✅
│   │   └── index.ts              # Modified: export ContentArea
│   └── ui/
│       ├── sonner.tsx            # Phase 01 ✅ (richColors enabled)
│       ├── status-badge.tsx      # Phase 02 ✅ (colors match design-system.md)
│       ├── alert-dialog.tsx      # Phase 04 ✅ (via shadcn)
│       └── confirm-dialog.tsx    # Phase 04 ✅ (text-white for destructive)
├── types/
│   ├── status.ts                 # Phase 02 ✅
│   └── index.ts                  # Modified: export status types
└── app/
    ├── layout.tsx                # Modified: add Toaster
    └── dev-tools/
        └── page.tsx              # Phase 05 ✅ (merged UI test)
```

## Dependencies

```bash
pnpm add sonner --filter web
pnpm dlx shadcn@latest add alert-dialog --cwd apps/web
```

## Success Criteria

- [x] Toast notifications work (4 variants) ✅
- [x] All status badges render correctly ✅
- [x] ConfirmDialog with destructive variant ✅
- [x] Layout exports updated ✅
- [x] Build passes, no TS errors ✅

## Code Review

**Status:** ✅ APPROVED
**Date:** 2026-01-04
**Report:** [code-reviewer-260104-1318-milestone2-core-ui.md](../reports/code-reviewer-260104-1318-milestone2-core-ui.md)

**Key Findings:**
- Quality Score: 8.5/10
- Security: ✅ CLEAN (0 vulnerabilities)
- Build: ✅ PASSED (TypeScript + production build)
- YAGNI/KISS/DRY: 100% compliant
- Critical Issues: 0
- Medium Issues: 2 (console.log cleanup, barrel exports)

**Follow-up Tasks:**
1. ~~Remove console.log từ ui-test page~~ ✅ Fixed
2. Create `components/ui/index.ts` barrel export (optional)
3. Add JSDoc comments (optional)

## Post-Review Changes

| Change | File | Description |
|--------|------|-------------|
| Toast colors | `sonner.tsx` | Added `richColors` prop for semantic colors |
| Delete button | `confirm-dialog.tsx` | Fixed `text-white` for destructive variant |
| Badge colors | `status-badge.tsx` | Updated all colors to match design-system.md |
| UI Test merge | `dev-tools/page.tsx` | Merged ui-test into main dev-tools page |

## Manual Testing Checklist

### Phase 01 - Toast
```bash
pnpm dev --filter web
```
1. Mở browser → `http://localhost:3000`
2. Mở DevTools Console, chạy:
   ```js
   // Import toast trong component hoặc test page
   toast.success("Test success")
   toast.error("Test error")
   toast.info("Test info")
   toast.warning("Test warning")
   ```
3. Verify: Toast xuất hiện bottom-right, đúng màu

### Phase 02 - Status Badges
1. Tạo test page hoặc dùng `/design-system` page
2. Render tất cả badge variants:
   - DesignStatusBadge: 5 statuses
   - PriorityBadge: 4 priorities
   - FulfillmentBadge: 5 statuses
   - SourceBadge: 3 sources
3. Verify: Màu đúng, text đúng

### Phase 03 - Layout
1. Mở `/orders` hoặc bất kỳ page nào
2. Verify: Padding consistent (p-6)
3. Test `noPadding` prop nếu cần

### Phase 04 - ConfirmDialog
1. Tạo button trigger dialog
2. Test:
   - Click Cancel → Dialog đóng, không gọi callback
   - Click Confirm → Dialog đóng, gọi callback
   - Destructive variant → Button màu đỏ
3. Verify: Focus trap hoạt động (Tab không escape)

### Phase 05 - UI Test (merged into dev-tools)
1. Navigate to `http://localhost:3000/dev-tools`
2. Click all Toast buttons → Verify toasts with semantic colors
3. Check all badge variants (colors from design-system.md)
4. Test ConfirmDialog default + destructive (white text)

### Final Verification
```bash
pnpm build --filter web
pnpm lint --filter web
```

## Risks

| Risk | Mitigation |
|------|------------|
| Duplicate badges | Keep feature-specific as-is, add shared independently |
| Sonner styling conflicts | Use shadcn-recommended config |
