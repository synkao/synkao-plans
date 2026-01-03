# Wireframe Alignment Report

**Date:** 2026-01-02
**Spec Source:** `docs/design/synkao-page-ui-spec.md`
**Plan:** `plans/260102-0716-synkao-mock-demo/`

---

## Executive Summary

| Category | Alignment | Notes |
|----------|-----------|-------|
| Routes/Pages | ✅ 100% | All 12 demo pages covered |
| Layout/Shell | ✅ 100% | GlassSidebar, Header specs match |
| Design System | ✅ 100% | Colors, fonts, glass effects aligned |
| Components | ✅ 95% | Minor gaps for static demo |
| Wireframes | ✅ 90% | Core UI elements covered |

**Overall Status:** Plan đã aligned với wireframe spec. Ready to implement!

---

## 1. Page Coverage

### Spec Routes vs Plan Routes

| Page | Spec Route | Plan Route | Status |
|------|------------|------------|--------|
| Login | `/login` | `(auth)/login` | ✅ Phase 5 |
| Forgot Password | `/forgot-password` | `(auth)/forgot-password` | ✅ Phase 5 |
| Reset Password | `/reset-password` | — | ⏭️ Skip (static) |
| Dashboard | `/dashboard` | `(main)/` | ✅ Phase 4 |
| Orders List | `/orders` | `(main)/orders` | ✅ Phase 3 |
| Order Detail | `/orders/:id` | `(main)/orders/[id]` | ✅ Phase 3 |
| Import Wizard | `/orders/import` | `(main)/orders/import` | ✅ Phase 3 |
| Export | `/orders/export` | — | ⏭️ Skip (static) |
| Design Backlog | `/design/backlog` | `(main)/design/backlog` | ✅ Phase 2 |
| Workspace List | `/design/workspace` | `(main)/design/workspace` | ✅ Phase 2 |
| Kanban Board | `/design/workspace/:id` | `(main)/design/workspace/[id]` | ✅ Phase 2 |
| Task Drawer | `/design/task/:id` | Sheet overlay | ✅ Phase 2 |
| Fulfillment | `/fulfillment/*` | — | ⏭️ Out of scope |
| Profile Settings | `/settings/profile` | — | ⏭️ Skip (static) |
| Team Settings | `/settings/team` | `(main)/settings/team` | ✅ Phase 4 |
| Workspaces | `/settings/workspaces` | `(main)/settings/workspaces` | ✅ Phase 4 |
| Store Settings | `/settings/stores` | — | ⏭️ Skip (static) |

**Result:** 12/12 required pages covered, 5 routes intentionally skipped

---

## 2. Layout/Shell Alignment

### Glass Sidebar (Spec Section 3.2)

| Property | Spec | Plan | Match |
|----------|------|------|-------|
| Width Expanded | 240px | 240px | ✅ |
| Width Collapsed | 64px | 64px | ✅ |
| Background | `rgba(255,255,255,0.6)` | `bg-white/60` | ✅ |
| Backdrop Blur | 12px | `backdrop-blur-md` | ✅ |
| Border | `rgba(226,232,240,0.8)` | `border-slate-200/80` | ✅ |
| Position | Fixed left | Fixed left | ✅ |
| Z-index | 40 | 40 | ✅ |

### Header Bar (Spec Section 3.3)

| Property | Spec | Plan | Match |
|----------|------|------|-------|
| Height | 64px | 64px | ✅ |
| Background | `#FFFFFF` | white | ✅ |
| Border | `1px solid #E2E8F0` | `border-slate-200` | ✅ |
| Position | Sticky top | sticky | ✅ |

### Sidebar Elements (Spec Section 3.2)

| Element | Spec | Plan | Status |
|---------|------|------|--------|
| Logo area (56px) | ✅ | logo.tsx | ✅ |
| Nav items | ✅ | sidebar-nav-items.tsx | ✅ |
| Expandable groups | ✅ | Design submenu | ✅ |
| Badge counts | ✅ | navigation.ts badges | ✅ |
| User info footer | ✅ | user-menu.tsx | ✅ |
| Active/hover states | ✅ | CSS classes | ✅ |

---

## 3. Typography Alignment

| Font | Spec | Plan | Status |
|------|------|------|--------|
| Heading | Be Vietnam Pro | Be Vietnam Pro | ✅ Phase 1 |
| Body | Inter | Inter | ✅ Phase 1 |
| Mono | JetBrains Mono | JetBrains Mono | ✅ Phase 1 |

---

## 4. Color System Alignment

### Phase Colors (Spec Section 3)

