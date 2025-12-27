---
phase: 01
title: "Setup Stores + QueryClient"
status: completed
effort: 2h
reviewed: 2025-12-25
review_status: approved
---

# Phase 01: Setup Stores + QueryClient

## Context

- **Parent Plan:** [plan.md](./plan.md)
- **Issue:** [#5](https://github.com/synkao/synkao/issues/5)
- **Spec Reference:** [Frontend Spec](../../docs/synkao-frontend-spec.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2025-12-25 |
| Priority | P0 |
| Implementation | ✅ Done |
| Review | ✅ Approved |

## Key Insights

1. **Providers exist** - `components/providers/index.tsx` đã có, chỉ cần extend
2. **Types empty** - `types/index.ts` trống, cần thêm User type
3. **Stores empty** - Folder có `.gitkeep`, tạo mới hoàn toàn
4. **MSW ready** - MockUser type đã có trong `mocks/types.ts`

## Requirements

### Packages

```bash
pnpm add zustand @tanstack/react-query
pnpm add -D @tanstack/react-query-devtools
```

### Auth Store (`auth.store.ts`)

```typescript
interface AuthState {
  user: User | null
  permissions: string[]
  isAuthenticated: boolean
  setUser: (user: User | null) => void
  setPermissions: (permissions: string[]) => void
  clearAuth: () => void
}
```

- NO persist (token in httpOnly cookie)
- Use selectors pattern
- Derive `isAuthenticated` from `user !== null`

### UI Store (`ui.store.ts`)

```typescript
interface UIState {
  sidebarOpen: boolean
  sidebarCollapsed: boolean
  toggleSidebar: () => void
  toggleSidebarCollapsed: () => void
  setSidebarOpen: (open: boolean) => void
}
```

- NO theme state (fixed Light Glass)
- Optional persist for sidebar preference

### QueryClient (`query-client.ts`)

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,     // 5 mins
      gcTime: 1000 * 60 * 30,       // 30 mins
      retry: 1,
      refetchOnWindowFocus: false,  // WS sẽ handle
    },
    mutations: {
      retry: 0,
    },
  },
})
```

### User Type (`types/index.ts`)

```typescript
export interface User {
  id: string
  email: string
  name: string
  role: 'STAFF' | 'DESIGNER' | 'ADMIN'
  avatar?: string
}
```

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Providers                          │
│  ┌───────────────────────────────────────────────┐  │
│  │           QueryClientProvider                  │  │
│  │  ┌─────────────────────────────────────────┐  │  │
│  │  │           MSWProvider (dev)             │  │  │
│  │  │  ┌───────────────────────────────────┐  │  │  │
│  │  │  │            App Content            │  │  │  │
│  │  │  └───────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘

Zustand: No provider needed (hook-based)
```

## Related Code Files

| File | Action | Notes |
|------|--------|-------|
| `apps/web/package.json` | UPDATE | Add 3 packages |
| `apps/web/src/stores/auth.store.ts` | CREATE | Auth state |
| `apps/web/src/stores/ui.store.ts` | CREATE | UI state |
| `apps/web/src/lib/query-client.ts` | CREATE | QueryClient |
| `apps/web/src/types/index.ts` | UPDATE | User type |
| `apps/web/src/components/providers/index.tsx` | UPDATE | Add QueryProvider |
| `apps/web/src/components/providers/query-provider.tsx` | CREATE | QueryClientProvider wrapper |
| `apps/web/src/app/dev-tools/page.tsx` | CREATE | Demo page for testing |

## Implementation Steps

### Step 1: Install Packages

```bash
cd apps/web
pnpm add zustand @tanstack/react-query
pnpm add -D @tanstack/react-query-devtools
```

### Step 2: Create User Type

Update `src/types/index.ts`:
- Add User interface
- Export UserRole type

### Step 3: Create Auth Store

Create `src/stores/auth.store.ts`:
- AuthState interface
- useAuthStore with Zustand
- Selector hooks: `useUser()`, `useIsAuthenticated()`, `usePermissions()`

### Step 4: Create UI Store

Create `src/stores/ui.store.ts`:
- UIState interface
- useUIStore with Zustand
- Selector hooks: `useSidebarOpen()`, `useSidebarCollapsed()`

### Step 5: Create QueryClient

Create `src/lib/query-client.ts`:
- Export createQueryClient function
- Configure defaults

### Step 6: Create Query Provider

Create `src/components/providers/query-provider.tsx`:
- Use `useState` for client (prevent SSR issues)
- Include DevTools in dev

### Step 7: Update Providers

Update `src/components/providers/index.tsx`:
- Import QueryProvider
- Wrap children with QueryProvider

### Step 8: Create Demo Page

Create `src/app/dev-tools/page.tsx`:
- Auth Store Demo: Login/Logout buttons, display user info
- UI Store Demo: Toggle sidebar buttons, display state
- Query Demo: Fetch tasks from MSW, display results
- All in one page for easy testing

### Step 9: Verify

```bash
pnpm lint
pnpm build
pnpm dev  # Test /dev-tools page
```

## Todo List

- [x] Install packages
- [x] Create User type
- [x] Create auth.store.ts
- [x] Create ui.store.ts
- [x] Create query-client.ts
- [x] Create query-provider.tsx
- [x] Update Providers component
- [x] Create /dev-tools demo page
- [x] Run lint & build
- [ ] Test demo page (manual browser test required)

## Success Criteria

1. ✅ `pnpm lint` passes
2. ✅ `pnpm build` succeeds
3. ✅ Stores importable: `import { useAuthStore } from '@/stores/auth.store'`
4. ✅ Query importable: `import { useQuery } from '@tanstack/react-query'`
5. ✅ DevTools visible in browser (dev mode)
6. ✅ Demo page `/dev-tools` works:
   - Login/Logout updates auth store
   - Sidebar toggle updates UI store
   - Query fetches tasks from MSW

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| React 19 compatibility | Medium | TanStack Query v5 supports React 18+, should work |
| SSR hydration mismatch | Low | Use `useState` for QueryClient creation |
| Zustand persist conflicts | N/A | Not using persist |

## Security Considerations

- ❌ NO tokens in store (httpOnly cookie only)
- ❌ NO sensitive data persisted
- ✅ User info from API response only

## Next Steps

After completion:
1. Create API hooks using React Query (Issue #6+)
2. Integrate auth store with login flow
3. Setup WebSocket for real-time (future)
