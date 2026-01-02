# Brainstorm Report: Issue #5 State Management

**Date:** 2025-12-25
**Issue:** [#5 - Setup Zustand + TanStack Query](https://github.com/synkao/synkao/issues/5)

---

## Problem Statement

Setup state management và data fetching cho Synkao POD Order Management frontend.

### Requirements từ Issue

1. Install packages
2. Create auth store (user, permissions)
3. Create UI store (sidebar, theme)
4. Configure QueryClient với defaults

---

## Approaches Evaluated

### Store Organization

| Option | Pros | Cons |
|--------|------|------|
| **A: Multiple Stores** ✅ | Separation of concerns, dễ test | - |
| B: Single Store + Slices | Centralized | Overkill cho scope nhỏ |

**Chosen:** Option A - Multiple stores (KISS principle)

### Auth Store Persistence

| Option | Pros | Cons |
|--------|------|------|
| **A: No Persist** ✅ | Secure, token in cookie | Need re-fetch on refresh |
| B: localStorage Persist | Fast restore | Security risk |

**Chosen:** No persist - token đã trong httpOnly cookie

### Theme System

| Option | Pros | Cons |
|--------|------|------|
| **A: No Theme State** ✅ | Simpler, YAGNI | Fixed theme only |
| B: Theme Switching | Flexibility | Unnecessary complexity |

**Chosen:** No theme state - App chỉ cần Light Glass theme

### Real-time Strategy

| Option | Pros | Cons |
|--------|------|------|
| **A: WS + Query Invalidation** ✅ | Simple, single source | Need WS setup |
| B: Separate WS State | Dual control | Complex sync |

**Chosen:** WebSocket events invalidate React Query cache

---

## Final Recommendations

### Packages

```bash
pnpm add zustand @tanstack/react-query
pnpm add -D @tanstack/react-query-devtools
```

### Auth Store

- Fields: `user`, `permissions`, `isAuthenticated` (derived)
- Actions: `setUser`, `setPermissions`, `clearAuth`
- No persist

### UI Store

- Fields: `sidebarOpen`, `sidebarCollapsed`
- Actions: `toggleSidebar`, `toggleSidebarCollapsed`
- No theme (fixed Light Glass)

### QueryClient Config

```typescript
{
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,    // 5 mins
      gcTime: 30 * 60 * 1000,      // 30 mins
      retry: 1,
      refetchOnWindowFocus: false, // WS handles updates
    },
  },
}
```

### DevTools

- Enable in development only
- React Query DevTools for debugging

---

## Implementation Considerations

1. **Provider Order:** QueryClientProvider > MSWProvider > App
2. **SSR Safety:** Use `useState` for QueryClient creation
3. **Selectors:** Always use selectors pattern for Zustand
4. **Type Safety:** Reuse MockUser → User type

---

## Risks

| Risk | Probability | Mitigation |
|------|-------------|------------|
| React 19 compat | Low | TanStack Query v5 tested |
| Hydration mismatch | Low | `useState` for client |

---

## Sources

- [Zustand GitHub](https://github.com/pmndrs/zustand)
- [Working with Zustand - TkDodo](https://tkdodo.eu/blog/working-with-zustand)
- [TanStack Query v5 Docs](https://tanstack.com/query/v5/docs/framework/react/installation)
- [React State Management 2025](https://www.developerway.com/posts/react-state-management-2025)

---

## Unresolved Questions

None - All decisions clarified through brainstorm session.