| Phase | Spec Color | Plan | Status |
|-------|------------|------|--------|
| Order | Slate 500 `#64748B` | `phase.order` | ✅ Phase 1 |
| Design | Amber 500 `#F59E0B` | `phase.design` | ✅ Phase 1 |
| Fulfill | Violet 500 `#8B5CF6` | `phase.fulfill` | ✅ Phase 1 |
| Complete | Emerald 500 `#10B981` | `phase.complete` | ✅ Phase 1 |
| Cancelled | Gray 500 `#6B7280` | `phase.cancelled` | ✅ Phase 1 |

### Status Colors (Spec Section 7.2)

| Status | Spec | Plan | Status |
|--------|------|------|--------|
| TODO | Blue 500 | `bg-blue-50/50` | ✅ Phase 2 |
| IN_PROGRESS | Amber 500 | `bg-amber-50/50` | ✅ Phase 2 |
| REVIEW | Violet 500 | `bg-violet-50/50` | ✅ Phase 2 |
| REVISION | Red 500 | `bg-red-50/50` | ✅ Phase 2 |
| DONE/APPROVED | Emerald 500 | `bg-emerald-50/50` | ✅ Phase 2 |

### Priority Colors (Spec Section 7.1)

| Priority | Spec | Plan | Status |
|----------|------|------|--------|
| URGENT | Red `#EF4444` | PRIORITY_CONFIG | ✅ Phase 2 |
| HIGH | Amber `#F59E0B` | PRIORITY_CONFIG | ✅ Phase 2 |
| NORMAL | Blue `#3B82F6` | PRIORITY_CONFIG | ✅ Phase 2 |
| LOW | Gray `#6B7280` | PRIORITY_CONFIG | ✅ Phase 2 |

---

## 5. Wireframe Element Coverage

### Dashboard (Spec Section 5.1)

| Element | Spec Wireframe | Plan | Status |
|---------|----------------|------|--------|
| 4 Stat Cards | ✅ | StatsGrid | ✅ Phase 4 |
| Trend indicators | ✅ | TrendBadge | ✅ Phase 4 |
| Pending tasks list | ✅ | PendingTasksCard | ✅ Phase 4 |
| Recent orders list | ✅ | RecentOrdersCard | ✅ Phase 4 |
| Activity feed | ✅ | ActivityFeed | ✅ Phase 4 |
| Date filter | ✅ | — | ⏭️ Skip (static) |

### Orders List (Spec Section 6.1)

| Element | Spec Wireframe | Plan | Status |
|---------|----------------|------|--------|
| Filter bar | ✅ | OrdersFilterBar | ✅ Phase 3 |
| Tab bar (status) | ✅ | OrderTabBar | ✅ Phase 3 |
| Expandable table | ✅ | OrdersTable + Collapsible | ✅ Phase 3 |
| Nested item rows | ✅ | OrderItemRow | ✅ Phase 3 |
| Bulk action bar | ✅ | BulkActionBar | ✅ Phase 3 |
| Pagination | ✅ | OrdersPagination | ✅ Phase 3 |
| Import/Export buttons | ✅ | PageHeader actions | ✅ Phase 3 |

### Order Detail (Spec Section 6.2)

| Element | Spec Wireframe | Plan | Status |
|---------|----------------|------|--------|
| Customer info card | ✅ | CustomerInfoCard | ✅ Phase 3 |
| Status stepper | ✅ | OrderStatusStepper | ✅ Phase 3 |
| Items list | ✅ | OrderItemsCard | ✅ Phase 3 |
| Order summary | ✅ | OrderSummaryCard | ✅ Phase 3 |
| Timeline | ✅ | OrderTimeline | ✅ Phase 3 |

### Import Wizard (Spec Section 6.3)

| Element | Spec Wireframe | Plan | Status |
|---------|----------------|------|--------|
| Step indicator | ✅ | ImportStepper | ✅ Phase 3 |
| File dropzone | ✅ | FileDropzone | ✅ Phase 3 |
| Preview table | ✅ | ImportPreviewTable | ✅ Phase 3 |
| Validation summary | ✅ | ImportValidationSummary | ✅ Phase 3 |
| Result summary | ✅ | ImportResult | ✅ Phase 3 |

### Design Backlog (Spec Section 7.1)

| Element | Spec Wireframe | Plan | Status |
|---------|----------------|------|--------|
| Filter bar | ✅ | BacklogFilterBar | ✅ Phase 2 |
| Stats bar | ✅ | BacklogStatsBar | ✅ Phase 2 |
| Data table | ✅ | BacklogTable | ✅ Phase 2 |
| Priority badges | ✅ | PriorityBadge | ✅ Phase 2 |
| Source badges | ✅ | SourceBadge | ⚠️ Need to add |
| Bulk action bar | ✅ | — | ⏭️ Skip (static) |

### Kanban Board (Spec Section 7.2)

