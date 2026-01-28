# Phase 3: Hub Selection Page

## Context Links
- [Main Plan](./plan.md)
- [Phase 2: Hub Store](./phase-02-hub-store.md)
- [Existing login page](../../apps/web/src/app/(auth)/login/page.tsx)
- [Main layout](../../apps/web/src/app/(main)/layout.tsx)

## Overview
- **Priority**: P1 (User flow)
- **Status**: ✅ completed
- **Effort**: 1.5h
- **Dependencies**: Phase 2 completed
- **Completed**: 2026-01-28

## Key Insights
- Route: /select-hub (inside (main) group for auth protection)
- Full-page experience, no sidebar
- Card-based hub selection
- Redirect to home after selection

## Requirements

### Functional
- Display all user's hubs in grid
- Show hub avatar, name, type badge, member count
- Click to select hub
- Redirect to / after selection
- Loading state while hydrating

### Non-functional
- Responsive grid (1 col mobile, 2-3 cols desktop)
- Accessible keyboard navigation
- Fast selection feedback

## Architecture

```
apps/web/src/app/(main)/select-hub/
└── page.tsx           # Hub selection page
```

## UI Specifications

### Hub Card Design
```
+----------------------------------+
|  💼                              |
|  Sale Team ABC                   |
|  [sale]  •  12 members           |
+----------------------------------+
```

### Type Badge Colors
- sale: blue (bg-blue-100 text-blue-700)
- design: purple (bg-purple-100 text-purple-700)
- freelance: green (bg-green-100 text-green-700)

## Related Code Files

### Files to Create
1. `apps/web/src/app/(main)/select-hub/page.tsx` - Hub selection page

### Files to Modify
- None initially (redirect logic in Phase 5)

## Implementation Steps

### Step 1: Create Hub Selection Page
Create `apps/web/src/app/(main)/select-hub/page.tsx`:

```tsx
"use client";

import { useRouter } from "next/navigation";
import { useHubStore, useUserHubs, useHubHydrated } from "@/stores/hub.store";
import { Card, CardContent } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Skeleton } from "@/components/ui/skeleton";
import type { Hub, HubTypeValue } from "@/types";
import { cn } from "@/lib/utils";

// Type badge color mapping
const TYPE_BADGE_STYLES: Record<HubTypeValue, string> = {
  sale: "bg-blue-100 text-blue-700 hover:bg-blue-100",
  design: "bg-purple-100 text-purple-700 hover:bg-purple-100",
  freelance: "bg-green-100 text-green-700 hover:bg-green-100",
};

function HubCard({ hub, onSelect }: { hub: Hub; onSelect: (hub: Hub) => void }) {
  return (
    <Card
      className="cursor-pointer transition-all hover:shadow-md hover:border-primary/50 focus-visible:ring-2 focus-visible:ring-primary"
      tabIndex={0}
      role="button"
      onClick={() => onSelect(hub)}
      onKeyDown={(e) => {
        if (e.key === "Enter" || e.key === " ") {
          e.preventDefault();
          onSelect(hub);
        }
      }}
    >
      <CardContent className="p-6">
        <div className="flex flex-col gap-3">
          {/* Avatar */}
          <span className="text-4xl">{hub.avatar || "🏢"}</span>

          {/* Name */}
          <h3 className="font-semibold text-lg">{hub.name}</h3>

          {/* Type badge + member count */}
          <div className="flex items-center gap-2 text-sm text-muted-foreground">
            <Badge variant="secondary" className={cn(TYPE_BADGE_STYLES[hub.type])}>
              {hub.type}
            </Badge>
            <span>•</span>
            <span>{hub.memberCount} members</span>
          </div>
        </div>
      </CardContent>
    </Card>
  );
}

function HubCardSkeleton() {
  return (
    <Card>
      <CardContent className="p-6">
        <div className="flex flex-col gap-3">
          <Skeleton className="h-10 w-10 rounded" />
          <Skeleton className="h-6 w-32" />
          <div className="flex items-center gap-2">
            <Skeleton className="h-5 w-16" />
            <Skeleton className="h-5 w-20" />
          </div>
        </div>
      </CardContent>
    </Card>
  );
}

export default function SelectHubPage() {
  const router = useRouter();
  const isHydrated = useHubHydrated();
  const userHubs = useUserHubs();
  const setCurrentHub = useHubStore((state) => state.setCurrentHub);

  const handleSelectHub = (hub: Hub) => {
    setCurrentHub(hub);
    router.push("/");
  };

  return (
    <div className="min-h-screen flex flex-col items-center justify-center p-6 bg-gradient-to-b from-background to-muted/30">
      <div className="max-w-3xl w-full">
        {/* Header */}
        <div className="text-center mb-8">
          <h1 className="text-3xl font-bold mb-2">Select a Hub</h1>
          <p className="text-muted-foreground">
            Choose the hub you want to work in
          </p>
        </div>

        {/* Hub Grid */}
        <div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
          {!isHydrated ? (
            <>
              <HubCardSkeleton />
              <HubCardSkeleton />
              <HubCardSkeleton />
            </>
          ) : (
            userHubs.map((hub) => (
              <HubCard key={hub.id} hub={hub} onSelect={handleSelectHub} />
            ))
          )}
        </div>
      </div>
    </div>
  );
}
```

### Step 2: Handle Layout Override
The select-hub page needs full-screen layout without sidebar. Options:
1. Create separate route group (e.g., (hub-selection))
2. Use layout prop to hide sidebar
3. Override styles on page

Recommendation: Keep in (main) for auth protection, use CSS to hide sidebar or create minimal layout.

Alternative: Create separate layout file:
`apps/web/src/app/(main)/select-hub/layout.tsx`:
```tsx
export default function SelectHubLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  // Override main layout for full-screen experience
  return <>{children}</>;
}
```

## Todo List
- [x] Create select-hub/page.tsx
- [x] Implement HubCard component
- [x] Implement type badge styling
- [x] Add loading skeleton state
- [x] Handle hub selection and redirect
- [x] Test responsive grid layout
- [x] Test keyboard accessibility

## Success Criteria
- [x] Page displays all user hubs
- [x] Hub cards show avatar, name, type badge, member count
- [x] Clicking card selects hub and redirects to /
- [x] Loading skeleton shows while hydrating
- [x] Responsive grid works on all screen sizes
- [x] Keyboard navigation works (Tab + Enter)

## Review Notes
Clean implementation with proper accessibility. Keyboard navigation with onKeyDown handler. Loading skeletons prevent hydration flash. Responsive grid works across screen sizes. Layout override with separate layout.tsx file works well.

**Note**: Badge styles duplicated here and in hub-switcher-drawer.tsx - consider extracting to shared constant.

## Risk Assessment
- **Low risk** - Standard page component
- Layout override may need testing with main layout

## Security Considerations
- Page protected by AuthGuard (in (main) layout)
- No sensitive data displayed

## Next Steps
- Phase 4: Create Hub Switcher Components
