---
phase: 4
title: "Placeholder Pages"
status: pending
effort: 1h
depends_on: [2]
---

# Phase 4: Placeholder Pages

## Context Links

- [Phase 2](./phase-02-layout-and-header-components.md) - Admin layout
- [PageHeader](../../apps/web/src/components/layout/page-header.tsx)

## Overview

- **Priority**: Low
- **Status**: Pending
- **Description**: Create placeholder pages for Spaces management (/admin/spaces, /admin/spaces/[id]) and Users management (/admin/users, /admin/users/[id])

## Key Insights

- Pages are placeholders for future implementation
- Use consistent layout pattern with header + empty state
- Dynamic routes ([id]) need params handling
- Can reuse existing empty state patterns

## Requirements

### Functional
- /admin/spaces - Shows "Spaces Management" header with placeholder content
- /admin/spaces/[id] - Shows "Space Details" with ID from params
- /admin/users - Shows "Users Management" header with placeholder content
- /admin/users/[id] - Shows "User Details" with ID from params

### Non-functional
- Consistent styling with dashboard page
- Clear "Coming Soon" messaging
- Include breadcrumb hints for future navigation

## Architecture

```
(admin)/
├── spaces/
│   ├── page.tsx       # Spaces list
│   └── [id]/
│       └── page.tsx   # Space detail
└── users/
    ├── page.tsx       # Users list
    └── [id]/
        └── page.tsx   # User detail
```

## Related Code Files

### Files to Create
- `apps/web/src/app/(admin)/spaces/page.tsx`
- `apps/web/src/app/(admin)/spaces/[id]/page.tsx`
- `apps/web/src/app/(admin)/users/page.tsx`
- `apps/web/src/app/(admin)/users/[id]/page.tsx`

## Implementation Steps

### Step 1: Create Spaces List Page

Create `apps/web/src/app/(admin)/spaces/page.tsx`:

```tsx
import { Boxes, Plus } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Card, CardContent } from '@/components/ui/card';

export default function AdminSpacesPage() {
  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div>
          <h1 className="text-2xl font-heading font-bold">Spaces</h1>
          <p className="text-muted-foreground">
            Manage workspaces and their configurations
          </p>
        </div>
        <Button disabled>
          <Plus className="mr-2 h-4 w-4" />
          Create Space
        </Button>
      </div>

      <Card>
        <CardContent className="flex flex-col items-center justify-center py-16 text-center">
          <Boxes className="h-12 w-12 text-muted-foreground/50 mb-4" />
          <h3 className="text-lg font-semibold">Spaces Management</h3>
          <p className="text-muted-foreground max-w-sm mt-2">
            This page will display all workspaces with options to view, edit, and manage their settings.
          </p>
          <p className="text-xs text-muted-foreground mt-4">Coming Soon</p>
        </CardContent>
      </Card>
    </div>
  );
}
```

### Step 2: Create Space Detail Page

Create `apps/web/src/app/(admin)/spaces/[id]/page.tsx`:

```tsx
import { ArrowLeft, Boxes, Settings } from 'lucide-react';
import Link from 'next/link';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';

interface SpaceDetailPageProps {
  params: Promise<{ id: string }>;
}

export default async function AdminSpaceDetailPage({ params }: SpaceDetailPageProps) {
  const { id } = await params;

  return (
    <div className="space-y-6">
      <div className="flex items-center gap-4">
        <Button variant="ghost" size="icon" asChild>
          <Link href="/admin/spaces">
            <ArrowLeft className="h-4 w-4" />
          </Link>
        </Button>
        <div className="flex-1">
          <h1 className="text-2xl font-heading font-bold">Space Details</h1>
          <p className="text-muted-foreground">
            Space ID: {id}
          </p>
        </div>
        <Button variant="outline" disabled>
          <Settings className="mr-2 h-4 w-4" />
          Settings
        </Button>
      </div>

      <div className="grid gap-6 md:grid-cols-2">
        <Card>
          <CardHeader>
            <CardTitle className="text-base">Space Information</CardTitle>
          </CardHeader>
          <CardContent className="flex flex-col items-center justify-center py-8 text-center">
            <Boxes className="h-10 w-10 text-muted-foreground/50 mb-3" />
            <p className="text-sm text-muted-foreground">
              Space details will be displayed here
            </p>
            <p className="text-xs text-muted-foreground mt-2">Coming Soon</p>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle className="text-base">Members</CardTitle>
          </CardHeader>
          <CardContent className="flex flex-col items-center justify-center py-8 text-center">
            <p className="text-sm text-muted-foreground">
              Space members list will be shown here
            </p>
            <p className="text-xs text-muted-foreground mt-2">Coming Soon</p>
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
```

### Step 3: Create Users List Page

Create `apps/web/src/app/(admin)/users/page.tsx`:

