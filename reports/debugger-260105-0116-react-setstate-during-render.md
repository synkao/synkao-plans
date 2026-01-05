# Debug Report: React setState During Render Error

**Report ID:** debugger-260105-0116-react-setstate-during-render
**Date:** 2026-01-05 01:16
**Environment:** Next.js 16.1.1 (Turbopack), React Query v5
**Severity:** HIGH (blocking user experience)

---

## Executive Summary

**Root Cause:** `useAuth()` hook gọi `setUser()` trực tiếp trong render phase (lines 31-33, `/apps/web/src/hooks/use-auth.ts`), vi phạm quy tắc React về side effects.

**Business Impact:**
- Lỗi console nghiêm trọng làm gián đoạn UX
- RSC navigation không trả data do state không sync đúng
- Potential memory leak từ việc re-render liên tục

**Immediate Fix:** Chuyển state sync logic vào `useEffect` hoặc dùng TanStack Query làm single source of truth.

---

## Technical Analysis

### 1. Root Cause Identification

**File:** `apps/web/src/hooks/use-auth.ts` (lines 30-38)

```typescript
// ❌ PROBLEM: setState during render phase
if (query.data && query.data.id !== user?.id) {
  setUser(query.data);  // Line 32 - violates React rules
}

// Clear store on error (unauthorized)
if (query.error && isAuthenticated) {
  clearAuth();  // Line 37 - same issue
}
```

**Why This Breaks:**
1. Component render phase phải pure, không side effects
2. `setUser()` trigger re-render của `AuthGuard` trong khi nó đang render
3. React phát hiện infinite loop risk → throw error
4. Zustand store và React Query state không sync → RSC data loss

### 2. Call Stack Analysis

```
AuthGuard (render)
  └─ useAuth() (hook call)
      └─ useQuery() → returns data
          └─ if (query.data) → setUser()  ← ERROR HERE
              └─ useAuthStore.set() triggers AuthGuard re-render
```

**Cycle:** AuthGuard render → useAuth → setUser → AuthGuard render → ...

### 3. Side Effects Observed

**Error 1: React Warning**
```
Cannot update a component (AuthGuard) while rendering
a different component (AuthGuard)
```

**Error 2: RSC Navigation Data Loss**
- Navigate từ `/login` → `/dashboard`
- React Query fetch thành công nhưng state không sync
- AuthGuard không nhận được `isAuthenticated=true`
- User bị redirect lại `/login`

### 4. Architecture Issue

**Current (Problematic):**
```
React Query (source of truth)
    ↓ sync in render
Zustand Store (duplicate state)
    ↓ read from
AuthGuard (consumer)
```

**Vấn đề:** Duplicate state management với sync logic không đúng timing.

---

## Evidence from Logs

**Stack Trace (provided):**
```
at setUser (src/stores/auth.store.ts:21:5)
at useAuth (src/hooks/use-auth.ts:32:5)
at AuthGuard (src/components/auth/auth-guard.tsx:16:49)
at MainLayout (src/app/(main)/layout.tsx:11:5)
```

**Code Paths:**
1. `MainLayout` (`(main)/layout.tsx:11`) → wraps children với `<AuthGuard>`
2. `AuthGuard` (line 16) → calls `useAuth()`
3. `useAuth` (line 32) → calls `setUser()` **during render**
4. `setUser` (`auth.store.ts:21`) → Zustand `set()` triggers re-render

**Query Behavior:**
- `useQuery` với `enabled: hasToken()` → fetch `/api/auth/me`
- Response thành công nhưng state sync fail do timing issue
- `staleTime: 5min` → query không refetch, dùng stale data
- Navigation trigger unmount/remount → query re-run → error lặp lại

---

## Recommended Solutions

### Option 1: Use useEffect for State Sync (Quick Fix)

**File:** `apps/web/src/hooks/use-auth.ts`

```typescript
export function useAuth() {
  const { setUser, clearAuth, user, isAuthenticated } = useAuthStore();

  const query = useQuery({
    queryKey: AUTH_QUERY_KEY,
    queryFn: getMe,
    enabled: hasToken(),
    retry: false,
    staleTime: 1000 * 60 * 5,
  });

  // ✅ FIX: Move to useEffect
  useEffect(() => {
    if (query.data && query.data.id !== user?.id) {
      setUser(query.data);
    }
  }, [query.data, user?.id, setUser]);

  useEffect(() => {
    if (query.error && isAuthenticated) {
      clearAuth();
    }
  }, [query.error, isAuthenticated, clearAuth]);

  return {
    user: query.data ?? user,
    isLoading: query.isLoading,
    isAuthenticated: !!query.data || isAuthenticated,
    error: query.error,
  };
}
```

