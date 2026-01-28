---
phase: 2
title: "Layout and Header Components"
status: pending
effort: 1.5h
depends_on: [1]
---

# Phase 2: Layout and Header Components

## Context Links

- [Phase 1](./phase-01-setup-navigation-and-guards.md)
- [Main layout](../../apps/web/src/app/(main)/layout.tsx)
- [AppHeader](../../apps/web/src/components/layout/app-header.tsx)
- [Navigation config](../../apps/web/src/lib/navigation.ts)

## Overview

- **Priority**: High
- **Status**: Pending
- **Description**: Create admin-specific layout with horizontal top navigation, AdminHeader with nav links and "Back to App" button

## Key Insights

- Main app uses sidebar layout (SidebarProvider + GlassSidebar)
- Admin portal uses horizontal top nav (different pattern)
- Share existing providers (QueryProvider, theme) via root layout
- AdminGuard wraps the layout

## Requirements

### Functional
- Admin layout wraps all `/admin/*` routes
- AdminHeader shows: Logo, nav links, "Back to App" button, user avatar
- Navigation uses `adminNavigation` config
- Active route highlighted in nav
- Mobile: hamburger menu or responsive collapse

### Non-functional
- Consistent glassmorphism styling with main app
- Mobile-first responsive design
- Use existing UI components (Button, Avatar, DropdownMenu)

## Architecture

```
(admin)/layout.tsx
├── AdminGuard (from Phase 1)
└── <div className="min-h-screen">
    ├── AdminHeader
    │   ├── Logo
    │   ├── AdminNav (horizontal links)
    │   ├── "Back to App" button
    │   └── UserAvatar
    └── <main>{children}</main>

AdminHeader (fixed top)
├── Logo (links to /admin)
├── AdminNav (Desktop: inline links, Mobile: dropdown)
├── Spacer
├── "Back to App" button
└── User avatar/dropdown
```

## Related Code Files

### Files to Create
- `apps/web/src/app/(admin)/layout.tsx`
- `apps/web/src/components/layout/admin-header.tsx`
- `apps/web/src/components/layout/admin-nav.tsx`

### Files to Modify
- `apps/web/src/components/layout/index.ts` - Export new components

## Implementation Steps

### Step 1: Create AdminNav Component

Create `apps/web/src/components/layout/admin-nav.tsx`:

```tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';
import { cn } from '@/lib/utils';
import { adminNavigation } from '@/lib/navigation';

export function AdminNav() {
  const pathname = usePathname();

  return (
    <nav className="hidden md:flex items-center gap-1">
      {adminNavigation.map((item) => {
        const isActive = pathname === item.href ||
          (item.href !== '/admin' && pathname.startsWith(item.href));
        const Icon = item.icon;

        return (
          <Link
            key={item.href}
            href={item.href}
            className={cn(
              'flex items-center gap-2 px-3 py-2 text-sm font-medium rounded-md transition-colors',
              isActive
                ? 'bg-primary/10 text-primary'
                : 'text-muted-foreground hover:text-foreground hover:bg-muted'
            )}
          >
            <Icon className="h-4 w-4" />
            {item.title}
          </Link>
        );
      })}
    </nav>
  );
}
```

### Step 2: Create AdminHeader Component

Create `apps/web/src/components/layout/admin-header.tsx`:

```tsx
'use client';

import Link from 'next/link';
import { useRouter } from 'next/navigation';
import { ArrowLeft, Menu } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import {
  Sheet,
  SheetContent,
  SheetHeader,
  SheetTitle,
  SheetTrigger,
} from '@/components/ui/sheet';
import { useAuth, useLogout } from '@/hooks/use-auth';
import { AdminNav } from './admin-nav';
import { adminNavigation } from '@/lib/navigation';
import { cn } from '@/lib/utils';
import { usePathname } from 'next/navigation';

export function AdminHeader() {
  const router = useRouter();
  const pathname = usePathname();
  const { user } = useAuth();
  const logoutMutation = useLogout();

  const handleLogout = async () => {
    await logoutMutation.mutateAsync();
    router.push('/login');
  };

  const initials = user?.name
    .split(' ')
    .map((n) => n[0])
    .join('')
    .toUpperCase()
    .slice(0, 2) || '';

  return (
    <header className="sticky top-0 z-50 w-full border-b bg-background/95 backdrop-blur supports-[backdrop-filter]:bg-background/60">
      <div className="container flex h-14 items-center gap-4">
        {/* Mobile menu */}
        <Sheet>
          <SheetTrigger asChild>
            <Button variant="ghost" size="icon" className="md:hidden">
              <Menu className="h-5 w-5" />
              <span className="sr-only">Toggle menu</span>
            </Button>
          </SheetTrigger>
          <SheetContent side="left" className="w-64">
            <SheetHeader>
              <SheetTitle>Admin Portal</SheetTitle>
            </SheetHeader>
            <nav className="mt-6 flex flex-col gap-1">
              {adminNavigation.map((item) => {
                const isActive = pathname === item.href ||
                  (item.href !== '/admin' && pathname.startsWith(item.href));
                const Icon = item.icon;

                return (
                  <Link
                    key={item.href}
                    href={item.href}
                    className={cn(
                      'flex items-center gap-3 px-3 py-2 text-sm font-medium rounded-md transition-colors',
                      isActive
                        ? 'bg-primary/10 text-primary'
                        : 'text-muted-foreground hover:text-foreground hover:bg-muted'
                    )}
                  >
                    <Icon className="h-4 w-4" />
                    {item.title}
                  </Link>
                );
              })}
            </nav>
          </SheetContent>
        </Sheet>

        {/* Logo */}
        <Link href="/admin" className="flex items-center gap-2 font-heading font-semibold">
          <span className="hidden sm:inline">Admin Portal</span>
          <span className="sm:hidden">Admin</span>
        </Link>

        {/* Desktop nav */}
        <AdminNav />

        {/* Spacer */}
        <div className="flex-1" />

        {/* Back to App */}
        <Button
          variant="ghost"
          size="sm"
          onClick={() => router.push('/')}
          className="gap-2"
        >
          <ArrowLeft className="h-4 w-4" />
          <span className="hidden sm:inline">Back to App</span>
        </Button>

        {/* User menu */}
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="ghost" size="icon" className="rounded-full">
              <Avatar className="h-8 w-8">
                <AvatarImage src={user?.avatar} alt={user?.name} />
                <AvatarFallback className="bg-primary/10 text-primary text-xs">
                  {initials}
                </AvatarFallback>
              </Avatar>
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end" className="w-56">
            <DropdownMenuLabel>
              <div className="flex flex-col">
                <span>{user?.name}</span>
                <span className="text-xs text-muted-foreground font-normal">
                  {user?.email}
                </span>
              </div>
            </DropdownMenuLabel>
            <DropdownMenuSeparator />
            <DropdownMenuItem onClick={() => router.push('/')}>
              Back to App
            </DropdownMenuItem>
            <DropdownMenuSeparator />
            <DropdownMenuItem
              className="text-destructive"
              onClick={handleLogout}
              disabled={logoutMutation.isPending}
            >
              Log out
            </DropdownMenuItem>
          </DropdownMenuContent>
        </DropdownMenu>
      </div>
    </header>
  );
}
```

### Step 3: Create Admin Layout

Create `apps/web/src/app/(admin)/layout.tsx`:

```tsx
import { AdminGuard } from '@/components/auth';
import { AdminHeader } from '@/components/layout/admin-header';

export default function AdminLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <AdminGuard>
      <div className="min-h-screen flex flex-col">
        <AdminHeader />
        <main className="flex-1 container py-6">{children}</main>
      </div>
    </AdminGuard>
  );
}
```

### Step 4: Update Layout Exports

Update `apps/web/src/components/layout/index.ts`:

```ts
export { GlassSidebar } from "./glass-sidebar";
export { SidebarNav } from "./sidebar-nav";
export { AppHeader } from "./app-header";
export { PageHeader, type BreadcrumbItem } from "./page-header";
export { UserMenu } from "./user-menu";
export { NotificationBell } from "./notification-bell";
export { Logo } from "./logo";
export { ContentArea, type ContentAreaProps } from "./content-area";
export { AdminHeader } from "./admin-header";
export { AdminNav } from "./admin-nav";
```

### Step 5: Create Placeholder Admin Dashboard Page

Create `apps/web/src/app/(admin)/page.tsx`:

```tsx
export default function AdminDashboardPage() {
  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-heading font-bold">Dashboard</h1>
        <p className="text-muted-foreground">Admin portal overview</p>
      </div>
      <div className="rounded-lg border bg-card p-8 text-center text-muted-foreground">
        Dashboard content coming in Phase 3
      </div>
    </div>
  );
}
```

## Todo List

- [ ] Create admin-nav.tsx component
- [ ] Create admin-header.tsx component
- [ ] Create (admin)/layout.tsx with AdminGuard
- [ ] Create placeholder (admin)/page.tsx
- [ ] Update layout/index.ts exports
- [ ] Test navigation links work correctly
- [ ] Test mobile hamburger menu
- [ ] Test "Back to App" button
- [ ] Verify non-admin users get redirected
- [ ] Run `pnpm build` to verify no errors

## Success Criteria

- [ ] Admin layout renders with AdminHeader
- [ ] Desktop shows inline nav links
- [ ] Mobile shows hamburger menu with Sheet
- [ ] Active route is highlighted
- [ ] "Back to App" navigates to `/`
- [ ] User avatar dropdown works
- [ ] Non-admin users redirected to `/`
- [ ] Build passes

## Security Considerations

- AdminGuard in layout ensures ALL admin pages are protected
- Double-check redirect logic for edge cases

## Next Steps

After completing Phase 2:
- Phase 3: Implement dashboard with stats cards and charts
