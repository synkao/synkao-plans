# Phase 5: Auth Hooks (~45min)

## Context

- Plan: [plan.md](./plan.md)
- Depends on: [Phase 4](./phase-04-verification.md)
- Issue: #63
- Research: [Auth Patterns](./research/researcher-auth-patterns.md)

## Overview

Create auth hooks using TanStack Query mutations and Zustand store integration. Implement localStorage token persistence.

## Requirements

1. Create auth-client.ts for token storage
2. Create use-auth.ts with useLogin, useLogout, useCurrentUser
3. Connect mutations to auth.store.ts
4. Handle success/error states

## Architecture

### Token Storage (localStorage only)

```typescript
// lib/auth-client.ts
const TOKEN_KEY = 'auth_token';

export const authClient = {
  setToken: (token: string) => localStorage.setItem(TOKEN_KEY, token),
  getToken: () => localStorage.getItem(TOKEN_KEY),
  clearToken: () => localStorage.removeItem(TOKEN_KEY),
  hasToken: () => !!localStorage.getItem(TOKEN_KEY),
};
```

### Auth Hooks Flow

```
useLogin()
  → POST /api/auth/login
  → onSuccess: authClient.setToken() + setUser()
  → return mutation

useLogout()
  → POST /api/auth/logout
  → onSuccess: authClient.clearToken() + clearAuth()
  → return mutation

useCurrentUser()
  → GET /api/auth/me (with Authorization header)
  → onSuccess: setUser()
  → return query
```

### Zustand Integration

```typescript
// hooks/use-auth.ts
import { useAuthStore } from '@/stores/auth.store';

const { setUser, clearAuth } = useAuthStore.getState();
```

## Related Files

| File | Action |
|------|--------|
| `apps/web/src/lib/auth-client.ts` | CREATE |
| `apps/web/src/hooks/use-auth.ts` | CREATE |
| `apps/web/src/stores/auth.store.ts` | VERIFY (no changes) |
| `apps/web/src/lib/query-client.ts` | UPDATE (add auth header) |

## Implementation Steps

### Step 1: Create auth-client.ts

```typescript
// lib/auth-client.ts
const TOKEN_KEY = 'auth_token';

export const authClient = {
  setToken: (token: string) => {
    if (typeof window !== 'undefined') {
      localStorage.setItem(TOKEN_KEY, token);
    }
  },

  getToken: () => {
    if (typeof window !== 'undefined') {
      return localStorage.getItem(TOKEN_KEY);
    }
    return null;
  },

  clearToken: () => {
    if (typeof window !== 'undefined') {
      localStorage.removeItem(TOKEN_KEY);
    }
  },

  hasToken: () => {
    if (typeof window !== 'undefined') {
      return !!localStorage.getItem(TOKEN_KEY);
    }
    return false;
  },
};
```

### Step 2: Create use-auth.ts

```typescript
// hooks/use-auth.ts
import { useMutation, useQuery, useQueryClient } from '@tanstack/react-query';
import { useAuthStore } from '@/stores/auth.store';
import { authClient } from '@/lib/auth-client';

interface LoginInput {
  email: string;
  password: string;
}

interface LoginResponse {
  user: User;
  token: string;
}

export function useLogin() {
  const queryClient = useQueryClient();
  const setUser = useAuthStore((s) => s.setUser);

  return useMutation({
    mutationFn: async (input: LoginInput): Promise<LoginResponse> => {
      const res = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(input),
      });
      if (!res.ok) throw new Error('Login failed');
      return res.json();
    },
    onSuccess: (data) => {
      authClient.setToken(data.token);
      setUser(data.user);
      queryClient.invalidateQueries({ queryKey: ['auth'] });
    },
  });
}

export function useLogout() {
  const queryClient = useQueryClient();
  const clearAuth = useAuthStore((s) => s.clearAuth);

  return useMutation({
    mutationFn: async () => {
      await fetch('/api/auth/logout', { method: 'POST' });
    },
    onSuccess: () => {
      authClient.clearToken();
      clearAuth();
      queryClient.clear();
    },
  });
}

export function useCurrentUser() {
  const setUser = useAuthStore((s) => s.setUser);

  return useQuery({
    queryKey: ['auth', 'me'],
    queryFn: async () => {
      const token = authClient.getToken();
      if (!token) throw new Error('No token');

      const res = await fetch('/api/auth/me', {
        headers: { Authorization: `Bearer ${token}` },
      });
      if (!res.ok) throw new Error('Unauthorized');
      return res.json();
    },
    enabled: authClient.hasToken(),
    retry: false,
    onSuccess: (user) => setUser(user),
  });
}
```

### Step 3: Add auth query keys (optional)

```typescript
// lib/query-keys.ts
export const authKeys = {
  all: () => ['auth'] as const,
  me: () => [...authKeys.all(), 'me'] as const,
};
```

### Step 4: Test hooks

Create simple test in browser console or React DevTools:
1. Call `useLogin` with test credentials
2. Verify token in localStorage
3. Call `useCurrentUser` to fetch user
4. Call `useLogout` to clear

## Todo List

- [ ] Create auth-client.ts
- [ ] Create use-auth.ts with useLogin
- [ ] Add useLogout mutation
- [ ] Add useCurrentUser query
- [ ] Handle SSR guard (typeof window check)
- [ ] Export hooks from hooks/index.ts (if exists)
- [ ] Test login flow manually

## Success Criteria

- [ ] authClient.setToken stores in localStorage
- [ ] authClient.getToken retrieves token
- [ ] useLogin mutation calls POST /api/auth/login
- [ ] useLogin onSuccess sets token + updates store
- [ ] useLogout clears token + clears store
- [ ] useCurrentUser fetches with Authorization header
- [ ] No SSR errors (typeof window checks)
