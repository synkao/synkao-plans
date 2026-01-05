---
title: "Backlog Stats Cards Enhancement"
description: "Update BacklogStatsBar từ 2 lên 4 stats cards: Total, Urgent, Redesign, Avg.Wait"
status: in_progress
priority: P3
effort: 30m
branch: main
tags: [ui, design-backlog, stats]
created: 2026-01-05
---

# Backlog Stats Cards Enhancement

## Overview

Update `BacklogStatsBar` component để hiển thị 4 cards thay vì 2, giúp Manager/Team Lead có thêm insight về workload và nguồn task.

## Current State

- 2 cards: Total Tasks, Urgent
- Layout: `grid-cols-2 max-w-md`

## Target State

| # | Card | Metric | Icon | Color |
|---|------|--------|------|-------|
| 1 | Total Tasks | `tasks.length` | FileStack | blue |
| 2 | Urgent | `isUrgent = true` | Flame | red |
| 3 | Redesign | `source = REDESIGN` | RefreshCw | amber |
| 4 | Avg. Wait | `avg(now - createdAt)` hours | Clock | gray |

## Implementation Phases

| Phase | Description | Status | File |
|-------|-------------|--------|------|
| 01 | Update BacklogStatsBar component | ⏳ Pending | [phase-01](./phase-01-update-stats-bar.md) |

## Files Changed

- `apps/web/src/features/design/components/backlog/backlog-stats-bar.tsx`

## Success Criteria

- [ ] 4 cards hiển thị đúng data
- [ ] Responsive: 4 cols desktop, 2x2 mobile
- [ ] Avg. Wait format: "36h", "-" if empty
- [ ] Giữ nguyên glassmorphism styling

## References

- Brainstorm: [reports/brainstorm-260105-1059-backlog-stats-cards.md](../reports/brainstorm-260105-1059-backlog-stats-cards.md)
