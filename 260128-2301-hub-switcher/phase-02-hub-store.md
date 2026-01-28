# Phase 2: Hub Store (Zustand)

## Context Links
- [Main Plan](./plan.md)
- [Phase 1: Types & Mock Data](./phase-01-types-and-mock-data.md)
- [Existing auth.store.ts](../../apps/web/src/stores/auth.store.ts)
- [Existing ui.store.ts](../../apps/web/src/stores/ui.store.ts)

## Overview
- **Priority**: P1 (Core state management)
- **Status**: ✅ completed
- **Effort**: 1h
- **Dependencies**: Phase 1 completed
- **Completed**: 2026-01-28

## Key Insights
- Follow existing Zustand patterns from auth.store.ts
- Use zustand/middleware for persist
- Selector pattern for optimized re-renders
- localStorage key: `synkao-current-hub`

## Requirements

### Functional
- Store currentHub state
- Store userHubs list (mock data)
- setCurrentHub action
- clearHub action for logout
- Persist currentHub to localStorage

### Non-functional
- Type-safe store
- Hydration handling for SSR
- Selector pattern for performance

## Architecture

```typescript
interface HubState {
  // State
  currentHub: Hub | null;
  userHubs: Hub[];
  isHydrated: boolean;

  // Actions
  setCurrentHub: (hub: Hub | null) => void;
  clearHub: () => void;
  setHydrated: () => void;
}
```

## Related Code Files

### Files to Create
1. `apps/web/src/stores/hub.store.ts` - Zustand hub store with persist

### Files to Modify
- None

## Implementation Steps

### Step 1: Create Hub Store
Create `apps/web/src/stores/hub.store.ts`:

```typescript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import type { Hub } from '@/types';
import { mockHubs } from '@/mocks/data/hubs.data';

const STORAGE_KEY = 'synkao-current-hub';

// Hub Store State
interface HubState {
  currentHub: Hub | null;
  userHubs: Hub[];
  isHydrated: boolean;
  setCurrentHub: (hub: Hub | null) => void;
  clearHub: () => void;
  setHydrated: () => void;
}

// Hub Store with persist
export const useHubStore = create<HubState>()(
  persist(
    (set) => ({
      currentHub: null,
      userHubs: mockHubs, // Use mock data
      isHydrated: false,

      setCurrentHub: (hub) => set({ currentHub: hub }),

      clearHub: () => set({ currentHub: null }),

      setHydrated: () => set({ isHydrated: true }),
    }),
    {
      name: STORAGE_KEY,
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ currentHub: state.currentHub }),
      onRehydrateStorage: () => (state) => {
        state?.setHydrated();
      },
    }
  )
);

// Selectors - use these for optimized re-renders
export const useCurrentHub = () => useHubStore((state) => state.currentHub);
export const useUserHubs = () => useHubStore((state) => state.userHubs);
export const useHubHydrated = () => useHubStore((state) => state.isHydrated);
```

### Step 2: Handle SSR Hydration
The persist middleware handles hydration automatically. Components should check `isHydrated` before rendering hub-dependent UI to avoid hydration mismatch.

Usage pattern:
```typescript
const isHydrated = useHubHydrated();
const currentHub = useCurrentHub();

if (!isHydrated) return <Skeleton />;
if (!currentHub) return <Redirect to="/select-hub" />;
```

## Todo List
- [x] Create hub.store.ts with persist middleware
- [x] Implement currentHub and userHubs state
- [x] Implement setCurrentHub and clearHub actions
- [x] Add hydration handling
- [x] Create selector hooks
- [x] Test localStorage persistence
- [x] Verify no hydration mismatch warnings

## Success Criteria
- [x] Hub state persists across page refresh
- [x] Selectors work for optimized re-renders
- [x] No SSR hydration warnings
- [x] clearHub works for logout flow
- [x] Mock hubs load correctly into userHubs

## Review Notes
Excellent implementation following existing auth.store patterns. Proper hydration handling with isHydrated flag. Selector pattern for performance optimization. localStorage persistence working correctly.

## Risk Assessment
- **Medium risk** - SSR hydration can cause issues
- Mitigation: isHydrated flag and proper hydration handling

## Security Considerations
- localStorage stores hub ID only
- No sensitive data in persisted state

## Next Steps
- Phase 3: Create Hub Selection Page
