# Review: Plan vs Design Specs

**Date:** 2026-01-02
**Reviewed by:** Claude
**Plan:** `260102-0716-synkao-mock-demo`

---

## Summary

| Category | Status | Notes |
|----------|--------|-------|
| Routes | ✅ Aligned | Tất cả routes match |
| Layout/Shell | ✅ Aligned | GlassSidebar, AppHeader specs đúng |
| Design System | ⚠️ Minor gaps | Cần bổ sung một số colors/tokens |
| Components | ⚠️ Missing some | Một số components trong spec chưa có trong plan |
| Pages | ✅ Aligned | 12 pages đúng với plan |

---

## 1. Route Structure Comparison

### Plan Routes
```
(auth)/login, (auth)/forgot-password
(main)/ (dashboard)
(main)/orders, (main)/orders/[id], (main)/orders/import
(main)/design/backlog, (main)/design/workspace, (main)/design/workspace/[id]
(main)/settings/team, (main)/settings/workspaces
```

### Spec Routes
```
/login, /forgot-password, /reset-password
/dashboard
/orders, /orders/:id, /orders/import, /orders/export
/design/backlog, /design/workspace, /design/workspace/:id, /design/task/:id
/fulfillment, /fulfillment/:id (Phase 2)
/settings/profile, /settings/team, /settings/workspaces, /settings/stores
```

### Gaps
| Route | In Plan | In Spec | Action |
|-------|---------|---------|--------|
| `/reset-password` | ❌ | ✅ | Skip - static demo |
| `/orders/export` | ❌ | ✅ | Skip - static demo |
| `/design/task/:id` | Drawer | Route | OK - using drawer |
| `/fulfillment/*` | ❌ | ✅ (Phase 2) | Skip - out of scope |
| `/settings/profile` | ❌ | ✅ | Skip - static demo |
| `/settings/stores` | ❌ | ✅ | Skip - static demo |

**Verdict:** ✅ Tất cả routes cần thiết cho static demo đều có trong plan.

---

## 2. Layout/Shell Comparison

### Sidebar Specs (from design-system.md)
| Property | Spec Value | Plan Value | Match |
|----------|------------|------------|-------|
| Width expanded | 240px | 240px | ✅ |
| Width collapsed | 64px | 64px | ✅ |
| Background | `rgba(255,255,255,0.6)` | `bg-white/60` | ✅ |
| Backdrop blur | 12px | `backdrop-blur-md` (12px) | ✅ |
| Border | `rgba(226,232,240,0.8)` | `border-slate-200/80` | ✅ |

### Header Specs
| Property | Spec Value | Plan Value | Match |
|----------|------------|------------|-------|
| Height | 64px | 64px | ✅ |
| Background | `#FFFFFF` | white | ✅ |
| Position | Sticky top | sticky | ✅ |

**Verdict:** ✅ Layout specs match.

---

## 3. Design System Gaps

### Colors to Add to tailwind.config.ts

**Phase Colors (from spec section 3):**
```typescript
phase: {
  order: '#64748B',      // Slate 500
  design: '#F59E0B',     // Amber 500
  fulfill: '#8B5CF6',    // Violet 500
  complete: '#10B981',   // Emerald 500
  cancelled: '#6B7280',  // Gray 500
}
```

**Plan có:** Glass colors
**Plan thiếu:** Phase colors (order, design, fulfill, complete, cancelled)

### Status Colors (from spec section 4)
Plan đã có trong `constants.ts`:
- TODO: blue ✅
- IN_PROGRESS: amber ✅
- REVIEW: violet ✅
- REVISION: red ✅
- DONE/APPROVED: emerald ✅

**Verdict:** ⚠️ Cần bổ sung phase colors vào Phase 1.

---

## 4. Components Comparison

### Components in Spec but Missing in Plan

| Component | Spec Section | Needed? | Action |
|-----------|--------------|---------|--------|
| `NotificationBell` | 3.3 Header | ⚠️ Static demo | Add placeholder |
| `FilterBar` (orders) | 6.1 | ✅ | Add to Phase 3 |
| `Pagination` | 6.1 | ✅ | Add to Phase 3 |
| `DesignerFilter` | 7.2 Kanban | ✅ | Add to Phase 2 |
| `WIPLimit` indicator | 7.2 Kanban | ❌ | Skip - static demo |
| `DesignPreviewModal` | 7.3 Drawer | ⚠️ Nice to have | Optional |