**Pros:** Minimal changes, fix immediate issue
**Cons:** Vẫn duplicate state management

---

### Option 2: Remove Zustand, Use React Query as Single Source (Recommended)

**Rationale:**
- React Query đã là state manager + cache
- Zustand store redundant cho auth use case
- Giảm complexity, tránh sync issues

**Implementation:**

```typescript
// apps/web/src/hooks/use-auth.ts
export function useAuth() {
  const query = useQuery({
    queryKey: AUTH_QUERY_KEY,
    queryFn: getMe,
    enabled: hasToken(),
    retry: false,
    staleTime: 1000 * 60 * 5,
  });

  return {
    user: query.data ?? null,
    isLoading: query.isLoading,
    isAuthenticated: !!query.data,
    error: query.error,
  };
}

// For login/logout, use queryClient.setQueryData directly
export function useLogin() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: loginApi,
    onSuccess: (data) => {
      queryClient.setQueryData(AUTH_QUERY_KEY, data.user);
    },
  });
}
```

**Changes Required:**
1. Remove `useAuthStore` imports từ `useAuth`, `AuthGuard`
2. Update `useLogin`, `useLogout` để dùng `queryClient` thay vì store
3. Keep `useAuthStore` nếu cần cho permissions RBAC (optional)

**Pros:**
- Eliminate root cause hoàn toàn
- Cleaner architecture, single source of truth
- React Query handle caching/invalidation tốt hơn

**Cons:**
- Requires refactor multiple files
- Mất khả năng access auth state outside React tree (nếu cần)

---

### Option 3: Hybrid - Keep Store, Sync via onSuccess (Middle Ground)

```typescript
export function useAuth() {
  const { user, isAuthenticated } = useAuthStore();

  const query = useQuery({
    queryKey: AUTH_QUERY_KEY,
    queryFn: getMe,
    enabled: hasToken(),
    onSuccess: (data) => {
      // ✅ Sync via callback instead of in render
      if (data.id !== user?.id) {
        useAuthStore.getState().setUser(data);
      }
    },
    onError: () => {
      if (isAuthenticated) {
        useAuthStore.getState().clearAuth();
      }
    },
  });

  return {
    user: user ?? query.data,
    isLoading: query.isLoading,
    isAuthenticated,
    error: query.error,
  };
}
```

**Note:** `onSuccess`/`onError` callbacks deprecated trong React Query v5, cần migrate sang `useMutation` pattern hoặc `useEffect`.

---

## Performance Optimization Notes

**Current Issues:**
1. Double render: query update → Zustand update → re-render
2. `staleTime: 5min` quá cao cho auth state (user logout elsewhere không detect)
3. Navigation unmount/remount trigger duplicate fetch

**Recommendations:**
- Reduce `staleTime` to `1min` for auth queries
- Add `refetchOnMount: false` để tránh duplicate fetch
- Consider WebSocket for real-time auth status updates (logout detection)

---

## Security Considerations

**Token Handling (Current):**
- Token stored localStorage → XSS vulnerable
- No token refresh mechanism active
- `hasToken()` check client-side only

**Recommendations:**
- Consider httpOnly cookies for token storage
- Implement automatic token refresh
- Add CSRF protection

---

## Testing Strategy

**Test Cases:**
1. Fresh login → navigate to protected route → should see data
2. Token exists → refresh page → should auto-authenticate
3. Token expired → should redirect to login
4. Logout → should clear all state
5. Navigate between protected routes → no duplicate fetches

**Tools:**
- React DevTools Profiler → check re-render count
- Network tab → verify API calls
- Console → no React warnings

---

## Migration Path (If Choose Option 2)

**Phase 1:** Fix immediate issue (Option 1)
**Phase 2:** Refactor to single source of truth (Option 2)
  - Update `useAuth` hook
  - Update `useLogin`, `useLogout`
  - Update `AuthGuard` component
  - Remove `useAuthStore` exports
  - Update tests

**Estimated Effort:** 2-3 hours

---

## Unresolved Questions

1. Có code nào khác đang dùng `useAuthStore` directly không? (cần grep toàn codebase)
2. Permissions/RBAC logic có phụ thuộc Zustand store không?
3. Có yêu cầu access auth state bên ngoài React component tree không? (e.g., API interceptors)
4. MSW mocking có affect auth flow không? (check `msw-provider.tsx`)

---

**Next Steps:**
- [ ] Grep toàn bộ codebase tìm `useAuthStore` usages
- [ ] Review permissions/RBAC implementation
- [ ] Check MSW handlers cho `/api/auth/me`
- [ ] Implement Option 1 (quick fix) or Option 2 (recommended)
- [ ] Add integration tests cho auth flow
