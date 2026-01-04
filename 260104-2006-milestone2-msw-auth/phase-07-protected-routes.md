# Phase 7: Protected Routes (~45min)

## Context

- Plan: [plan.md](./plan.md)
- Depends on: [Phase 6](./phase-06-login-page.md)
- Issue: #63
- Research: [Auth Patterns - Protected Routes](./research/researcher-auth-patterns.md#c-protected-routes-middleware)

## Overview

Create auth-guard component for client-side route protection. Update (main) layout to wrap children with guard. Implement redirect to /login when no token.

## Requirements

1. Create AuthGuard component
2. Check token on mount and redirect if missing
3. Show loading state during check
4. Update (main)/layout.tsx to use AuthGuard
5. Handle hydration safely

## Architecture

### Client-Side Protection (not middleware)

For MSW mock environment, use client-side protection:

```
User navigates to /design
         ↓
AuthGuard mounts
         ↓
Check localStorage for token
         ↓
┌─────────────────────────────────────┐
│ Token exists?                       │
├─────────────────────────────────────┤
│ YES → Fetch /api/auth/me            │
│       → Success: Render children    │
│       → Fail: Redirect to /login    │
├─────────────────────────────────────┤
│ NO  → Redirect to /login            │
└─────────────────────────────────────┘
```

### Component Hierarchy

```
(main)/layout.tsx
└── AuthGuard
    └── SidebarProvider
        └── GlassSidebar
        └── SidebarInset
            └── AppHeader
            └── {children}
```

### Loading State

Show skeleton or loading spinner while checking auth:

```typescript
if (isLoading) {
  return <div className="min-h-screen flex items-center justify-center">
    <Spinner />
  </div>;
}
```

## Related Files

| File | Action |
|------|--------|
| `apps/web/src/components/auth/auth-guard.tsx` | CREATE |
| `apps/web/src/components/auth/index.ts` | CREATE |
| `apps/web/src/app/(main)/layout.tsx` | UPDATE |

## Implementation Steps

### Step 1: Create auth-guard.tsx

```typescript
// components/auth/auth-guard.tsx
'use client';

import { useEffect, useState } from 'react';
import { useRouter } from 'next/navigation';
import { authClient } from '@/lib/auth-client';
import { useCurrentUser } from '@/hooks/use-auth';

interface AuthGuardProps {
  children: React.ReactNode;
}

export function AuthGuard({ children }: AuthGuardProps) {
  const router = useRouter();
  const [isChecking, setIsChecking] = useState(true);
  const { data: user, isLoading, isError } = useCurrentUser();

  useEffect(() => {
    // No token = redirect immediately
    if (!authClient.hasToken()) {
      router.replace('/login');
      return;
    }
    setIsChecking(false);
  }, [router]);

  useEffect(() => {
    // Auth check failed = redirect
    if (isError) {
      authClient.clearToken();
      router.replace('/login');
    }
  }, [isError, router]);

  // Still checking token existence
  if (isChecking) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="animate-pulse text-muted-foreground">Loading...</div>
      </div>
    );
  }

  // Fetching user data
  if (isLoading) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="animate-pulse text-muted-foreground">Loading...</div>
      </div>
    );
  }

  // User loaded successfully
  if (user) {
    return <>{children}</>;
  }

  // Fallback - should not reach here
  return null;
}
```

### Step 2: Create auth/index.ts

```typescript
// components/auth/index.ts
export { AuthGuard } from './auth-guard';
```

### Step 3: Update (main)/layout.tsx

```typescript
// app/(main)/layout.tsx
import { SidebarProvider, SidebarInset } from "@/components/ui/sidebar";
import { GlassSidebar, AppHeader } from "@/components/layout";
import { AuthGuard } from "@/components/auth";

export default function MainLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <AuthGuard>
      <SidebarProvider>
        <GlassSidebar />
        <SidebarInset>
          <AppHeader />
          <main className="flex-1 p-6">{children}</main>
        </SidebarInset>
      </SidebarProvider>
    </AuthGuard>
  );
}
```

### Step 4: Test protection flow

1. Clear localStorage
2. Navigate to /design/workspace
3. Verify redirect to /login
4. Login with test credentials
5. Verify redirect to / or original route
6. Refresh page - should stay logged in
7. Clear token manually - should redirect

## Alternative: Simpler Guard (if useCurrentUser causes issues)

```typescript
export function AuthGuard({ children }: AuthGuardProps) {
  const router = useRouter();
  const [isReady, setIsReady] = useState(false);

  useEffect(() => {
    if (!authClient.hasToken()) {
      router.replace('/login');
    } else {
      setIsReady(true);
    }
  }, [router]);

  if (!isReady) {
    return <div className="min-h-screen flex items-center justify-center">
      <div className="animate-pulse">Loading...</div>
    </div>;
  }

  return <>{children}</>;
}
```

## Todo List

- [ ] Create components/auth directory
- [ ] Create auth-guard.tsx
- [ ] Create auth/index.ts export
- [ ] Update (main)/layout.tsx with AuthGuard
- [ ] Test: No token → redirect to /login
- [ ] Test: Valid token → render children
- [ ] Test: Invalid token → redirect to /login
- [ ] Test: Page refresh maintains auth
- [ ] Verify no hydration errors

## Success Criteria

- [ ] AuthGuard component created
- [ ] Protected routes redirect to /login when no token
- [ ] Protected routes render when token valid
- [ ] Loading state shown during auth check
- [ ] No hydration mismatch errors
- [ ] Page refresh maintains authenticated state
- [ ] Token removal triggers redirect
