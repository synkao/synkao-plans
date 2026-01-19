# Debug Report: POST /api/auth/login Returns 404

**Date:** 2026-01-16 19:35
**Reporter:** debugger-a7c1a9e
**Severity:** High (blocks authentication flow)
**Status:** Root cause identified

---

## Executive Summary

MSW handler registered for `/api/v1/auth/login` but auth client calls `/api/auth/login` (missing `/v1`). URL path mismatch causes 404 since no handler intercepts request.

**Impact:**
- All auth endpoints (login, logout, me, refresh) return 404
- Authentication completely non-functional
- Users cannot log in

**Root Cause:**
- Auth client (`lib/auth-client.ts`) uses `/api` as base URL
- MSW handlers (`mocks-v2/handlers/auth.ts`) use `/api/v1/auth` per convention
- MSWProvider correctly initializes worker but handlers don't match requested paths

---

## Technical Analysis

### 1. URL Pattern Mismatch

**Client Request:**
```typescript
// apps/web/src/lib/auth-client.ts:5
const API_BASE = '/api';

// apps/web/src/lib/auth-client.ts:44
const response = await fetch(`${API_BASE}/auth/login`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(credentials),
});
```
**Actual URL:** `POST /api/auth/login`

**MSW Handler:**
```typescript
// apps/web/src/mocks-v2/handlers/auth.ts:12
const BASE = '/api/v1/auth';

// apps/web/src/mocks-v2/handlers/auth.ts:16
http.post(`${BASE}/login`, async ({ request }) => {
  // handler code
})
```
**Registered URL:** `POST /api/v1/auth/login`

**Result:** MSW worker running but no handler matches `/api/auth/login` → 404

### 2. Architecture Inconsistency

**Correct Standard (Per Convention):**
```typescript
// apps/web/src/lib/api/endpoints.ts:6
export const API_BASE = '/api/v1';

export const ENDPOINTS = {
  auth: {
    login: `${API_BASE}/auth/login`,    // /api/v1/auth/login ✓
    logout: `${API_BASE}/auth/logout`,  // /api/v1/auth/logout ✓
    me: `${API_BASE}/auth/me`,          // /api/v1/auth/me ✓
    refresh: `${API_BASE}/auth/refresh` // /api/v1/auth/refresh ✓
  },
  // ... other endpoints
}
```

**Convention Document:**
```
File: docs/frontend/mock-api-convention.md:57-58

URL Structure: /api/v1/{module}/{resource}[/{id}][/{action}]
Version prefix: /api/v1 (cố định)
```

**Actual MSW Handlers:** All use `/api/v1` correctly
- Auth: `/api/v1/auth/*` ✓
- Design: `/api/v1/design/*` ✓
- Orders: `/api/v1/orders/*` ✓
- Points: `/api/v1/points/*` ✓
- Users: `/api/v1/users/*` ✓

**Problem File:** Only `auth-client.ts` uses wrong base URL `/api`

### 3. MSW Worker Status

**Initialization:** ✓ Working
```typescript
// apps/web/src/components/providers/msw-provider.tsx:24
const { worker } = await import('@/mocks-v2/browser');

await worker.start({
  onUnhandledRequest: 'bypass',
  serviceWorker: {
    url: '/mockServiceWorker.js',
  },
});
```

**Handler Registration:** ✓ Working
```typescript
// apps/web/src/mocks-v2/handlers/index.ts:21-27
export const handlers: HttpHandler[] = [
  ...authHandlers,       // 4 endpoints: login, logout, me, refresh
  ...designHandlers,     // 21 endpoints
  ...ordersHandlers,     // 7 endpoints
  ...pointsHandlers,     // 9 endpoints
  ...usersHandlers,      // 4 endpoints
];

// apps/web/src/mocks-v2/browser.ts:6
export const worker = setupWorker(...handlers);
```

**Service Worker:** ✓ Present
```bash
-rw-r--r-- 1 taquanglinh staff 8.9K Jan 16 18:47 apps/web/public/mockServiceWorker.js
```

**Verification:** MSW v2.12.7, integrity checksum valid, worker active

### 4. Affected Consumers

**Direct Usage:**
```typescript
// apps/web/src/hooks/use-auth.ts:5-11
import {
  login as loginApi,
  logout as logoutApi,
  getMe,
  hasToken,
  type LoginCredentials,
} from '@/lib/auth-client';
```

**Indirect Usage:**
- `apps/web/src/components/auth/auth-guard.tsx`
- Any component using `useAuth()`, `useLogin()`, `useLogout()` hooks

### 5. Timeline Evidence

**Recent MSW Redesign (Commit b24607b - Jan 15, 2026):**
- Implemented 45 endpoints across 5 modules
- 100% compliance with mock-api-convention.md
- All handlers correctly use `/api/v1` base path
- Auth client (`lib/auth-client.ts`) NOT updated during redesign

**Legacy vs New Structure:**
- Old: `apps/web/src/mocks/` (legacy, uses `/api` base - inconsistent)
- New: `apps/web/src/mocks-v2/` (MSW 2.x compliant, uses `/api/v1` - correct)
- MSWProvider switched to `mocks-v2` but auth client still uses old pattern

