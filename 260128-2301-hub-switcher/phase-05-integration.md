# Phase 5: Integration

## Context Links
- [Main Plan](./plan.md)
- [Phase 4: Hub Switcher Components](./phase-04-hub-switcher-components.md)
- [glass-sidebar.tsx](../../apps/web/src/components/layout/glass-sidebar.tsx)
- [Main layout](../../apps/web/src/app/(main)/layout.tsx)

## Overview
- **Priority**: P1 (Final integration)
- **Status**: ⚠️ completed with issues
- **Effort**: 1h
- **Dependencies**: Phase 4 completed
- **Completed**: 2026-01-28

## Key Insights
- HubSwitcher placed in SidebarHeader below Logo
- Redirect logic: no hub selected -> /select-hub
- Exception: /select-hub page itself should not redirect
- Use AuthGuard pattern for hub check

## Requirements

### Functional
- Add HubSwitcher to sidebar header
- Redirect to /select-hub if no hub selected
- Exception: don't redirect on /select-hub page
- Clear hub on logout

### Non-functional
- Smooth integration with existing layout
- No layout shift during hydration

## Architecture

```
Integration points:
1. glass-sidebar.tsx - Add HubSwitcher in header
2. (main)/layout.tsx - Add hub redirect logic (or create HubGuard)
3. auth.store.ts - Clear hub on logout (optional)
```

## Related Code Files

### Files to Modify
1. `apps/web/src/components/layout/glass-sidebar.tsx` - Add HubSwitcher
2. `apps/web/src/components/layout/index.ts` - Export HubSwitcher (optional)
3. Create `apps/web/src/components/hub/hub-guard.tsx` - Redirect logic

### Files to Create
1. `apps/web/src/components/hub/hub-guard.tsx` - Hub check guard

## Implementation Steps

### Step 1: Update Glass Sidebar
Update `apps/web/src/components/layout/glass-sidebar.tsx`:

```tsx
"use client";

import {
  Sidebar,
  SidebarContent,
  SidebarFooter,
  SidebarHeader,
  SidebarRail,
  useSidebar,
} from "@/components/ui/sidebar";
import { Logo } from "./logo";
import { SidebarNav } from "./sidebar-nav";
import { UserMenu } from "./user-menu";
import { Separator } from "@/components/ui/separator";
import { HubSwitcher } from "@/components/hub";

export function GlassSidebar() {
  const { state } = useSidebar();
  const collapsed = state === "collapsed";

  return (
    <Sidebar
      collapsible="icon"
      className="border-r border-sidebar-border bg-white/60 backdrop-blur-[12px]"
    >
      <SidebarHeader className="p-4">
        <Logo collapsed={collapsed} />
        {/* Hub Switcher below logo */}
        <div className="mt-3">
          <HubSwitcher collapsed={collapsed} />
        </div>
      </SidebarHeader>
      <Separator className="mx-4 w-auto" />
      <SidebarContent className="p-2">
        <SidebarNav />
      </SidebarContent>
      <Separator className="mx-4 w-auto" />
      <SidebarFooter className="p-2">
        <UserMenu collapsed={collapsed} />
      </SidebarFooter>
      <SidebarRail />
    </Sidebar>
  );
}
```

### Step 2: Create Hub Guard Component
Create `apps/web/src/components/hub/hub-guard.tsx`:

