# Brainstorm Report: FE-3 Design Workspace Milestone Review

**Date:** 2026-01-04
**Milestone:** [FE-3: Design Workspace](https://github.com/synkao/synkao/milestone/3)
**Deadline:** 2026-01-09 (5 days remaining)

---

## Executive Summary

Milestone FE-3 **đã hoàn thành ~85%**. 6/8 issues done, còn 2 issues cần bổ sung tính năng.

| Metric | Value |
|--------|-------|
| Total Issues | 8 |
| Completed | 6 (75%) |
| Partial | 2 (25%) |
| Est. Remaining | 6-7h |

---

## Issue Status Overview

### ✅ Completed (6/8)

| # | Title | Est | Components |
|---|-------|-----|------------|
| #12 | Kanban Board layout | 4h | `kanban-board.tsx`, `kanban-column.tsx` |
| #13 | Task Card | 3h | `task-card.tsx` |
| #14 | Drag & Drop | 8h | `sortable-task-card.tsx`, `drag-overlay-card.tsx`, `drop-placeholder.tsx` |
| #15 | Task Detail Drawer | 6h | 15 components trong `/drawer/` |
| #16 | Version List + Upload | 4h | `design-versions-section.tsx`, `version-upload-form.tsx`, `image-lightbox.tsx` |
| #19 | Mock API handlers | 2h | `tasks.ts` (6 endpoints) |

### ⚠️ Partial (2/8)

| # | Title | Done | Missing |
|---|-------|------|---------|
| #17 | Backlog page | Table, filters, stats, search | **Bulk action bar** (Add to Workspace, Assign, Set Priority) |
| #18 | Workspace selector | WorkspaceCard component | **Header dropdown** + **Create modal** |

---

## Technical Analysis

### Architecture Match ✅

Codebase structure khớp với best practices đề xuất:

```
apps/web/src/
├── features/design/
│   ├── components/
│   │   ├── kanban/      ✅ 7 files
│   │   ├── drawer/      ✅ 15 files
│   │   └── backlog/     ✅ 4 files
│   ├── hooks/           ✅ 3 files
│   └── lib/             ✅ utils, constants
├── mocks/handlers/      ✅ tasks.ts
└── stores/              ✅ zustand stores
```

### Dependencies ✅

All recommended packages already installed:

- `@dnd-kit/core`, `@dnd-kit/sortable`, `@dnd-kit/utilities`
- `@tanstack/react-query`
- `zustand`
- `msw`
- `react-dropzone`, `react-hook-form`, `zod`

---

## Gap Analysis

### #17 Backlog Bulk Actions

**Current:**
- ✅ BacklogTable với selection state
- ✅ BacklogFilterBar với search, priority, source filters
- ✅ BacklogStatsBar với metrics
- ❌ No action bar when items selected

**Needed:**
```tsx
// New component: BacklogActionBar
<BacklogActionBar
  selectedCount={selectedIds.size}
  onAddToWorkspace={(workspaceId) => {...}}
  onAssignDesigner={(designerId) => {...}}
  onSetPriority={(priority) => {...}}
  onClearSelection={() => {...}}
/>
```

**Est:** 3-4h

### #18 Workspace Selector

**Current:**
- ✅ WorkspaceCard for listing workspaces
- ✅ `/design/workspace/page.tsx` shows workspace list
- ✅ `/design/workspace/[id]/page.tsx` shows board
- ❌ No dropdown selector in header
- ❌ No create workspace modal

**Needed:**
```tsx
// Component: WorkspaceSelector (header dropdown)
<WorkspaceSelector
  currentWorkspace={activeWorkspace}
  workspaces={workspaces}
  onSelect={(id) => router.push(`/design/workspace/${id}`)}
  onCreate={() => setCreateModalOpen(true)}
/>

// Component: CreateWorkspaceModal
<CreateWorkspaceModal
  open={createModalOpen}
  onOpenChange={setCreateModalOpen}
  onCreated={(workspace) => {...}}
/>
```

**Est:** 3h

---

## Recommended Implementation Order

```
1. #17 Bulk Actions (3-4h)
   └── Lý do: P0, directly impacts user workflow

2. #18 Workspace Selector (3h)
   └── Lý do: P1, can ship without if needed
```

---

## Technical Decisions Confirmed

| Decision | Choice | Rationale |
|----------|--------|-----------|
| DnD Library | @dnd-kit | ✅ Already implemented |
| Server State | TanStack Query | ✅ Setup done |
| UI State | Zustand | ✅ Used for auth, ui stores |
| Mock API | MSW | ✅ 6 endpoints working |
| Architecture | Feature-based | ✅ Clean separation |

---

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Bulk actions edge cases | Medium | Start with simple single-action, add batch later |
| Workspace context complexity | Low | Use URL params for workspace ID, avoid global state |
| Deadline pressure (5 days) | Low | 6-7h work, achievable in 1-2 days |

---

## Next Steps

1. **Close issues #12-16, #19** - Mark as complete on GitHub
2. **Implement #17** - Add bulk action bar to backlog
3. **Implement #18** - Add workspace selector dropdown + create modal
4. **Test end-to-end** - Full workflow from backlog → workspace → kanban

---

## Unresolved Questions

1. #17: Bulk "Add to Workspace" - Có cần xác nhận modal không?
2. #18: Create workspace - Có cần setup default columns không hay chỉ name/description?

---

*Report generated during brainstorm session 2026-01-04*
