---
title: "Issue #5: Setup Zustand + TanStack Query"
description: "State management và data fetching config cho Synkao frontend"
status: completed
priority: P0
effort: 2h
branch: main
tags: [fe, setup, zustand, tanstack-query]
created: 2025-12-25
completed: 2025-12-25
---

# Issue #5: Setup Zustand + TanStack Query

## Overview

Thiết lập state management (Zustand) và data fetching (TanStack Query) cho Synkao frontend.

## Context

- **Issue:** [#5](https://github.com/synkao/synkao/issues/5)
- **Milestone:** FE-1: Setup + MSW
- **Labels:** fe, P0, setup
- **Due:** 2025-12-26

## Key Decisions (from Brainstorm)

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Theme | Fixed Light Glass | No dark mode needed |
| Auth token | httpOnly cookie | Backend handles |
| Auth store | No persist | Security |
| UI store | Sidebar only | No theme state |
| Real-time | WebSocket + Query invalidation | Simple pattern |
| DevTools | Dev only | Debug productivity |

## Phases

| # | Phase | Status | Effort |
|---|-------|--------|--------|
| 1 | [Setup Stores + QueryClient](./phase-01-setup-stores-query.md) | ✅ DONE | 2h |

## Files to Create/Modify

```
apps/web/
├── package.json                    # Add dependencies
├── src/
│   ├── app/
│   │   └── dev-tools/
│   │       └── page.tsx           # NEW: Demo page
│   ├── stores/
│   │   ├── auth.store.ts          # NEW: User + permissions
│   │   └── ui.store.ts            # NEW: Sidebar state
│   ├── lib/
│   │   └── query-client.ts        # NEW: QueryClient config
│   ├── types/
│   │   └── index.ts               # UPDATE: Add User type
│   └── components/
│       └── providers/
│           ├── index.tsx          # UPDATE: Add QueryClientProvider
│           └── query-provider.tsx # NEW: Query provider
```

## Success Criteria

- [ ] Packages installed
- [ ] Auth store with selectors working
- [ ] UI store with sidebar toggle working
- [ ] QueryClient configured with defaults
- [ ] DevTools visible in development
- [ ] Demo page `/dev-tools` functional
- [ ] Build passes without errors

## Reports

- [Brainstorm Decisions](./reports/brainstorm-decisions.md)
