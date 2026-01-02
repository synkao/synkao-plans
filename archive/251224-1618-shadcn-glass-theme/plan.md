---
title: "Setup shadcn/ui + Light Functional Glassmorphism"
description: "Cấu hình shadcn/ui với Light Glass theme theo mock mới + Phase Colors"
status: done
priority: P0
effort: 2h
issue: 2
branch: main
tags: [frontend, ui, design-system]
created: 2025-12-24
updated: 2025-12-25
---

# Setup shadcn/ui + Light Functional Glassmorphism

## Overview

Update design system từ dark glass sang Light Functional Glassmorphism theo mock mới. Thêm Phase Colors cho workflow tracking.

## Context

- **Issue:** #2
- **Depends on:** Issue #1 (Next.js + Tailwind setup) - Done
- **Mock:** [synkao-design-system-demo.html](./synkao-design-system-demo.html)
- **Brainstorm:** [brainstorm-251225-1023](../reports/brainstorm-251225-1023-light-glass-design-system-update.md)
- **Design System:** [synkao-design-system.md](../../docs/synkao-design-system.md)
- **Working dir:** `apps/web/`

## Key Decisions

| Decision | Choice |
|----------|--------|
| Theme direction | Light-only (no dark mode) |
| Phase colors | Required (5 colors) |
| Typography | Keep Inter |
| Glass opacity | 70% (hover: 85%) |
| Brand color | Blue-500 (#3B82F6) |

## Phases

| # | Phase | Status | Effort | Link |
|---|-------|--------|--------|------|
| 1 | Initialize shadcn/ui | Done | 45min | [phase-01](./phase-01-initialize-shadcn.md) |
| 2 | Light Glass + Phase Colors | Done (2025-12-25 11:00) | 1h | [phase-02](./phase-02-light-glass-theme.md) |
| 3 | Update Documentation | Done (2025-12-25 11:05) | 15min | [phase-03](./phase-03-update-docs.md) |

## Changes Summary

### Glass Effect
- Opacity: 60% → 70%
- Hover: 70% → 85%
- Background: White → Slate-50 (#F8FAFC)

### Brand Color
- Primary: Black → Blue-500 (#3B82F6)

### Phase Colors (NEW)
| Phase | Color | Hex |
|-------|-------|-----|
| Order | Slate-500 | #64748B |
| Design | Amber-500 | #F59E0B |
| Fulfill | Violet-500 | #8B5CF6 |
| Complete | Emerald-500 | #10B981 |
| Cancelled | Gray-500 | #6B7280 |

### Dark Mode
- Removed entirely (KISS principle)

## Files to Update

| File | Changes |
|------|---------|
| `apps/web/src/app/globals.css` | Core CSS variables, remove dark mode |
| `docs/synkao-design-system.md` | Update documentation |

## Success Criteria

- [x] Light glass effect matches mock (70% opacity)
- [x] Brand color is Blue-500
- [x] Phase colors available: `bg-phase-order`, `bg-phase-design`, etc.
- [x] No dark mode remnants
- [x] Documentation updated