### Components Plan Has Extra (Good)

| Component | Plan Phase | Notes |
|-----------|------------|-------|
| `SortableTaskCard` | Phase 2 | dnd-kit specific |
| `DragOverlayCard` | Phase 2 | dnd-kit specific |
| `useKanbanState` | Phase 2 | Local state hook |

**Verdict:** ⚠️ Cần bổ sung một số components.

---

## 5. Page-Level Feature Comparison

### Dashboard (Phase 4)
| Feature (Spec) | In Plan | Status |
|----------------|---------|--------|
| 4 Stat cards | ✅ | Match |
| Trend indicator | ✅ | Match |
| Pending tasks list | ✅ | Match |
| Recent orders list | ✅ | Match |
| Activity feed | ✅ | Match |
| Date filter dropdown | ❌ | Skip - static demo |

### Orders List (Phase 3)
| Feature (Spec) | In Plan | Status |
|----------------|---------|--------|
| Expandable rows | ✅ | Match |
| Tab bar filter | ✅ | Match |
| Filter bar | ⚠️ | Need to add |
| Bulk action bar | ✅ | Match |
| Pagination | ⚠️ | Need to add |

### Kanban Board (Phase 2)
| Feature (Spec) | In Plan | Status |
|----------------|---------|--------|
| 5 columns | ✅ | Match |
| Drag-drop | ✅ | Match (dnd-kit) |
| Task cards | ✅ | Match |
| Designer filter | ⚠️ | Need to add |
| Column WIP limit | ❌ | Skip - static demo |

### Task Drawer (Phase 2)
| Feature (Spec) | In Plan | Status |
|----------------|---------|--------|
| Task info section | ✅ | Match |
| Design notes | ✅ | Match |
| Design versions | ✅ | Match |
| Timeline | ✅ | Match |
| Upload area | ❌ | Skip - static demo |
| Review actions | ❌ | Skip - static demo |

---

## 6. Typography & Fonts

### Spec
- Heading: `Be Vietnam Pro`
- Body: `Inter`
- Mono: `JetBrains Mono`

### Plan (Phase 1)
- Mentions adding Inter + JetBrains Mono
- **Gap:** Be Vietnam Pro not mentioned

**Action:** Update Phase 1 to include Be Vietnam Pro font.

---

## 7. Recommended Updates

### Phase 1 Updates
1. ✅ ~~Add `Be Vietnam Pro` font to root layout~~ - DONE
2. ✅ ~~Add phase colors to tailwind.config.ts~~ - DONE

### Phase 2 Updates
1. ✅ ~~Add `DesignerFilter` component to Kanban~~ - DONE
2. ✅ ~~Add assignee filter functionality (local state)~~ - DONE

### Phase 3 Updates
1. ✅ ~~Add `OrdersFilterBar` component~~ - DONE
2. ✅ ~~Add `Pagination` component (static, no API)~~ - DONE

### Phase 4 Updates
1. ✅ No changes needed

### Phase 5 Updates
1. ✅ No changes needed

---

## 8. Validation Checklist

| Check | Status |
|-------|--------|
| All 12 demo pages covered | ✅ |
| Glass morphism specs match | ✅ |
| Color system aligned | ✅ Phase colors added |
| Kanban columns match | ✅ |
| Task statuses match | ✅ |
| Priority colors match | ✅ |
| Typography defined | ✅ Be Vietnam Pro added |
| Component inventory sufficient | ✅ Filter components added |

---

## 9. Conclusion

Plan is now **100% aligned** với design specs. Tất cả gaps đã được fix:

1. ✅ **Font:** Be Vietnam Pro đã được thêm vào Phase 1
2. ✅ **Colors:** Phase colors đã được thêm vào tailwind.config.ts
3. ✅ **Components:** DesignerFilter, OrdersFilterBar, Pagination đã được thêm
4. **Can skip:** Real-time features, upload, review actions (static demo)

**Recommendation:** Ready to implement! 🚀
