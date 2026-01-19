# Phase 2: Shared Components

## Context Links

- Spec: `docs/frontend/synkao-frontend-spec.md` sections 3.1, 3.2
- Pattern: `apps/web/src/features/design/components/status-badge.tsx`
- Constants: Phase 1 `features/orders/lib/constants.ts`
- UI components: `apps/web/src/components/ui/badge.tsx`

## Overview

| Field | Value |
|-------|-------|
| Priority | P1 |
| Status | pending |
| Effort | 1h |
| Date | 2026-01-19 |

Create reusable badge and cell components for order display across list and detail views.

## Key Insights

- Follow `StatusBadge` pattern from design feature
- Use constants from Phase 1 for consistent styling
- Fulfiller cell needs to show name + status icon per spec

## Requirements

### Functional
- `OrderStatusBadge`: Display order status (PENDING, CONFIRMED, etc.)
- `DesignPhaseBadge`: Display design phase (PENDING, IN_PROGRESS, COMPLETED)
- `ItemDesignStatusBadge`: Display item-level design status
- `FulfillerCell`: Show fulfiller name with status indicator

### Non-functional
- Consistent with existing badge patterns
- Tailwind JIT compatible (no dynamic classes)

## Architecture

```
components/shared/
├── order-status-badge.tsx
├── design-phase-badge.tsx
├── item-design-status-badge.tsx
├── fulfiller-cell.tsx
└── index.ts
```

## Related Code Files

**To Create:**
- `features/orders/components/shared/order-status-badge.tsx`
- `features/orders/components/shared/design-phase-badge.tsx`
- `features/orders/components/shared/item-design-status-badge.tsx`
- `features/orders/components/shared/fulfiller-cell.tsx`
- `features/orders/components/shared/index.ts`

**Reference:**
- `features/design/components/status-badge.tsx`
- `components/ui/badge.tsx`

## Implementation Steps

### 1. Create OrderStatusBadge

**File:** `features/orders/components/shared/order-status-badge.tsx`

```typescript
'use client';

import type { OrderStatusType } from '../../types';
import { Badge } from '@/components/ui/badge';
import { cn } from '@/lib/utils';
import { ORDER_STATUS_CONFIG } from '../../lib/constants';

export interface OrderStatusBadgeProps {
  status: OrderStatusType;
  size?: 'sm' | 'md';
}

/**
 * Badge displaying order status with appropriate color
 */
export function OrderStatusBadge({ status, size = 'md' }: OrderStatusBadgeProps) {
  const config = ORDER_STATUS_CONFIG[status];

  return (
    <Badge
      variant="outline"
      className={cn(
        'border-0 font-medium',
        config.bgClass,
        config.textClass,
        size === 'sm' && 'text-xs px-1.5 py-0'
      )}
    >
      {config.label}
    </Badge>
  );
}
```

### 2. Create DesignPhaseBadge

**File:** `features/orders/components/shared/design-phase-badge.tsx`

```typescript
'use client';

import type { DesignPhaseStatusType } from '../../types';
import { Badge } from '@/components/ui/badge';
import { cn } from '@/lib/utils';
import { DESIGN_PHASE_CONFIG } from '../../lib/constants';

export interface DesignPhaseBadgeProps {
  status: DesignPhaseStatusType;
  size?: 'sm' | 'md';
}

/**
 * Badge displaying order design phase status
 */
export function DesignPhaseBadge({ status, size = 'md' }: DesignPhaseBadgeProps) {
  const config = DESIGN_PHASE_CONFIG[status];

  return (
    <Badge
      variant="outline"
      className={cn(
        'border-0 font-medium',
        config.bgClass,
        config.textClass,
        size === 'sm' && 'text-xs px-1.5 py-0'
      )}
    >
      {config.label}
    </Badge>
  );
}
```

### 3. Create ItemDesignStatusBadge

**File:** `features/orders/components/shared/item-design-status-badge.tsx`

