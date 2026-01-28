---
phase: 1
title: "Setup Navigation and Guards"
status: pending
effort: 1.5h
---

# Phase 1: Setup Navigation and Guards

## Context Links

- [Navigation config](../../apps/web/src/lib/navigation.ts)
- [AuthGuard](../../apps/web/src/components/auth/auth-guard.tsx)
- [UserMenu](../../apps/web/src/components/layout/user-menu.tsx)
- [Mock users](../../apps/web/src/mocks/data/users.data.ts)

## Overview

- **Priority**: High (blocking for other phases)
- **Status**: Pending
- **Description**: Install Recharts, create AdminGuard component, add admin navigation config, update UserMenu to show "Admin Portal" for admin users

## Key Insights

- Existing `AuthGuard` uses `useAuth()` hook which returns `user.role`
- Mock users already have `role: 'ADMIN' | 'STAFF' | 'DESIGNER'`
- Navigation config uses `NavItem[]` with icons from lucide-react

## Requirements

### Functional
- Install `recharts` package for dashboard charts
- AdminGuard redirects non-admin users to home (`/`)
- "Admin Portal" menu item appears ONLY for users with `role === 'ADMIN'`
- Admin navigation config includes: Dashboard, Spaces, Users

### Non-functional
- Reuse existing auth patterns
- Type-safe navigation config

## Architecture

```
AdminGuard
├── Check user.role === 'ADMIN'
├── If not admin → redirect to '/'
└── If admin → render children

UserMenu
├── Existing items (Profile, Settings, Logout)
└── NEW: "Admin Portal" (conditional on user.role === 'ADMIN')
```

## Related Code Files

### Files to Create
- `apps/web/src/components/auth/admin-guard.tsx`

### Files to Modify
- `apps/web/src/lib/navigation.ts` - Add `adminNavigation` export
- `apps/web/src/components/auth/index.ts` - Export AdminGuard
- `apps/web/src/components/layout/user-menu.tsx` - Add Admin Portal link

## Implementation Steps

### Step 1: Install Recharts

```bash
cd apps/web && pnpm add recharts
```

### Step 2: Create AdminGuard Component

Create `apps/web/src/components/auth/admin-guard.tsx`:

```tsx
'use client';

import { useEffect } from 'react';
import { useRouter, usePathname } from 'next/navigation';
import { useAuth } from '@/hooks/use-auth';
import { hasToken } from '@/lib/auth-client';
import { Loader2 } from 'lucide-react';

interface AdminGuardProps {
  children: React.ReactNode;
}

export function AdminGuard({ children }: AdminGuardProps) {
  const router = useRouter();
  const pathname = usePathname();
  const { user, isLoading, isAuthenticated } = useAuth();

  useEffect(() => {
    if (isLoading) return;

    // Not authenticated at all -> login
    if (!hasToken() && !isAuthenticated) {
      const callbackUrl = encodeURIComponent(pathname);
      router.replace(`/login?callbackUrl=${callbackUrl}`);
      return;
    }

    // Authenticated but not admin -> home
    if (user && user.role !== 'ADMIN') {
      router.replace('/');
    }
  }, [isLoading, isAuthenticated, user, router, pathname]);

  // Loading state
  if (isLoading || (!isAuthenticated && hasToken())) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <Loader2 className="h-8 w-8 animate-spin text-primary" />
      </div>
    );
  }

  // Not authenticated
  if (!isAuthenticated && !hasToken()) {
    return null;
  }

  // Not admin
  if (user && user.role !== 'ADMIN') {
    return null;
  }

  return <>{children}</>;
}
```

### Step 3: Export AdminGuard

Update `apps/web/src/components/auth/index.ts`:

```ts
export * from './auth-guard';
export * from './admin-guard';
```

### Step 4: Add Admin Navigation Config

Update `apps/web/src/lib/navigation.ts`:

```ts
import {
  LayoutDashboard,
  Package,
  Palette,
  Settings,
  Users,
  Boxes,
  Shield,
  type LucideIcon,
} from "lucide-react";

// ... existing NavItem interface and mainNavigation ...

// Admin portal navigation
export const adminNavigation: NavItem[] = [
  {
    title: "Dashboard",
    href: "/admin",
    icon: LayoutDashboard,
  },
  {
    title: "Spaces",
    href: "/admin/spaces",
    icon: Boxes,
  },
  {
    title: "Users",
    href: "/admin/users",
    icon: Users,
  },
];

// Admin route labels for breadcrumbs
export const adminRouteLabels: Record<string, string> = {
  "/admin": "Dashboard",
  "/admin/spaces": "Spaces",
  "/admin/users": "Users",
};
```

### Step 5: Update UserMenu with Admin Portal Link

Update `apps/web/src/components/layout/user-menu.tsx`:

Add after the Settings menu item (before the separator that precedes Logout):

```tsx
// Add import
import { Shield } from 'lucide-react';

// Inside the DropdownMenuContent, add after Settings item:
{user.role === 'ADMIN' && (
  <>
    <DropdownMenuSeparator />
    <DropdownMenuItem onClick={() => router.push('/admin')}>
      <Shield className="mr-2 h-4 w-4" />
      Admin Portal
    </DropdownMenuItem>
  </>
)}
```

Full UserMenu component structure:
```tsx
<DropdownMenuContent align="end" className="w-56">
  <DropdownMenuLabel>...</DropdownMenuLabel>
  <DropdownMenuSeparator />
  <DropdownMenuItem>Profile</DropdownMenuItem>
  <DropdownMenuItem>Settings</DropdownMenuItem>
  {user.role === 'ADMIN' && (
    <>
      <DropdownMenuSeparator />
      <DropdownMenuItem onClick={() => router.push('/admin')}>
        <Shield className="mr-2 h-4 w-4" />
        Admin Portal
      </DropdownMenuItem>
    </>
  )}
  <DropdownMenuSeparator />
  <DropdownMenuItem>Log out</DropdownMenuItem>
</DropdownMenuContent>
```

## Todo List

- [ ] Install recharts: `cd apps/web && pnpm add recharts`
- [ ] Create AdminGuard component
- [ ] Export AdminGuard from auth/index.ts
- [ ] Add adminNavigation to navigation.ts
- [ ] Add adminRouteLabels to navigation.ts
- [ ] Update UserMenu with Admin Portal link (conditional on ADMIN role)
- [ ] Run `pnpm build` to verify no compile errors

## Success Criteria

- [ ] `recharts` is in package.json dependencies
- [ ] AdminGuard redirects non-admin users to `/`
- [ ] AdminGuard shows loading spinner during auth check
- [ ] "Admin Portal" appears in UserMenu ONLY for admin users
- [ ] `adminNavigation` and `adminRouteLabels` exported from navigation.ts
- [ ] Build passes with no TypeScript errors

## Security Considerations

- AdminGuard must check BOTH authentication AND admin role
- Redirect to `/` for non-admin (not to login, since they're authenticated)
- Guard runs on client side; server-side protection needed for real API

## Next Steps

After completing Phase 1:
- Phase 2: Create admin layout and header components
- Admin layout will wrap routes with AdminGuard
