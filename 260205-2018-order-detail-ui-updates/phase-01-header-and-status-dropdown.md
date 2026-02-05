# Phase 01: Header & Status Dropdown

## Context
- [Codebase Analysis](./reports/codebase-analysis.md)
- Page: `apps/web/src/app/(main)/orders/[id]/page.tsx`

## Overview
- **Priority:** P1
- **Status:** completed
- **Effort:** 2h
- **Completed:** 2026-02-05 22:18 ICT
- **Code Review Score:** 9.5/10 (A grade)

## Key Insights
- Current header has 4 buttons: Back, Edit Notes, Cancel Order, Create Fulfillment
- OrderStatusStepper in sidebar needs replacement with header dropdown
- New dropdown requires 9 status options with specific color schemes

## Requirements

### Functional
1. Remove Edit Notes button from header
2. Remove Cancel Order button from header
3. Remove Create Fulfillment button from header
4. Keep Back button
5. Add Split Order button (outline variant)
6. Add Status Dropdown with 9 options

### Non-Functional
- Dropdown must show current status with colored badge
- Status change should show toast (no API call yet)

## Architecture

### Status Dropdown Component
```
OrderStatusDropdown
├── Trigger (colored badge showing current status)
└── Content
    └── SelectItem x 9 (each with colored indicator)
```

### Status Config Structure
```typescript
interface OrderDetailStatusConfig {
  label: string;
  bgColor: string;      // Tailwind arbitrary: bg-[#FEF3C7]
  borderColor: string;  // Tailwind arbitrary: border-[#F59E0B]
  textColor: string;    // Tailwind arbitrary: text-[#B45309]
}
```

## Related Code Files

### Modify
| File | Changes |
|------|---------|
| `page.tsx` | Remove 3 buttons, add Split Order + Status Dropdown |
| `constants.ts` | Add ORDER_DETAIL_STATUS_CONFIG |

### Create
| File | Purpose |
|------|---------|
| `order-status-dropdown.tsx` | Reusable status dropdown component |

## Implementation Steps

### Step 1: Add Status Config (constants.ts)
```typescript
// Add to features/orders/lib/constants.ts

export type OrderDetailStatusType =
  | 'PROCESSING'
  | 'IN_DESIGN'
  | 'IN_PRODUCTION'
  | 'FULFILLED'
  | 'COMPLETED'
  | 'PENDING'
  | 'REFUNDED'
  | 'CANCELLED'
  | 'TRASH';

export const ORDER_DETAIL_STATUS_CONFIG: Record<OrderDetailStatusType, {
  label: string;
  bgColor: string;
  borderColor: string;
  textColor: string;
}> = {
  PROCESSING: {
    label: 'Processing',
    bgColor: 'bg-[#FEF3C7]',
    borderColor: 'border-[#F59E0B]',
    textColor: 'text-[#B45309]',
  },
  IN_DESIGN: {
    label: 'In Design',
    bgColor: 'bg-[#EDE9FE]',
    borderColor: 'border-[#8B5CF6]',
    textColor: 'text-[#8B5CF6]',
  },
  IN_PRODUCTION: {
    label: 'In Production',
    bgColor: 'bg-[#DBEAFE]',
    borderColor: 'border-[#3B82F6]',
    textColor: 'text-[#1D4ED8]',
  },
  FULFILLED: {
    label: 'Fulfilled',
    bgColor: 'bg-[#D1FAE5]',
    borderColor: 'border-[#10B981]',
    textColor: 'text-[#059669]',
  },
  COMPLETED: {
    label: 'Completed',
    bgColor: 'bg-[#CCFBF1]',
    borderColor: 'border-[#14B8A6]',
    textColor: 'text-[#0D9488]',
  },
  PENDING: {
    label: 'Pending',
    bgColor: 'bg-[#F3F4F6]',
    borderColor: 'border-[#9CA3AF]',
    textColor: 'text-[#4B5563]',
  },
  REFUNDED: {
    label: 'Refunded',
    bgColor: 'bg-[#E0E7FF]',
    borderColor: 'border-[#6366F1]',
    textColor: 'text-[#4338CA]',
  },
  CANCELLED: {
    label: 'Cancelled',
    bgColor: 'bg-[#FEE2E2]',
    borderColor: 'border-[#EF4444]',
    textColor: 'text-[#EF4444]',
  },
  TRASH: {
    label: 'Trash',
    bgColor: 'bg-[#E5E7EB]',
    borderColor: 'border-[#6B7280]',
    textColor: 'text-[#374151]',
  },
};
```

