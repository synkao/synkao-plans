# Phase 03: Layout Wrappers

## Context

- **Parent Plan:** [plan.md](plan.md)
- **Issue:** #6 Layout Components
- **Dependencies:** None

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P0 |
| Effort | 1h |
| Implementation | ⬜ Pending |
| Review | ⬜ Pending |

## Key Insights

- Layout ~80% done: GlassSidebar, PageHeader, AppHeader exist
- Missing: ContentArea wrapper for consistent page content padding
- YAGNI: Skip AppShell wrapper unless proven needed
- Current `(main)/layout.tsx` already handles full layout logic

## Requirements

1. Create ContentArea component for page content wrapping
2. Update layout exports
3. Optional: AppShell only if reuse pattern emerges

## Related Files

| File | Action |
|------|--------|
| `apps/web/src/components/layout/content-area.tsx` | Create |
| `apps/web/src/components/layout/index.ts` | Modify |
| `apps/web/src/app/(main)/layout.tsx` | Reference |

## Architecture

```
Current layout.tsx:
<SidebarProvider>
  <GlassSidebar />
  <SidebarInset>
    <AppHeader />
    <main className="flex-1 p-6">{children}</main>  ← Extract to ContentArea
  </SidebarInset>
</SidebarProvider>

After:
<SidebarProvider>
  <GlassSidebar />
  <SidebarInset>
    <AppHeader />
    <ContentArea>{children}</ContentArea>
  </SidebarInset>
</SidebarProvider>
```

## Implementation Steps

### Step 1: Create content-area.tsx

**File:** `apps/web/src/components/layout/content-area.tsx`

```tsx
import { cn } from "@/lib/utils";

export interface ContentAreaProps {
  children: React.ReactNode;
  className?: string;
  /** Remove default padding */
  noPadding?: boolean;
  /** Full height mode */
  fullHeight?: boolean;
}

/**
 * Wrapper for main page content with consistent padding
 */
export function ContentArea({
  children,
  className,
  noPadding = false,
  fullHeight = false,
}: ContentAreaProps) {
  return (
    <main
      className={cn(
        "flex-1",
        !noPadding && "p-6",
        fullHeight && "h-[calc(100vh-4rem)]", // subtract header height
        className
      )}
    >
      {children}
    </main>
  );
}
```

### Step 2: Update layout/index.ts

**File:** `apps/web/src/components/layout/index.ts`

Add export:
```tsx
export { ContentArea, type ContentAreaProps } from "./content-area";
```

### Step 3: (Optional) Update main layout

**File:** `apps/web/src/app/(main)/layout.tsx`

Replace inline main with ContentArea:
```tsx
import { ContentArea } from "@/components/layout";

// Replace:
<main className="flex-1 p-6">{children}</main>

// With:
<ContentArea>{children}</ContentArea>
```

## Todo

- [ ] Create content-area.tsx
- [ ] Update layout/index.ts exports
- [ ] Optionally update (main)/layout.tsx
- [ ] Verify build passes

## Success Criteria

- [ ] ContentArea exported from layout/
- [ ] Props: className, noPadding, fullHeight work
- [ ] Build passes
- [ ] No visual regression

## Risk Assessment

| Risk | Level | Mitigation |
|------|-------|------------|
| Visual regression | Low | Keep same default padding (p-6) |
| Unnecessary abstraction | Low | Simple wrapper, easy to inline if needed |

## Security Considerations

- None - layout wrapper only

## Next Steps

After completion → Phase 04: Confirm Dialog