---

## Code References

### Problem Files

1. **Auth Client (OUTDATED)**
   - Path: `/Users/taquanglinh/Documents/synkao/apps/web/src/lib/auth-client.ts`
   - Line 5: `const API_BASE = '/api';` ❌
   - Line 44: `fetch(\`${API_BASE}/auth/login\`)` → `/api/auth/login` ❌
   - Line 62: `fetch(\`${API_BASE}/auth/logout\`)` → `/api/auth/logout` ❌
   - Line 76: `fetch(\`${API_BASE}/auth/me\`)` → `/api/auth/me` ❌
   - Line 94: `fetch(\`${API_BASE}/auth/refresh\`)` → `/api/auth/refresh` ❌

2. **MSW Auth Handler (CORRECT)**
   - Path: `/Users/taquanglinh/Documents/synkao/apps/web/src/mocks-v2/handlers/auth.ts`
   - Line 12: `const BASE = '/api/v1/auth';` ✓
   - Line 16: `http.post(\`${BASE}/login\`)` → `/api/v1/auth/login` ✓
   - Line 65: `http.post(\`${BASE}/logout\`)` → `/api/v1/auth/logout` ✓
   - Line 71: `http.get(\`${BASE}/me\`)` → `/api/v1/auth/me` ✓
   - Line 85: `http.post(\`${BASE}/refresh\`)` → `/api/v1/auth/refresh` ✓

3. **Correct Endpoint Standard (REFERENCE)**
   - Path: `/Users/taquanglinh/Documents/synkao/apps/web/src/lib/api/endpoints.ts`
   - Line 6: `export const API_BASE = '/api/v1';` ✓
   - Line 10-13: Auth endpoints correctly defined with `/api/v1` prefix ✓

### Supporting Files

4. **MSW Provider**
   - Path: `/Users/taquanglinh/Documents/synkao/apps/web/src/components/providers/msw-provider.tsx`
   - Line 24: Correctly imports from `@/mocks-v2/browser` ✓

5. **Convention Document**
   - Path: `/Users/taquanglinh/Documents/synkao/docs/frontend/mock-api-convention.md`
   - Line 57-62: Specifies `/api/v1/{module}/{resource}` pattern
   - Line 63: Lists `auth` as valid module

---

## Recommended Fixes

### Option 1: Update Auth Client (RECOMMENDED)

**Action:** Migrate `auth-client.ts` to use `/api/v1` base URL

**Rationale:**
- Aligns with convention document
- Matches all other MSW handlers
- Matches `lib/api/endpoints.ts` standard
- Zero impact on MSW handlers (already correct)

**Implementation:**
```typescript
// apps/web/src/lib/auth-client.ts:5
- const API_BASE = '/api';
+ const API_BASE = '/api/v1';
```

**Impact:**
- 1 line change
- Fixes all 4 auth endpoints
- No breaking changes (MSW immediately works)

**Risk:** Low - straightforward change

### Option 2: Add Duplicate Handlers (NOT RECOMMENDED)

**Action:** Register duplicate handlers for `/api/auth/*` in MSW

**Rationale:** Backward compatibility

**Why Not:**
- Violates convention
- Technical debt
- Inconsistent with 41 other endpoints
- Increases maintenance burden

---

## Additional Issues Found

### 1. Inconsistent Import Path

**Auth Client Uses Legacy Types:**
```typescript
// apps/web/src/lib/auth-client.ts:2
import type { MockUser } from '@/mocks/types';
```

Should import from `mocks-v2`:
```typescript
import type { User } from '@/mocks-v2/types';
```

### 2. Dual Mock Structure

**Both directories exist:**
- `/apps/web/src/mocks/` (legacy, 17 files)
- `/apps/web/src/mocks-v2/` (current, 30 files)

**Recommendation:** Remove `/mocks/` after migration complete to prevent confusion

---

## Verification Steps

After applying fix, verify:

1. **MSW Intercepts Request:**
   ```bash
   # Browser console should show:
   [MSW] POST /api/v1/auth/login (200 OK)
   ```

2. **Login Flow Works:**
   - Login form accepts credentials
   - Token stored in localStorage
   - User redirected to dashboard

3. **Protected Routes Work:**
   - `/api/v1/auth/me` returns user data
   - AuthGuard allows access

4. **Logout Works:**
   - `/api/v1/auth/logout` succeeds
   - Token removed
   - Redirect to login

---

## Related Documentation

- Convention: `/Users/taquanglinh/Documents/synkao/docs/frontend/mock-api-convention.md`
- Endpoints Standard: `/Users/taquanglinh/Documents/synkao/apps/web/src/lib/api/endpoints.ts`
- MSW Redesign Commit: `b24607b` (Jan 15, 2026)

---

## Unresolved Questions

1. Should we migrate `MockUser` type from `/mocks/types` to `/mocks-v2/types` or keep both?
2. When will `/mocks/` directory be removed entirely?
3. Are there other clients besides `auth-client.ts` still using `/api` base URL?
