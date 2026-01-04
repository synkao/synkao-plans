# Phase 01: Toast Setup (Sonner)

## Context

- **Parent Plan:** [plan.md](plan.md)
- **Issue:** #11 Modal, Drawer, Toast
- **Dependencies:** None

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P0 |
| Effort | 45 min |
| Implementation | ⬜ Pending |
| Review | ⬜ Pending |

## Key Insights

- `sonner` is shadcn-recommended toast library
- Simple API: `toast.success()`, `toast.error()`, etc.
- Requires adding `<Toaster />` to root layout

## Requirements

1. Install sonner package
2. Create wrapper component matching design system
3. Add Toaster to root layout
4. Verify toast works in dev

## Related Files

| File | Action |
|------|--------|
| `apps/web/src/components/ui/sonner.tsx` | Create |
| `apps/web/src/app/layout.tsx` | Modify |
| `apps/web/package.json` | Auto-updated |

## Implementation Steps

### Step 1: Install sonner

```bash
pnpm add sonner --filter web
```

### Step 2: Create sonner.tsx

**File:** `apps/web/src/components/ui/sonner.tsx`

```tsx
"use client"

import { Toaster as Sonner } from "sonner"

type ToasterProps = React.ComponentProps<typeof Sonner>

function Toaster({ ...props }: ToasterProps) {
  return (
    <Sonner
      theme="light"
      className="toaster group"
      position="bottom-right"
      toastOptions={{
        classNames: {
          toast:
            "group toast group-[.toaster]:bg-background group-[.toaster]:text-foreground group-[.toaster]:border-border group-[.toaster]:shadow-lg",
          description: "group-[.toast]:text-muted-foreground",
          actionButton:
            "group-[.toast]:bg-primary group-[.toast]:text-primary-foreground",
          cancelButton:
            "group-[.toast]:bg-muted group-[.toast]:text-muted-foreground",
        },
      }}
      {...props}
    />
  )
}

export { Toaster }
```

### Step 3: Add to root layout

**File:** `apps/web/src/app/layout.tsx`

Add import and component:
```tsx
import { Toaster } from "@/components/ui/sonner"

// In body, after providers:
<Toaster />
```

### Step 4: Test usage

```tsx
import { toast } from "sonner"

// In any component:
toast.success("Order saved!")
toast.error("Failed to save")
toast.info("Processing...")
toast.warning("Low stock alert")
```

## Todo

- [ ] Install sonner
- [ ] Create sonner.tsx wrapper
- [ ] Add Toaster to layout
- [ ] Test all 4 variants

## Success Criteria

- [ ] `pnpm build --filter web` passes
- [ ] Toast appears bottom-right
- [ ] 4 variants work: success, error, info, warning
- [ ] Styling matches design system

## Risk Assessment

| Risk | Level | Mitigation |
|------|-------|------------|
| Styling conflicts | Low | Use shadcn-recommended config |
| SSR issues | Low | "use client" directive |

## Security Considerations

- None - UI only component

## Next Steps

After completion → Phase 02: Status Badges
