# Codebase Analysis Report

## Current Order Detail Page Structure

### Page Location
- `apps/web/src/app/(main)/orders/[id]/page.tsx`

### Components (in `features/orders/components/order-detail/`)
| Component | Purpose | Lines | Notes |
|-----------|---------|-------|-------|
| `order-status-stepper.tsx` | Horizontal step progress | 115 | TO BE REMOVED (replaced by dropdown) |
| `order-metadata-card.tsx` | Timestamps + Update Status btn | 64 | Remove Update Status button |
| `customer-info-card.tsx` | Customer info + External ID + Source | 168 | Remove External ID & Source rows |
| `order-summary-card.tsx` | Financial breakdown + payment | 83 | Simplify payment row |
| `order-item-card.tsx` | Item with design actions | 239 | Modify task button logic |
| `order-timeline.tsx` | Event timeline | - | No changes |
| `edit-notes-dialog.tsx` | Edit notes modal | - | TO BE REMOVED from header |
| `cancel-order-dialog.tsx` | Cancel order modal | - | TO BE REMOVED from header |

### Page Header Actions (Current)
- Back button
- Edit Notes button
- Cancel Order button (conditional)
- Create Fulfillment button (labeled "Add Tracking")

### Sidebar Cards (Current Order)
1. OrderMetadataCard - timestamps + Update Status button
2. OrderStatusStepper - horizontal stepper
3. OrderSummaryCard - financial + payment with badge
4. OrderTimeline - event log

---

## Key Existing Patterns

### UI Components Available
- `@/components/ui/select.tsx` - Radix Select (good for status dropdown)
- `@/components/ui/dialog.tsx` - Radix Dialog (for upload modal)
- `@/components/ui/file-uploader.tsx` - Drag & drop with preview
- `@/components/ui/badge.tsx` - Status badges

### Constants Location
- `features/orders/lib/constants.ts` - Status configs, colors, labels

### Types Location
- `features/orders/types/order.types.ts` - Re-exports from mocks
- `mocks/types.ts` - Source of truth for enums

### Platform Icon Pattern
```typescript
// features/orders/components/order-list/platform-icon.tsx
PLATFORM_CONFIG: Record<PlatformType, { letter: string; bgClass: string }>
```

---

## Required New Status Options (Issue #77)

| Status | Background | Border | Text |
|--------|-----------|--------|------|
| Processing | #FEF3C7 | #F59E0B | #B45309 |
| In Design | #EDE9FE | #8B5CF6 | #8B5CF6 |
| In Production | #DBEAFE | #3B82F6 | #1D4ED8 |
| Fulfilled | #D1FAE5 | #10B981 | #059669 |
| Completed | #CCFBF1 | #14B8A6 | #0D9488 |
| Pending | #F3F4F6 | #9CA3AF | #4B5563 |
| Refunded | #E0E7FF | #6366F1 | #4338CA |
| Cancelled | #FEE2E2 | #EF4444 | #EF4444 |
| Trash | #E5E7EB | #6B7280 | #374151 |

**Note:** Current `OrderStatusType` has 7 values. New requirement has 9. Need to add: `PROCESSING`, `IN_PRODUCTION`, `FULFILLED`, `TRASH`. Remove: `CONFIRMED`, `READY_TO_FULFILL`, `SHIPPED`.

---

## Data Dependencies

### MockOrder Fields Used
- `status: OrderStatusType` - need new status values
- `storeId?: string` - for store lookup
- `externalId: string` - move to sidebar Order Info card

### MockStore (stores.data.ts)
```typescript
interface MockStore {
  id: string;
  name: string;
  platform: PlatformType;
  domain: string;
}
```

---

## Files to Create

1. `order-status-dropdown.tsx` - New dropdown component
2. `design-upload-modal.tsx` - File upload dialog
3. `create-task-modal.tsx` - Task creation (position dropdown)

## Files to Modify

1. `page.tsx` - Header actions, layout
2. `order-metadata-card.tsx` - Add Store/External ID, remove button
3. `customer-info-card.tsx` - Remove External ID & Source
4. `order-summary-card.tsx` - Simplify payment row
5. `order-item-card.tsx` - Task button logic
6. `constants.ts` - New status config, platform icons

## Files to Delete (Not Required)
- None (components can be kept for potential reuse)