```typescript
'use client';

import type { ItemDesignStatusType } from '../../types';
import { Badge } from '@/components/ui/badge';
import { cn } from '@/lib/utils';
import { ITEM_DESIGN_STATUS_CONFIG } from '../../lib/constants';

export interface ItemDesignStatusBadgeProps {
  status?: ItemDesignStatusType;
  size?: 'sm' | 'md';
}

/**
 * Badge displaying order item design status
 * Used in order list item rows
 */
export function ItemDesignStatusBadge({ status, size = 'sm' }: ItemDesignStatusBadgeProps) {
  const config = ITEM_DESIGN_STATUS_CONFIG[status || 'NOT_REQUIRED'];

  // Return dash for NOT_REQUIRED
  if (!status || status === 'NOT_REQUIRED') {
    return <span className="text-gray-400">—</span>;
  }

  return (
    <Badge
      variant="outline"
      className={cn(
        'border-0 font-medium',
        config.bgClass,
        config.textClass,
        size === 'sm' && 'text-xs px-1.5 py-0'
      )}
    >
      {config.label}
    </Badge>
  );
}
```

### 4. Create FulfillerCell

**File:** `features/orders/components/shared/fulfiller-cell.tsx`

Per spec:
- Chưa assign: `—` (dash)
- Đã assign: Tên Fulfiller
- Đang ship: Tên + 🚚 icon
- Đã giao: Tên + ✓

```typescript
'use client';

import { Truck, Check } from 'lucide-react';
import { cn } from '@/lib/utils';

export interface FulfillerCellProps {
  fulfillerName?: string;
  isShipped?: boolean;
  isDelivered?: boolean;
  className?: string;
}

/**
 * Cell displaying fulfiller name with status indicator
 * Shows: dash if unassigned, name if assigned, name+truck if shipping, name+check if delivered
 */
export function FulfillerCell({
  fulfillerName,
  isShipped,
  isDelivered,
  className,
}: FulfillerCellProps) {
  if (!fulfillerName) {
    return <span className={cn('text-gray-400', className)}>—</span>;
  }

  return (
    <span className={cn('inline-flex items-center gap-1 text-sm', className)}>
      <span className="text-gray-700">{fulfillerName}</span>
      {isDelivered && (
        <Check className="h-3.5 w-3.5 text-emerald-500" />
      )}
      {isShipped && !isDelivered && (
        <Truck className="h-3.5 w-3.5 text-cyan-500" />
      )}
    </span>
  );
}
```

### 5. Create index file

**File:** `features/orders/components/shared/index.ts`

```typescript
export { OrderStatusBadge } from './order-status-badge';
export type { OrderStatusBadgeProps } from './order-status-badge';

export { DesignPhaseBadge } from './design-phase-badge';
export type { DesignPhaseBadgeProps } from './design-phase-badge';

export { ItemDesignStatusBadge } from './item-design-status-badge';
export type { ItemDesignStatusBadgeProps } from './item-design-status-badge';

export { FulfillerCell } from './fulfiller-cell';
export type { FulfillerCellProps } from './fulfiller-cell';
```

## Todo List

- [ ] Create `features/orders/components/shared/order-status-badge.tsx`
- [ ] Create `features/orders/components/shared/design-phase-badge.tsx`
- [ ] Create `features/orders/components/shared/item-design-status-badge.tsx`
- [ ] Create `features/orders/components/shared/fulfiller-cell.tsx`
- [ ] Create `features/orders/components/shared/index.ts`
- [ ] Visual test in design system or storybook

## Success Criteria

- All badges render with correct colors per status
- FulfillerCell shows appropriate icon based on state
- Components are reusable across list and detail views
- No TypeScript errors

## Risk Assessment

| Risk | Mitigation |
|------|------------|
| Tailwind purging dynamic classes | Use static class maps in constants |
| Badge variant compatibility | Use existing Badge component from shadcn/ui |

## Security Considerations

- Display-only components, no user input handling
- No XSS risk (using React)

## Next Steps

Proceed to Phase 3: Order List Components (OrderRow, OrderItemRow, OrderListTable, Filters)
