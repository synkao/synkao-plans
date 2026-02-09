---
title: "Design Task Drawer UI Updates"
description: "Remove print specs, add description, workspace dropdown, merge status button"
status: pending
priority: P2
effort: 2h
branch: main
tags: [design, ui, drawer, issue-80]
created: 2026-02-08
---

# Design Task Drawer UI Updates (Issue #80)

## Overview

Refactor TaskDetailDrawer: remove Print Specs section entirely, add read-only Description section, add Workspace change dropdown with confirm dialog in header, merge QuickStatusActions into single dropdown button.

## Phases

| # | Phase | Status | Effort | Files Changed |
|---|-------|--------|--------|---------------|
| 1 | [Remove Print Specs + Add Description](./phase-01-remove-print-specs-add-description.md) | pending | 45min | 6 modified, 2 deleted |
| 2 | [Workspace Dropdown + Merge Status Button](./phase-02-workspace-dropdown-merge-status-button.md) | pending | 1h15min | 3 modified, 1 created |

## Key Dependencies

- shadcn/ui components: Sheet, DropdownMenu, AlertDialog, Button, Badge
- Mock data: mockWorkspaces (2 workspaces), mockTasks (60 tasks)
- MockTask type already has `description?: string` and `workspaceId?: string`

## Architecture Notes

- No new state management needed; all changes are UI-level
- Workspace change is demo-only (console.log), no persistence
- Description section is read-only, inline in drawer (no separate component)
- WorkspaceChangeDropdown is new file due to AlertDialog complexity (~80-100 LOC)

## Risk Assessment

- **Low**: Removing PrintSpecs is safe -- only 3 files reference it (print-specs-section.tsx, edit-specs-modal.tsx, order-items.data.ts)
- **Low**: order-item-card-details.tsx has zero printSpecs references
- **Low**: MockTask.description field already exists in type definition

## Compile/Lint Check

After each phase, run:
```bash
cd apps/web && pnpm tsc --noEmit && pnpm lint
```