```tsx
import { Users, Plus, Search } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Card, CardContent } from '@/components/ui/card';
import { Input } from '@/components/ui/input';

export default function AdminUsersPage() {
  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div>
          <h1 className="text-2xl font-heading font-bold">Users</h1>
          <p className="text-muted-foreground">
            Manage user accounts and permissions
          </p>
        </div>
        <Button disabled>
          <Plus className="mr-2 h-4 w-4" />
          Add User
        </Button>
      </div>

      <div className="flex items-center gap-4">
        <div className="relative flex-1 max-w-sm">
          <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-muted-foreground" />
          <Input
            placeholder="Search users..."
            className="pl-9"
            disabled
          />
        </div>
      </div>

      <Card>
        <CardContent className="flex flex-col items-center justify-center py-16 text-center">
          <Users className="h-12 w-12 text-muted-foreground/50 mb-4" />
          <h3 className="text-lg font-semibold">Users Management</h3>
          <p className="text-muted-foreground max-w-sm mt-2">
            This page will display all users with options to view profiles, manage roles, and control access permissions.
          </p>
          <p className="text-xs text-muted-foreground mt-4">Coming Soon</p>
        </CardContent>
      </Card>
    </div>
  );
}
```

### Step 4: Create User Detail Page

Create `apps/web/src/app/(admin)/users/[id]/page.tsx`:

```tsx
import { ArrowLeft, User, Shield, Mail } from 'lucide-react';
import Link from 'next/link';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Avatar, AvatarFallback } from '@/components/ui/avatar';

interface UserDetailPageProps {
  params: Promise<{ id: string }>;
}

export default async function AdminUserDetailPage({ params }: UserDetailPageProps) {
  const { id } = await params;

  return (
    <div className="space-y-6">
      <div className="flex items-center gap-4">
        <Button variant="ghost" size="icon" asChild>
          <Link href="/admin/users">
            <ArrowLeft className="h-4 w-4" />
          </Link>
        </Button>
        <div className="flex-1">
          <h1 className="text-2xl font-heading font-bold">User Details</h1>
          <p className="text-muted-foreground">
            User ID: {id}
          </p>
        </div>
        <Button variant="outline" disabled>
          Edit User
        </Button>
      </div>

      <div className="grid gap-6 md:grid-cols-3">
        <Card className="md:col-span-1">
          <CardHeader>
            <CardTitle className="text-base">Profile</CardTitle>
          </CardHeader>
          <CardContent className="flex flex-col items-center text-center">
            <Avatar className="h-20 w-20 mb-4">
              <AvatarFallback className="text-2xl bg-primary/10 text-primary">
                ?
              </AvatarFallback>
            </Avatar>
            <p className="text-sm text-muted-foreground">
              User profile will be displayed here
            </p>
            <p className="text-xs text-muted-foreground mt-2">Coming Soon</p>
          </CardContent>
        </Card>

        <Card className="md:col-span-2">
          <CardHeader>
            <CardTitle className="text-base">Account Information</CardTitle>
          </CardHeader>
          <CardContent className="space-y-4">
            <div className="flex items-center gap-3 p-3 rounded-lg border">
              <Mail className="h-5 w-5 text-muted-foreground" />
              <div>
                <p className="text-xs text-muted-foreground">Email</p>
                <p className="text-sm">Will be loaded from API</p>
              </div>
            </div>
            <div className="flex items-center gap-3 p-3 rounded-lg border">
              <Shield className="h-5 w-5 text-muted-foreground" />
              <div>
                <p className="text-xs text-muted-foreground">Role</p>
                <p className="text-sm">Will be loaded from API</p>
              </div>
            </div>
            <div className="flex items-center gap-3 p-3 rounded-lg border">
              <User className="h-5 w-5 text-muted-foreground" />
              <div>
                <p className="text-xs text-muted-foreground">Status</p>
                <p className="text-sm">Will be loaded from API</p>
              </div>
            </div>
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
```

## Todo List

- [ ] Create (admin)/spaces/page.tsx
- [ ] Create (admin)/spaces/[id]/page.tsx
- [ ] Create (admin)/users/page.tsx
- [ ] Create (admin)/users/[id]/page.tsx
- [ ] Test navigation between pages
- [ ] Test dynamic route params work
- [ ] Test back button navigation
- [ ] Run `pnpm build` to verify no errors

## Success Criteria

- [ ] /admin/spaces displays Spaces Management placeholder
- [ ] /admin/spaces/123 displays Space Details with ID "123"
- [ ] /admin/users displays Users Management placeholder
- [ ] /admin/users/abc displays User Details with ID "abc"
- [ ] Back buttons navigate correctly
- [ ] All pages use consistent styling
- [ ] Build passes

## Security Considerations

- All pages protected by AdminGuard in layout
- Dynamic IDs should be validated when real API is added

## Next Steps

After completing Phase 4:
- Feature complete for Admin Portal MVP
- Future enhancements:
  - Implement real data fetching
  - Add CRUD operations for spaces/users
  - Add search/filter functionality
  - Add pagination for lists
