# Debug Report: `/api/auth/me` Returns 404

**Date:** 2026-01-16
**Status:** Root cause identified
**Severity:** High

## Executive Summary

The `/api/auth/me` endpoint returns 404 because client requests `/api/auth/me` but MSW handlers are configured for `/api/v1/auth/me`. API version mismatch between client and mock server.

**Impact:**
- Authentication fails in development
- User session cannot be validated
- Protected routes inaccessible

## Root Cause

**Path Mismatch:**
- **Client:** Uses `API_BASE = '/api'` → calls `/api/auth/me`
- **MSW Handlers:** Use `BASE = '/api/v1/auth'` → intercepts `/api/v1/auth/me`
- **Result:** Request bypasses MSW, hits Next.js, returns 404 (no route exists)

## Evidence

### 1. Client Configuration
**File:** `apps/web/src/lib/auth-client.ts`
```typescript
const API_BASE = '/api';  // Line 5

export async function getMe(): Promise<MockUser> {
  const response = await fetch(`${API_BASE}/auth/me`, {  // Line 76
    headers: getAuthHeaders(),
  });
  // ...
}
```

### 2. MSW Handler Configuration
**File:** `apps/web/src/mocks-v2/handlers/auth.ts`
```typescript
const BASE = '/api/v1/auth';  // Line 12

export const authHandlers = [
  // GET /api/v1/auth/me
  http.get(`${BASE}/me`, async ({ request }) => {  // Line 71
    // ...
  }),
];
```

### 3. Endpoint Constants (Correct)
**File:** `apps/web/src/lib/api/endpoints.ts`
```typescript
export const API_BASE = '/api/v1';  // Line 6

export const ENDPOINTS = {
  auth: {
    me: `${API_BASE}/auth/me`,  // Results in '/api/v1/auth/me'
  },
};
```

### 4. Next.js API Routes
**Directory:** `apps/web/src/app/api/`
```
api/
└── health/
    └── route.ts
```
No auth routes exist → 404 when `/api/auth/*` called directly.

## Analysis

### Timeline
1. User visits protected page
2. `useAuth()` hook calls `getMe()` from `auth-client.ts`
3. Fetch sends GET request to `/api/auth/me`
4. MSW worker doesn't intercept (expects `/api/v1/auth/me`)
5. Request hits Next.js server
6. No matching route handler found
7. Returns 404

### Why It Happens
**Inconsistent API Base:**
- `auth-client.ts` hardcodes `/api` for legacy compatibility
- `endpoints.ts` correctly uses `/api/v1`
- MSW handlers correctly use `/api/v1/auth`
- Other modules likely use `ENDPOINTS` const (working)
- Auth module uses old `auth-client.ts` (broken)

### Affected Code
- `apps/web/src/lib/auth-client.ts` (all functions)
- `apps/web/src/hooks/use-auth.ts` (imports auth-client)

## Recommendations

### Immediate Fix (Priority 1)
**Update `auth-client.ts` to use correct API base:**

```typescript
// Before
const API_BASE = '/api';

// After
import { ENDPOINTS } from './api/endpoints';

// Replace hardcoded paths
export async function login(credentials: LoginCredentials): Promise<LoginResponse> {
  const response = await fetch(ENDPOINTS.auth.login, {  // Use centralized constant
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(credentials),
  });
  // ...
}

export async function getMe(): Promise<MockUser> {
  const response = await fetch(ENDPOINTS.auth.me, {  // Use centralized constant
    headers: getAuthHeaders(),
  });
  // ...
}

export async function logout(): Promise<void> {
  await fetch(ENDPOINTS.auth.logout, {  // Use centralized constant
    method: 'POST',
    headers: getAuthHeaders(),
  });
  removeToken();
}

export async function refreshToken(): Promise<string> {
  const response = await fetch(ENDPOINTS.auth.refresh, {  // Use centralized constant
    method: 'POST',
    headers: getAuthHeaders(),
  });
  // ...
}
```

**Files to modify:**
- `apps/web/src/lib/auth-client.ts` (update 4 functions)

**Testing:**
1. Import ENDPOINTS from `./api/endpoints`
2. Replace all `${API_BASE}/auth/*` with `ENDPOINTS.auth.*`
3. Run dev server: `pnpm dev`
4. Test login flow
5. Verify `/api/v1/auth/me` returns user data

### Long-term Improvements (Priority 2)

**1. Consolidate API Client Pattern**
- Create unified API client helper
- All modules import from centralized constants
- Single source of truth for endpoint paths

**2. Add Endpoint Validation**
- Verify MSW handlers match ENDPOINTS constants
- Unit test to ensure path parity
- Fail build if mismatch detected

**3. Documentation**
- Document API versioning strategy
- Update mock API development guide
- Add migration guide for future versions

**4. Monitoring**
- Add logging for unhandled requests in MSW
- Log warning when API_BASE doesn't match expected pattern

## Security Considerations

None. This is development-only issue (MSW runs in dev mode). Production uses real backend with proper routing.

## Testing Plan

**Unit Tests:**
- Mock fetch and verify correct URLs called
- Test all auth functions use `/api/v1/*` paths

**Integration Tests:**
- Login → verify token stored
- getMe → verify user data returned
- Logout → verify token cleared
- Refresh → verify new token received

**Manual Tests:**
1. Login at `/login`
2. Navigate to `/design` (protected)
3. Verify auth state persists
4. Check browser DevTools Network tab → all auth calls use `/api/v1/auth/*`
5. Refresh page → verify session maintained

## Files Changed

**Modified:**
- `apps/web/src/lib/auth-client.ts` (4 function calls, 1 import)

**No changes needed:**
- `apps/web/src/mocks-v2/handlers/auth.ts` (already correct)
- `apps/web/src/lib/api/endpoints.ts` (already correct)
- `apps/web/src/hooks/use-auth.ts` (no changes needed)

## Success Criteria

- [x] Root cause identified
- [ ] Fix implemented in `auth-client.ts`
- [ ] Manual testing passes
- [ ] No 404 errors in console
- [ ] Auth flow works end-to-end

## Next Steps

1. Apply immediate fix to `auth-client.ts`
2. Test login/logout/getMe flows
3. Verify no regressions in other auth features
4. Consider consolidating API client pattern across codebase
5. Update documentation if needed

## Unresolved Questions

None. Solution is straightforward path alignment.
