# Research: Auth Patterns for Next.js App Router + TanStack Query

## Current Implementation Status

### 1. State Management (Zustand)
**File:** `apps/web/src/stores/auth.store.ts`

```typescript
interface AuthState {
  user: User | null;
  permissions: string[];
  isAuthenticated: boolean;
  setUser: (user: User | null) => void;
  setPermissions: (permissions: string[]) => void;
  clearAuth: () => void;
}
```

**Status:** ✅ Basic structure in place
- Zustand store manages user state + permissions
- Selectors optimize re-renders (`useUser`, `useIsAuthenticated`, `usePermissions`)
- No localStorage persistence yet
- No token management in store

### 2. TanStack Query Setup (v5)
**Files:**
- `apps/web/src/lib/query-client.ts`
- `apps/web/src/components/providers/query-provider.tsx`

**Config:**
```typescript
defaultOptions: {
  queries: { staleTime: 5min, gcTime: 30min, retry: 1, refetchOnWindowFocus: false },
  mutations: { retry: 0 }
}
```

**Status:** ✅ Proper setup
- QueryClient created once per session (useState pattern)
- SSR hydration safe
- No auth interceptors configured
- No error handling for 401s

### 3. MSW Mocking
**File:** `apps/web/src/mocks/handlers/auth.ts`

**Endpoints:**
- `POST /api/auth/login` → returns `{ user, token }`
- `POST /api/auth/logout`
- `POST /api/auth/refresh`
- `GET /api/auth/me`

**Status:** ⚠️ Basic handlers exist, no auth persistence in mock

### 4. Route Structure
**Protected routes:** `(main)` group - requires middleware check
**Auth routes:** `(auth)` group - login page (no form logic yet)

**Status:** ⚠️ Layout defined, no protection middleware

---

## Best Practices for Next.js App Router + TanStack Query

### A. Token Persistence
**Pattern: Dual Storage (Cookie + Memory)**

```typescript
// lib/auth-client.ts
export const authClient = {
  setToken: (token: string) => {
    localStorage.setItem('auth_token', token);
    document.cookie = `auth_token=${token}; HttpOnly; Secure; SameSite=Strict`;
  },

  getToken: () => localStorage.getItem('auth_token'),

  clearToken: () => {
    localStorage.removeItem('auth_token');
    document.cookie = 'auth_token=; expires=Thu, 01 Jan 1970 00:00:00 UTC;';
  }
};
```

**Why:** Cookies for server-side hydration, localStorage for client-side access.

### B. Auth Mutations with TanStack Query
**Pattern: Login Mutation**

```typescript
export const useLoginMutation = () => {
  const { setUser } = useAuthStore();
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (credentials: LoginInput) => {
      const res = await fetch('/api/auth/login', {
        method: 'POST',
        body: JSON.stringify(credentials),
      });
      if (!res.ok) throw new Error('Login failed');
      return res.json();
    },
    onSuccess: (data) => {
      authClient.setToken(data.token);
      setUser(data.user);
      queryClient.invalidateQueries({ queryKey: ['auth', 'me'] });
      // Redirect handled by parent component
    },
  });
};
```

### C. Protected Routes (Middleware)
**Pattern: Server-side check at request time**

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('auth_token')?.value;
  const isAuthRoute = request.nextUrl.pathname.startsWith('/(auth)');

  if (!token && !isAuthRoute) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  if (token && isAuthRoute) {
    return NextResponse.redirect(new URL('/', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next|static).*)'],
};
```

### D. Query Keys Pattern
**Use consistent namespacing for auth queries:**

```typescript
// lib/query-keys.ts
export const authQueryKeys = {
  all: () => ['auth'] as const,
  me: () => [...authQueryKeys.all(), 'me'] as const,
  permissions: () => [...authQueryKeys.all(), 'permissions'] as const,
};
```

### E. Hydration Pattern
**Pattern: useEffect + localStorage in layout**

```typescript
// app/(main)/layout.tsx
'use client';

import { useEffect } from 'react';
import { useAuthStore } from '@/stores/auth.store';

export default function MainLayout({ children }) {
  const { setUser } = useAuthStore();

  useEffect(() => {
    // Hydrate from token on mount
    const token = localStorage.getItem('auth_token');
    if (token) {
      fetch('/api/auth/me')
        .then(r => r.json())
        .then(setUser)
        .catch(() => localStorage.removeItem('auth_token'));
    }
  }, []);

  return <>{children}</>;
}
```

---

## Recommended Enhancements

### Priority 1: Critical
1. **Token persistence** - Add localStorage/cookie handling
2. **Login form + mutation** - Implement useLoginMutation + form validation
3. **Middleware protection** - Add route middleware check
4. **Error handling** - 401/403 response interceptor in queryClient

### Priority 2: Important
1. **Auth context wrapper** - Consider Context for SSR layouts (optional with Zustand)
2. **Refresh token flow** - Auto-refresh on 401 response
3. **Logout mutation** - Clear token + invalidate queries
4. **useMe hook** - Query cache for `GET /api/auth/me`

### Priority 3: Enhancement
1. **Permission checks** - Component-level role guards
2. **Session timeout** - Auto-logout after inactivity
3. **Deep link handling** - Store redirect target before login

---

## Quick Reference: Complete Flow

```
1. User navigates to /login
   ├─ Login form rendered (no auth needed)
   └─ useLoginMutation() handles credentials

2. User submits credentials
   ├─ useMutation calls POST /api/auth/login
   ├─ MSW handler returns { user, token }
   ├─ onSuccess: setToken() + setUser() + invalidateQueries()
   └─ Redirect to /dashboard

3. User accesses protected route
   ├─ Middleware checks auth_token cookie/localStorage
   └─ If missing → redirect to /login

4. Protected page hydrates
   ├─ useEffect fetches GET /api/auth/me
   ├─ Zustand stores user in memory
   └─ Component renders with auth context
```

---

## Code Examples: Implementation Ready

### Login Page Form
```typescript
// (auth)/login/page.tsx
'use client';

import { useLoginMutation } from '@/hooks/mutations/use-login-mutation';
import { FormEvent, useState } from 'react';
import { useRouter } from 'next/navigation';

export default function LoginPage() {
  const router = useRouter();
  const mutation = useLoginMutation();
  const [credentials, setCredentials] = useState({ email: '', password: '' });

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    mutation.mutate(credentials, {
      onSuccess: () => router.push('/'),
    });
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input value={credentials.email} onChange={(e) => setCredentials({...credentials, email: e.target.value})} />
      <input type="password" value={credentials.password} onChange={(e) => setCredentials({...credentials, password: e.target.value})} />
      <button disabled={mutation.isPending}>{mutation.isPending ? 'Signing in...' : 'Sign in'}</button>
    </form>
  );
}
```

---

## Unresolved Questions
1. Should refresh token be stored (HttpOnly cookie only)?
2. Is multi-workspace support needed for permissions model?
3. Should useMe query auto-refetch or only on manual invalidation?
4. What is 401/403 error handling strategy (silent logout vs. toast)?
