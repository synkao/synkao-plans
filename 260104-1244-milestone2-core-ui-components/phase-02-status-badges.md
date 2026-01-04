# Phase 02: Status Badges

## Context

- **Parent Plan:** [plan.md](plan.md)
- **Issue:** #8 Status Badges
- **Dependencies:** None

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P0 |
| Effort | 2h |
| Implementation | ⬜ Pending |
| Review | ⬜ Pending |

## Key Insights

- Existing `features/design/lib/constants.ts` has color configs
- Existing `features/design/components/priority-badge.tsx` is feature-specific
- Need shared badges in `components/ui/` for reuse across features
- Use cva pattern consistent with existing `badge.tsx`

## Requirements

4 badge types với color variants:

| Type | Values |
|------|--------|
| Design Status | TODO, IN_PROGRESS, REVIEW, REVISION, APPROVED |
| Priority | LOW, NORMAL, HIGH, URGENT |
| Fulfillment | PENDING, PROCESSING, SHIPPED, DELIVERED, FAILED |
| Source | NEW_ORDER, REDESIGN, COMPLAINT |

## Related Files

| File | Action |
|------|--------|
| `apps/web/src/components/ui/status-badge.tsx` | Create |
| `apps/web/src/types/status.ts` | Create |
| `apps/web/src/components/ui/badge.tsx` | Reference (existing) |
| `apps/web/src/features/design/lib/constants.ts` | Reference (existing) |

## Architecture

```
status-badge.tsx
├── DesignStatusBadge (status prop)
├── PriorityBadge (priority prop)
├── FulfillmentBadge (status prop)
└── SourceBadge (source prop)

Each uses cva() for variant styling
```

## Implementation Steps

### Step 1: Create types/status.ts

**File:** `apps/web/src/types/status.ts`

```tsx
// Design task status
export type DesignStatus =
  | "TODO"
  | "IN_PROGRESS"
  | "REVIEW"
  | "REVISION"
  | "APPROVED";

// Task/order priority
export type Priority = "LOW" | "NORMAL" | "HIGH" | "URGENT";

// Fulfillment/shipping status
export type FulfillmentStatus =
  | "PENDING"
  | "PROCESSING"
  | "SHIPPED"
  | "DELIVERED"
  | "FAILED";

// Order source type
export type OrderSource = "NEW_ORDER" | "REDESIGN" | "COMPLAINT";
```

### Step 2: Create status-badge.tsx

**File:** `apps/web/src/components/ui/status-badge.tsx`

```tsx
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";
import type {
  DesignStatus,
  Priority,
  FulfillmentStatus,
  OrderSource
} from "@/types/status";

// Base style for all status badges
const baseBadgeClass = "inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium";

// Design Status Badge
const designStatusVariants = cva(baseBadgeClass, {
  variants: {
    status: {
      TODO: "bg-slate-100 text-slate-700",
      IN_PROGRESS: "bg-blue-100 text-blue-700",
      REVIEW: "bg-purple-100 text-purple-700",
      REVISION: "bg-amber-100 text-amber-700",
      APPROVED: "bg-green-100 text-green-700",
    },
  },
});

export interface DesignStatusBadgeProps {
  status: DesignStatus;
  className?: string;
}

export function DesignStatusBadge({ status, className }: DesignStatusBadgeProps) {
  const label = status.replace(/_/g, " ");
  return (
    <span className={cn(designStatusVariants({ status }), className)}>
      {label}
    </span>
  );
}

// Priority Badge
const priorityVariants = cva(baseBadgeClass, {
  variants: {
    priority: {
      LOW: "bg-gray-100 text-gray-600",
      NORMAL: "bg-blue-100 text-blue-700",
      HIGH: "bg-amber-100 text-amber-700",
      URGENT: "bg-red-100 text-red-700",
    },
  },
});

export interface PriorityBadgeProps {
  priority: Priority;
  className?: string;
}

export function PriorityBadge({ priority, className }: PriorityBadgeProps) {
  return (
    <span className={cn(priorityVariants({ priority }), className)}>
      {priority}
    </span>
  );
}

// Fulfillment Badge
const fulfillmentVariants = cva(baseBadgeClass, {
  variants: {
    status: {
      PENDING: "bg-slate-100 text-slate-600",
      PROCESSING: "bg-blue-100 text-blue-700",
      SHIPPED: "bg-indigo-100 text-indigo-700",
      DELIVERED: "bg-green-100 text-green-700",
      FAILED: "bg-red-100 text-red-700",
    },
  },
});

export interface FulfillmentBadgeProps {
  status: FulfillmentStatus;
  className?: string;
}

export function FulfillmentBadge({ status, className }: FulfillmentBadgeProps) {
  return (
    <span className={cn(fulfillmentVariants({ status }), className)}>
      {status}
    </span>
  );
}

// Source Badge
const sourceVariants = cva(baseBadgeClass + " border", {
  variants: {
    source: {
      NEW_ORDER: "bg-emerald-50 text-emerald-700 border-emerald-200",
      REDESIGN: "bg-sky-50 text-sky-700 border-sky-200",
      COMPLAINT: "bg-rose-50 text-rose-700 border-rose-200",
    },
  },
});

export interface SourceBadgeProps {
  source: OrderSource;
  className?: string;
}

export function SourceBadge({ source, className }: SourceBadgeProps) {
  const label = source.replace(/_/g, " ");
  return (
    <span className={cn(sourceVariants({ source }), className)}>
      {label}
    </span>
  );
}
```

### Step 3: Update types/index.ts

Add export for status types:
```tsx
export * from "./status";
```

## Todo

- [ ] Create types/status.ts
- [ ] Create status-badge.tsx with 4 badge components
- [ ] Update types/index.ts exports
- [ ] Verify build passes

## Success Criteria

- [ ] All 4 badge types exported
- [ ] Colors match design system
- [ ] TypeScript types correct
- [ ] No conflicts with existing feature badges
- [ ] Build passes

## Risk Assessment

| Risk | Level | Mitigation |
|------|-------|------------|
| Naming conflict with feature badge | Low | Different file path, independent |
| Color mismatch | Low | Use established color palette |

## Security Considerations

- None - UI only components

## Next Steps

After completion → Phase 03: Layout Wrappers
