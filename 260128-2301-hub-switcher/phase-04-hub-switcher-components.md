# Phase 4: Hub Switcher Components

## Context Links
- [Main Plan](./plan.md)
- [Phase 3: Hub Selection Page](./phase-03-hub-selection-page.md)
- [Existing Sheet component](../../apps/web/src/components/ui/sheet.tsx)
- [Existing Logo component](../../apps/web/src/components/layout/logo.tsx)
- [Existing glass-sidebar.tsx](../../apps/web/src/components/layout/glass-sidebar.tsx)

## Overview
- **Priority**: P1 (Core UI)
- **Status**: ✅ completed
- **Effort**: 2h
- **Dependencies**: Phase 3 completed
- **Completed**: 2026-01-28

## Key Insights
- HubSwitcher button placed next to Logo in sidebar header
- Drawer uses Sheet with `side="left"`
- Active hub shows checkmark indicator
- Reuse type badge styling from Phase 3

## Requirements

### Functional
- HubSwitcher: Show avatar + hub name button
- Click opens left-side drawer
- Drawer shows all user hubs
- Active hub marked with checkmark
- Click hub to switch and close drawer

### Non-functional
- Smooth slide animation from left
- Consistent with existing sidebar styling
- Accessible (keyboard, screen reader)

## Architecture

```
apps/web/src/components/hub/
├── hub-switcher.tsx           # Button trigger
└── hub-switcher-drawer.tsx    # Left-side Sheet with hub list
```

## UI Specifications

### HubSwitcher Button (Collapsed Sidebar)
```
+-------+
|  💼   |
+-------+
```

### HubSwitcher Button (Expanded Sidebar)
```
+---------------------------+
|  💼  Sale Team ABC   ▼    |
+---------------------------+
```

### Hub Switcher Drawer
```
+---------------------------+
|  Switch Hub          [X]  |
+---------------------------+
|  💼  Sale Team ABC     ✓  |
|      [sale] • 12 members  |
+---------------------------+
|  🎨  Design Studio A      |
|      [design] • 8 members |
+---------------------------+
|  🚀  Freelance Team B     |
|      [freelance] • 5 mem  |
+---------------------------+
```

### Type Badge Colors (reuse from Phase 3)
- sale: blue (bg-blue-100 text-blue-700)
- design: purple (bg-purple-100 text-purple-700)
- freelance: green (bg-green-100 text-green-700)

## Related Code Files

### Files to Create
1. `apps/web/src/components/hub/hub-switcher.tsx` - Trigger button
2. `apps/web/src/components/hub/hub-switcher-drawer.tsx` - Drawer content
3. `apps/web/src/components/hub/index.ts` - Barrel export

### Files to Modify
- None (integration in Phase 5)

## Implementation Steps

### Step 1: Create Hub Type Badge Styles Constant
Create shared constant for type badge styles (can be in hub-switcher-drawer or shared):

```typescript
import type { HubTypeValue } from "@/types";

export const HUB_TYPE_BADGE_STYLES: Record<HubTypeValue, string> = {
  sale: "bg-blue-100 text-blue-700 hover:bg-blue-100",
  design: "bg-purple-100 text-purple-700 hover:bg-purple-100",
  freelance: "bg-green-100 text-green-700 hover:bg-green-100",
};
```

### Step 2: Create Hub Switcher Drawer
Create `apps/web/src/components/hub/hub-switcher-drawer.tsx`:

```tsx
"use client";

import { Check } from "lucide-react";
import {
  Sheet,
  SheetContent,
  SheetHeader,
  SheetTitle,
} from "@/components/ui/sheet";
import { Badge } from "@/components/ui/badge";
import { useHubStore, useUserHubs, useCurrentHub } from "@/stores/hub.store";
import type { Hub, HubTypeValue } from "@/types";
import { cn } from "@/lib/utils";

const HUB_TYPE_BADGE_STYLES: Record<HubTypeValue, string> = {
  sale: "bg-blue-100 text-blue-700 hover:bg-blue-100",
  design: "bg-purple-100 text-purple-700 hover:bg-purple-100",
  freelance: "bg-green-100 text-green-700 hover:bg-green-100",
};

interface HubSwitcherDrawerProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
}

function HubListItem({
  hub,
  isActive,
  onSelect,
}: {
  hub: Hub;
  isActive: boolean;
  onSelect: (hub: Hub) => void;
}) {
  return (
    <button
      className={cn(
        "w-full flex items-start gap-3 p-3 rounded-lg text-left transition-colors",
        "hover:bg-accent focus-visible:ring-2 focus-visible:ring-primary focus-visible:outline-none",
        isActive && "bg-accent"
      )}
      onClick={() => onSelect(hub)}
    >
      {/* Avatar */}
      <span className="text-2xl flex-shrink-0">{hub.avatar || "🏢"}</span>

      {/* Content */}
      <div className="flex-1 min-w-0">
        <div className="font-medium truncate">{hub.name}</div>
        <div className="flex items-center gap-2 mt-1">
          <Badge variant="secondary" className={cn("text-xs", HUB_TYPE_BADGE_STYLES[hub.type])}>
            {hub.type}
          </Badge>
          <span className="text-xs text-muted-foreground">
            {hub.memberCount} members
          </span>
        </div>
      </div>

      {/* Active indicator */}
      {isActive && (
        <Check className="h-5 w-5 text-primary flex-shrink-0" />
      )}
    </button>
  );
}

export function HubSwitcherDrawer({ open, onOpenChange }: HubSwitcherDrawerProps) {
  const userHubs = useUserHubs();
  const currentHub = useCurrentHub();
  const setCurrentHub = useHubStore((state) => state.setCurrentHub);

  const handleSelectHub = (hub: Hub) => {
    setCurrentHub(hub);
    onOpenChange(false);
  };

  return (
    <Sheet open={open} onOpenChange={onOpenChange}>
      <SheetContent side="left" className="w-80 sm:w-96">
        <SheetHeader>
          <SheetTitle>Switch Hub</SheetTitle>
        </SheetHeader>

        <div className="mt-6 space-y-2">
          {userHubs.map((hub) => (
            <HubListItem
              key={hub.id}
              hub={hub}
              isActive={currentHub?.id === hub.id}
              onSelect={handleSelectHub}
            />
          ))}
        </div>
      </SheetContent>
    </Sheet>
  );
}
```

