---
title: "FE-3 Remaining Issues Implementation"
description: "Complete #17 Backlog Bulk Actions and #18 Workspace Selector"
status: completed
priority: P0
effort: 6-7h
issue: 17, 18
branch: main
tags: [frontend, design, kanban, workspace]
created: 2026-01-04
---

# FE-3 Remaining Issues Implementation

## Overview

Hoàn thành 2 issues còn lại của FE-3 milestone:
- **#17** Backlog Bulk Actions (P0) - Action bar khi select tasks
- **#18** Workspace Selector (P1) - Header dropdown + create modal

## Phases

| # | Phase | Status | Effort | Link |
|---|-------|--------|--------|------|
| 1 | Backlog Bulk Actions | ✅ Done | 3-4h | [phase-01](./phase-01-backlog-bulk-actions.md) |
| 2 | Workspace Selector | ✅ Done | 3h | [phase-02-workspace-selector.md](./phase-02-workspace-selector.md) |

## Dependencies

- Existing: `BacklogTable` với selection state (`selectedIds`)
- Existing: `WorkspaceCard` component
- Existing: `mockWorkspaces`, `mockUsers` data
- UI: `DropdownMenu`, `Dialog`, `Select` từ shadcn/ui

## Architecture

```
apps/web/src/
├── features/design/components/
│   ├── backlog/
│   │   ├── backlog-action-bar.tsx    ← NEW #17
│   │   └── index.ts                  ← UPDATE
│   └── workspace/                    ← NEW folder
│       ├── workspace-selector.tsx    ← NEW #18
│       ├── create-workspace-modal.tsx← NEW #18
│       └── index.ts                  ← NEW
├── components/layout/
│   └── app-header.tsx                ← UPDATE #18
└── mocks/handlers/
    └── workspaces.ts                 ← NEW (bulk + CRUD)
```

## Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Bulk action feedback | Toast (direct) | User confirmed no modal needed |
| Create workspace fields | Name + Desc only | Columns auto-created |
| State management | Local state + lift | Keep simple, use props |
| Form validation | react-hook-form + zod | Already in codebase |

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Selection state scope | Low | Lift to parent if needed |
| MSW handler conflicts | Low | Namespace endpoints |

## Reports

- [Brainstorm Report](../reports/brainstorm-260104-2326-fe3-milestone-review.md)