```tsx
"use client";

import { useEffect } from "react";
import { useRouter, usePathname } from "next/navigation";
import { useCurrentHub, useHubHydrated } from "@/stores/hub.store";

interface HubGuardProps {
  children: React.ReactNode;
}

const EXEMPT_PATHS = ["/select-hub"];

export function HubGuard({ children }: HubGuardProps) {
  const router = useRouter();
  const pathname = usePathname();
  const isHydrated = useHubHydrated();
  const currentHub = useCurrentHub();

  useEffect(() => {
    // Wait for hydration
    if (!isHydrated) return;

    // Skip redirect for exempt paths
    if (EXEMPT_PATHS.includes(pathname)) return;

    // Redirect if no hub selected
    if (!currentHub) {
      router.replace("/select-hub");
    }
  }, [isHydrated, currentHub, pathname, router]);

  // Show nothing while checking (prevent flash)
  // Note: This may cause layout shift; adjust as needed
  if (!isHydrated) {
    return null;
  }

  // Don't block render for exempt paths
  if (EXEMPT_PATHS.includes(pathname)) {
    return <>{children}</>;
  }

  // Block render until hub is selected
  if (!currentHub) {
    return null;
  }

  return <>{children}</>;
}
```

### Step 3: Update Hub Component Barrel Export
Update `apps/web/src/components/hub/index.ts`:

```typescript
export { HubSwitcher } from "./hub-switcher";
export { HubSwitcherDrawer } from "./hub-switcher-drawer";
export { HubGuard } from "./hub-guard";
```

### Step 4: Integrate Hub Guard into Main Layout
Update `apps/web/src/app/(main)/layout.tsx`:

```tsx
import { SidebarProvider, SidebarInset } from '@/components/ui/sidebar';
import { GlassSidebar, AppHeader } from '@/components/layout';
import { AuthGuard } from '@/components/auth';
import { HubGuard } from '@/components/hub';

export default function MainLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <AuthGuard>
      <HubGuard>
        <SidebarProvider>
          <GlassSidebar />
          <SidebarInset>
            <AppHeader />
            <main className="flex-1 p-6">{children}</main>
          </SidebarInset>
        </SidebarProvider>
      </HubGuard>
    </AuthGuard>
  );
}
```

### Step 5: (Optional) Clear Hub on Logout
If needed, update auth logout flow to also clear hub:

```typescript
// In logout handler
import { useHubStore } from "@/stores/hub.store";

const clearHub = useHubStore.getState().clearHub;
clearHub();
```

## Todo List
- [x] Update glass-sidebar.tsx with HubSwitcher
- [x] Create hub-guard.tsx
- [x] Update hub/index.ts exports
- [x] Integrate HubGuard in main layout
- [x] Test redirect flow (no hub -> /select-hub)
- [x] Test exempt path (/select-hub doesn't redirect)
- [x] Test hub switching from drawer
- [x] Test collapsed sidebar state
- [x] Test full user flow: login -> select-hub -> main app -> switch hub
- [ ] **BUG**: Fix admin route exemption

## Success Criteria
- [x] HubSwitcher visible in sidebar below logo
- [x] No hub selected -> redirects to /select-hub
- [x] /select-hub page works without redirect loop
- [x] Select hub -> redirects to main app
- [x] Hub persists across page refresh
- [x] Switch hub works from drawer
- [x] Collapsed sidebar shows hub avatar
- [ ] **FAILING**: Admin routes exempt from hub requirement

## Review Notes
Integration successful with HubGuard matching AuthGuard pattern. HubSwitcher properly placed in glass-sidebar header. Redirect logic works for /select-hub exemption. Hydration handled properly.

**CRITICAL BUG FOUND**: Admin routes `/admin/*` not exempt from HubGuard. Current EXEMPT_PATHS only includes `/select-hub`. Admin users will be redirected to hub selection when accessing admin portal.

**Fix Required**:
```typescript
// hub-guard.tsx:12
const EXEMPT_PATHS = ["/select-hub"];

// Should be:
const isExemptPath = (path: string) =>
  EXEMPT_PATHS.includes(path) || path.startsWith("/admin");
```

## Risk Assessment
- **Medium risk** - Guard integration may cause hydration issues
- Mitigation: Careful handling of isHydrated state
- Test: Verify no flash of content during redirect

## Security Considerations
- Hub check happens client-side (acceptable for mock data approach)
- Real implementation would need server-side hub validation

## Next Steps
- Testing all flows
- Consider edge cases (logout, expired session, etc.)
- Future: Connect to real backend API
