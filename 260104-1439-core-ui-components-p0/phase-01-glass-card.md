# Phase 1: Glass Card Component

## Context
- **Issue:** #7
- **Parent Plan:** [plan.md](./plan.md)
- **Reference:** `apps/web/src/components/layout/glass-sidebar.tsx`

## Overview
| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P0 |
| Effort | 3h |
| Status | in_progress (pending dev-tools demo) |
| Dependencies | None (Tailwind only) |
| Review | plans/reports/code-reviewer-260104-1509-glass-card-phase1.md |

## Key Insights

Từ codebase analysis:
- `glass-sidebar.tsx` dùng: `bg-white/60 backdrop-blur-[12px]`
- `card.tsx` dùng compound pattern: Card, CardHeader, CardContent, CardFooter
- Tất cả components dùng `data-slot` attribute
- Pattern: `cn()` cho class merging

## Requirements

1. 4 variants: default, hover, selected, clickable
2. Compound components: GlassCard, GlassCardHeader, GlassCardContent, GlassCardFooter
3. Consistent với existing card.tsx pattern
4. Support cho selected state via prop

## Architecture

```tsx
interface GlassCardProps extends React.ComponentProps<"div"> {
  variant?: "default" | "hover" | "selected" | "clickable";
  selected?: boolean;
}
```

### CSS Specs

| Variant | Classes |
|---------|---------|
| base | `bg-white/70 backdrop-blur-[10px] border border-white/20 rounded-xl shadow-sm` |
| hover | `+ hover:bg-white/80 transition-colors` |
| selected | `+ border-primary ring-2 ring-primary/20` |
| clickable | `+ cursor-pointer hover:shadow-lg hover:scale-[1.01] transition-all` |

## Related Code Files

- `apps/web/src/components/ui/card.tsx` - Base card pattern
- `apps/web/src/components/layout/glass-sidebar.tsx` - Glass effect reference
- `apps/web/src/lib/utils.ts` - `cn()` utility

## Implementation Steps

### Step 1: Create glass-card.tsx (2h)

```tsx
// apps/web/src/components/ui/glass-card.tsx
import * as React from "react";
import { cn } from "@/lib/utils";
import { cva, type VariantProps } from "class-variance-authority";

const glassCardVariants = cva(
  "flex flex-col gap-4 rounded-xl border bg-white/70 backdrop-blur-[10px] border-white/20 shadow-sm",
  {
    variants: {
      variant: {
        default: "",
        hover: "hover:bg-white/80 transition-colors",
        selected: "border-primary ring-2 ring-primary/20",
        clickable: "cursor-pointer hover:bg-white/80 hover:shadow-lg hover:scale-[1.01] transition-all",
      },
    },
    defaultVariants: {
      variant: "default",
    },
  }
);

interface GlassCardProps
  extends React.ComponentProps<"div">,
    VariantProps<typeof glassCardVariants> {
  selected?: boolean;
}

function GlassCard({ className, variant, selected, ...props }: GlassCardProps) {
  return (
    <div
      data-slot="glass-card"
      data-selected={selected}
      className={cn(
        glassCardVariants({ variant: selected ? "selected" : variant }),
        className
      )}
      {...props}
    />
  );
}
```

### Step 2: Add sub-components (30m)

```tsx
function GlassCardHeader({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="glass-card-header"
      className={cn("flex flex-col gap-1.5 px-4 pt-4", className)}
      {...props}
    />
  );
}

function GlassCardContent({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="glass-card-content"
      className={cn("px-4", className)}
      {...props}
    />
  );
}

function GlassCardFooter({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="glass-card-footer"
      className={cn("flex items-center px-4 pb-4", className)}
      {...props}
    />
  );
}
```

### Step 3: Export & Test (30m)

1. Export all components
2. Add to design-system showcase page
3. Test all 4 variants
4. Verify selected state toggle

## Todo List

- [x] Create glass-card.tsx với variants
- [x] Add GlassCardHeader component
- [x] Add GlassCardContent component
- [x] Add GlassCardFooter component
- [x] Add to design-system page
- [x] Test hover/click interactions
- [x] Verify selected state works
- [ ] Add demo to /dev-tools page (interactive toggle)

## Code Review Findings (2026-01-04)

**Status:** ✅ APPROVED - Production ready with recommended improvements

**Issues Found:**
- 0 Critical
- 0 High Priority
- 3 Medium Priority (accessibility enhancements)
- 4 Low Priority (developer experience)

**Recommended Improvements (Optional):**
- [ ] Add keyboard accessibility (role, tabIndex, focus-visible) - M2, M3
- [ ] Consider semantic HTML for title/description (h3, p elements) - M1
- [ ] Add reduced-motion support - L1
- [ ] Add JSDoc documentation - L2

## Success Criteria

- [x] Renders với glass effect visible
- [x] 4 variants work as expected
- [x] selected prop toggles selected variant
- [x] Clickable variant has proper cursor
- [⚠️] Keyboard focusable when clickable (works but missing role/tabIndex)
- [x] Matches glass-sidebar style

**Achievement:** 5/6 core criteria met (83%) + 1 accessibility enhancement recommended

## Risk Assessment

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Backdrop-blur browser support | Low | Low | Modern browsers all support |
| Class conflicts với Card | Low | Medium | Use distinct data-slot |

## Security Considerations

None - pure presentational component.

## Next Steps

Sau khi complete Phase 1:
1. Proceed to Phase 2: Form Components
2. Refactor existing TaskCard to use GlassCard if appropriate