### Step 3: Create Hub Switcher Button
Create `apps/web/src/components/hub/hub-switcher.tsx`:

```tsx
"use client";

import { useState } from "react";
import { ChevronDown } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Skeleton } from "@/components/ui/skeleton";
import { useCurrentHub, useHubHydrated } from "@/stores/hub.store";
import { HubSwitcherDrawer } from "./hub-switcher-drawer";
import { cn } from "@/lib/utils";

interface HubSwitcherProps {
  collapsed?: boolean;
}

export function HubSwitcher({ collapsed = false }: HubSwitcherProps) {
  const [drawerOpen, setDrawerOpen] = useState(false);
  const isHydrated = useHubHydrated();
  const currentHub = useCurrentHub();

  // Loading state
  if (!isHydrated) {
    return (
      <div className={cn("flex items-center gap-2", collapsed && "justify-center")}>
        <Skeleton className="h-8 w-8 rounded" />
        {!collapsed && <Skeleton className="h-4 w-24" />}
      </div>
    );
  }

  // No hub selected state
  if (!currentHub) {
    return (
      <Button
        variant="ghost"
        size="sm"
        className={cn(
          "h-auto py-2",
          collapsed ? "w-10 p-2" : "w-full justify-start"
        )}
        onClick={() => setDrawerOpen(true)}
      >
        <span className="text-xl">🏢</span>
        {!collapsed && (
          <>
            <span className="ml-2 text-muted-foreground">Select hub</span>
            <ChevronDown className="ml-auto h-4 w-4 opacity-50" />
          </>
        )}
      </Button>
    );
  }

  return (
    <>
      <Button
        variant="ghost"
        size="sm"
        className={cn(
          "h-auto py-2",
          collapsed ? "w-10 p-2" : "w-full justify-start"
        )}
        onClick={() => setDrawerOpen(true)}
      >
        <span className="text-xl flex-shrink-0">{currentHub.avatar || "🏢"}</span>
        {!collapsed && (
          <>
            <span className="ml-2 truncate font-medium">{currentHub.name}</span>
            <ChevronDown className="ml-auto h-4 w-4 opacity-50 flex-shrink-0" />
          </>
        )}
      </Button>

      <HubSwitcherDrawer open={drawerOpen} onOpenChange={setDrawerOpen} />
    </>
  );
}
```

### Step 4: Create Barrel Export
Create `apps/web/src/components/hub/index.ts`:

```typescript
export { HubSwitcher } from "./hub-switcher";
export { HubSwitcherDrawer } from "./hub-switcher-drawer";
```

## Todo List
- [x] Create hub-switcher-drawer.tsx
- [x] Create hub-switcher.tsx
- [x] Create index.ts barrel export
- [x] Test drawer animation (slide from left)
- [x] Test collapsed vs expanded states
- [x] Test active hub checkmark
- [x] Test hub switching
- [x] Test keyboard accessibility

## Success Criteria
- [x] Button shows current hub avatar + name
- [x] Button shows chevron indicator
- [x] Click opens drawer from left
- [x] Drawer shows all hubs with type badges
- [x] Active hub has checkmark
- [x] Click hub switches and closes drawer
- [x] Collapsed state shows avatar only
- [x] Loading skeleton during hydration

## Review Notes
Excellent component architecture. Clean separation between trigger button and drawer. Proper hydration handling with loading skeletons. Collapsed/expanded states work well. Active hub indicator (checkmark) properly implemented. Keyboard focus management good.

**Note**: Badge styles duplicated with select-hub page - recommend extracting to shared constant as planned in Step 1.

## Risk Assessment
- **Low risk** - Standard component patterns
- Sheet component already supports `side="left"`

## Security Considerations
- No sensitive data in drawer
- State managed through Zustand store

## Next Steps
- Phase 5: Integration with main app