| Element | Spec Wireframe | Plan | Status |
|---------|----------------|------|--------|
| 5 columns | ✅ | KanbanColumn x5 | ✅ Phase 2 |
| Task cards | ✅ | TaskCard | ✅ Phase 2 |
| Drag & drop | ✅ | dnd-kit | ✅ Phase 2 |
| Designer filter | ✅ | DesignerFilter | ✅ Phase 2 |
| WIP limits | ✅ | — | ⏭️ Skip (static) |
| Column highlight on drag | ✅ | isOver ring effect | ✅ Phase 2 |

### Task Card (Spec Section 7.2)

| Element | Spec Wireframe | Plan | Status |
|---------|----------------|------|--------|
| Thumbnail | ✅ | TaskCard | ✅ Phase 2 |
| Order number | ✅ | TaskCard | ✅ Phase 2 |
| Item name | ✅ | TaskCard | ✅ Phase 2 |
| Customer name | ✅ | TaskCard | ✅ Phase 2 |
| Priority badge | ✅ | PriorityBadge | ✅ Phase 2 |
| Assignee avatar | ✅ | Avatar | ✅ Phase 2 |
| Age indicator | ✅ | relative time | ✅ Phase 2 |

### Task Drawer (Spec Section 7.3)

| Element | Spec Wireframe | Plan | Status |
|---------|----------------|------|--------|
| Order/Item header | ✅ | TaskHeader | ✅ Phase 2 |
| Task info section | ✅ | TaskInfoSection | ✅ Phase 2 |
| Design notes | ✅ | DesignNotesSection | ✅ Phase 2 |
| Design versions | ✅ | DesignVersionsSection | ✅ Phase 2 |
| Timeline | ✅ | TimelineSection | ✅ Phase 2 |
| Upload area | ✅ | — | ⏭️ Skip (static) |
| Review actions | ✅ | — | ⏭️ Skip (static) |
| Status dropdown | ✅ | StatusDropdown | ✅ Phase 2 |

### Team Settings (Spec Section 8.1)

| Element | Spec Wireframe | Plan | Status |
|---------|----------------|------|--------|
| Search/filter | ✅ | FilterBar | ✅ Phase 4 |
| User table | ✅ | TeamTable | ✅ Phase 4 |
| Role badges | ✅ | RoleBadge | ✅ Phase 4 |
| Status badges | ✅ | StatusBadge | ✅ Phase 4 |
| Invite modal | ✅ | InviteModal | ✅ Phase 4 |

### Workspace Settings (Spec Section 8.2)

| Element | Spec Wireframe | Plan | Status |
|---------|----------------|------|--------|
| Workspace cards | ✅ | WorkspaceSettingsCard | ✅ Phase 4 |
| Create card | ✅ | CreateWorkspaceCard | ✅ Phase 4 |
| Create/Edit modal | ✅ | WorkspaceModal | ✅ Phase 4 |
| Icon picker | ✅ | IconPicker | ✅ Phase 4 |
| Color picker | ✅ | ColorPicker | ✅ Phase 4 |

### Auth Pages (Spec Section 4)

| Element | Spec Wireframe | Plan | Status |
|---------|----------------|------|--------|
| Glass card | ✅ | GlassAuthCard | ✅ Phase 5 |
| Logo | ✅ | Logo | ✅ Phase 5 |
| Email input | ✅ | Input | ✅ Phase 5 |
| Password toggle | ✅ | PasswordInput | ✅ Phase 5 |
| Remember checkbox | ✅ | Checkbox | ✅ Phase 5 |
| Submit button | ✅ | Button | ✅ Phase 5 |
| Footer copyright | ✅ | AuthLayout footer | ✅ Phase 5 |

---

## 6. Common Patterns (Spec Section 9)

| Pattern | Spec | Plan | Status |
|---------|------|------|--------|
| Table skeleton | ✅ | TableSkeleton | ✅ Phase 5 |
| Card skeleton | ✅ | CardSkeleton | ✅ Phase 5 |
| Empty state | ✅ | EmptyState | ✅ Phase 5 |
| Error state | ✅ | ErrorState | ✅ Phase 5 |
| Toast notifications | ✅ | Sonner (existing) | ✅ |

---

## 7. Gaps & Action Items

### Minor Gaps (Should Add)

| Item | Phase | Action |
|------|-------|--------|
| SourceBadge (NEW/REDO/COMPLAINT) | Phase 2 | Add to backlog table |
| NotificationBell placeholder | Phase 1 | Add to header (static) |

### Intentionally Skipped (Static Demo)

- Real form validation
- File upload functionality
- Review approve/reject actions
- WIP limits in Kanban
- Date filter in dashboard
- Bulk actions (functional)

---

## 8. Conclusion

**Plan is 95%+ aligned với wireframe spec.**

Tất cả core UI elements đã được cover. Các gaps còn lại là minor và intentionally skipped cho static demo.

**Recommendation:** Proceed with implementation!
