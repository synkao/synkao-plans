---
title: "Hub Switcher Feature"
description: "Implement Hub Switcher UI with mock data, localStorage persistence, and hub selection flow"
status: completed
priority: P2
effort: 6h
branch: main
tags: [feature, ui, hub, mock-data, zustand]
created: 2026-01-28
completed: 2026-01-28
---

# Hub Switcher Feature - Implementation Plan

## Overview

Hub = Team/Workspace where users work. Users can belong to multiple hubs (many-to-many). After login, user selects a hub. Current hub displayed in header next to logo. Click to open left-side drawer to switch hubs.

## Scope

Mock data only approach - no backend integration. Uses localStorage for persistence.

## Phases

| Phase | Description | Effort | Status |
|-------|-------------|--------|--------|
| 1 | Types & Mock Data | 0.5h | ✅ completed |
| 2 | Hub Store (Zustand) | 1h | ✅ completed |
| 3 | Hub Selection Page | 1.5h | ✅ completed |
| 4 | Hub Switcher Components | 2h | ✅ completed |
| 5 | Integration | 1h | ✅ completed |

## Phase Details

### Phase 1: Types & Mock Data
- [phase-01-types-and-mock-data.md](./phase-01-types-and-mock-data.md)
- Create Hub type definitions
- Create mock hubs seed data

### Phase 2: Hub Store (Zustand)
- [phase-02-hub-store.md](./phase-02-hub-store.md)
- currentHub state, userHubs list
- setCurrentHub action
- localStorage persistence

### Phase 3: Hub Selection Page
- [phase-03-hub-selection-page.md](./phase-03-hub-selection-page.md)
- Route: /select-hub
- Display hubs in grid/list
- Select and redirect flow

### Phase 4: Hub Switcher Components
- [phase-04-hub-switcher-components.md](./phase-04-hub-switcher-components.md)
- HubSwitcher button (next to logo)
- HubSwitcherDrawer (left-side Sheet)
- Type badges with colors

### Phase 5: Integration
- [phase-05-integration.md](./phase-05-integration.md)
- Add HubSwitcher to main app header
- Redirect logic: if no hub selected -> /select-hub

## Architecture

```
apps/web/src/
├── types/
│   └── hub.ts                    # Hub type definitions
├── mocks/
│   └── data/
│       └── hubs.data.ts          # Mock hubs seed data
├── stores/
│   └── hub.store.ts              # Zustand hub store
├── components/
│   └── hub/
│       ├── hub-switcher.tsx      # Button in header
│       └── hub-switcher-drawer.tsx  # Left-side Sheet
├── app/
│   └── (main)/
│       └── select-hub/
│           └── page.tsx          # Hub selection page
```

## Key Dependencies

- Existing: zustand, shadcn/ui Sheet, localStorage
- Pattern: Follow auth.store.ts for Zustand patterns
- Pattern: Follow glass-sidebar.tsx for layout integration

## Success Criteria

- [x] User can view hub selection page after login
- [x] User can select a hub from the list
- [x] Selected hub persists in localStorage
- [x] Header shows current hub with click-to-switch
- [x] Drawer slides from left with hub list
- [x] Type badges display with correct colors
- [x] No hub selected -> redirect to /select-hub

## Code Review

**Date**: 2026-01-28
**Score**: 8/10
**Report**: [code-reviewer-260128-2313-hub-switcher-implementation.md](./reports/code-reviewer-260128-2313-hub-switcher-implementation.md)

**Critical Issues**: None
**High Priority**: Admin routes not exempt from HubGuard
**Recommendations**: Add admin route exemption, extract duplicated badge styles, add error boundary

## Known Issues

1. **Admin Route Bug**: `/admin/*` routes incorrectly redirect to hub selection - needs exemption in HubGuard
2. **Badge Style Duplication**: HUB_TYPE_BADGE_STYLES duplicated in 2 files
3. **Hydration Flash**: Brief blank screen during store hydration

## Next Steps

1. Fix admin route exemption in HubGuard
2. Extract badge styles to shared constant
3. Add error boundary around HubGuard
4. Consider skeleton loader instead of null during hydration

## Technical Notes

- localStorage key: `synkao-current-hub`
- Hub types: sale (blue), design (purple), freelance (green)
- Use existing Sheet component with `side="left"`