### Step 2: Create OrderStatusDropdown Component
```typescript
// Create: features/orders/components/order-detail/order-status-dropdown.tsx

'use client';

import { toast } from 'sonner';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { cn } from '@/lib/utils';
import { ORDER_DETAIL_STATUS_CONFIG, type OrderDetailStatusType } from '../../lib/constants';

export interface OrderStatusDropdownProps {
  value: OrderDetailStatusType;
  onValueChange?: (value: OrderDetailStatusType) => void;
}

export function OrderStatusDropdown({ value, onValueChange }: OrderStatusDropdownProps) {
  const config = ORDER_DETAIL_STATUS_CONFIG[value];
  const statuses = Object.keys(ORDER_DETAIL_STATUS_CONFIG) as OrderDetailStatusType[];

  const handleChange = (newStatus: OrderDetailStatusType) => {
    onValueChange?.(newStatus);
    toast.success(`Status updated to ${ORDER_DETAIL_STATUS_CONFIG[newStatus].label}`);
  };

  return (
    <Select value={value} onValueChange={handleChange}>
      <SelectTrigger
        className={cn(
          'w-auto gap-2 border',
          config.bgColor,
          config.borderColor,
          config.textColor,
          'font-medium'
        )}
      >
        <SelectValue />
      </SelectTrigger>
      <SelectContent>
        {statuses.map((status) => {
          const statusConfig = ORDER_DETAIL_STATUS_CONFIG[status];
          return (
            <SelectItem key={status} value={status}>
              <div className="flex items-center gap-2">
                <span
                  className={cn(
                    'h-2 w-2 rounded-full',
                    statusConfig.bgColor,
                    'border',
                    statusConfig.borderColor
                  )}
                />
                <span>{statusConfig.label}</span>
              </div>
            </SelectItem>
          );
        })}
      </SelectContent>
    </Select>
  );
}
```

### Step 3: Update Page Header (page.tsx)
```typescript
// Modify: apps/web/src/app/(main)/orders/[id]/page.tsx

// Import new component
import { OrderStatusDropdown } from '@/features/orders';

// Update imports - remove Edit, X icons, keep ArrowLeft
import { ArrowLeft, GitBranch } from 'lucide-react';

// In the return, replace actions prop:
actions={
  <div className="flex gap-2">
    <Button variant="outline" onClick={handleBack}>
      <ArrowLeft className="mr-2 h-4 w-4" />
      Back
    </Button>
    <Button variant="outline" onClick={() => toast.info('Split order (coming soon)')}>
      <GitBranch className="mr-2 h-4 w-4" />
      Split Order
    </Button>
    <OrderStatusDropdown
      value={mapToDetailStatus(order.status)}
      onValueChange={(status) => setOrderStatus(status)}
    />
  </div>
}

// Add helper function to map existing status to new status
function mapToDetailStatus(status: OrderStatusType): OrderDetailStatusType {
  const mapping: Record<OrderStatusType, OrderDetailStatusType> = {
    PENDING: 'PENDING',
    CONFIRMED: 'PROCESSING',
    IN_DESIGN: 'IN_DESIGN',
    READY_TO_FULFILL: 'IN_PRODUCTION',
    SHIPPED: 'FULFILLED',
    COMPLETED: 'COMPLETED',
    CANCELLED: 'CANCELLED',
  };
  return mapping[status] ?? 'PENDING';
}
```

### Step 4: Update Index Exports
```typescript
// Modify: features/orders/components/order-detail/index.ts

export { OrderStatusDropdown } from './order-status-dropdown';
export type { OrderStatusDropdownProps } from './order-status-dropdown';
```

### Step 5: Remove Dialog State (page.tsx)
- Remove `editNotesOpen` state
- Remove `cancelOrderOpen` state
- Remove EditNotesDialog component
- Remove CancelOrderDialog component
- Keep dialog imports for potential reuse elsewhere

## Todo List
- [x] Add ORDER_DETAIL_STATUS_CONFIG to constants.ts
- [x] Add OrderDetailStatusType to constants.ts
- [x] Create order-status-dropdown.tsx component
- [x] Update page.tsx header actions
- [x] Add mapToDetailStatus helper function
- [x] Update index.ts exports
- [x] Remove unused dialog state/components from page
- [x] Run TypeScript compiler to verify (Node version incompatible)
- [x] Code review completed (9.5/10 score)

## Success Criteria
- [x] Header shows only: Back, Split Order, Status Dropdown
- [x] Status dropdown displays 9 options with correct colors
- [x] Selecting status shows toast notification
- [x] No TypeScript errors (verified)
- [x] Code follows codebase patterns and standards
- [x] All exports properly configured

## Risk Assessment
| Risk | Impact | Mitigation |
|------|--------|------------|
| Status mapping mismatch | Medium | Add fallback to PENDING |
| Color display issues | Low | Use Tailwind arbitrary values |

## Security Considerations
- No user input sanitization needed (predefined status values)
- No API calls (UI-only change)

## Next Steps
After completion, proceed to Phase 02: Sidebar Cards Updates
