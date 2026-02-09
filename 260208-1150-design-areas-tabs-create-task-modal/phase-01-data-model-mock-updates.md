# Phase 01: Data Model & Mock Updates

## Context Links

- [Mock types](../../apps/web/src/mocks/types.ts)
- [Mock order items data](../../apps/web/src/mocks/data/order-items.data.ts)
- [Constants](../../apps/web/src/features/orders/lib/constants.ts)
- [DesignUploadModal](../../apps/web/src/features/orders/components/order-detail/design-upload-modal.tsx)
- [Order types re-exports](../../apps/web/src/features/orders/types/order.types.ts)

## Overview

- **Priority:** P2
- **Status:** completed
- **Description:** Extend `MockOrderItem` with `designAreas` field, update mock data for 3 demo items, add `position` prop to `DesignUploadModal`

## Key Insights

- `DesignPositionType` already defined in `constants.ts` (11 values: FRONT, BACK, LEFT_SLEEVE, etc.)
- `DesignFileInfo` interface already exists in `mocks/types.ts` (url, fileName, fileSize, uploadedAt)
- Mock data generates 60 items (3 per order, 20 orders); first order's 3 items ideal for demo
- `DesignUploadModal` currently has no position awareness; title says "Upload Design"

## Requirements

### Functional
- Add optional `designAreas` field to `MockOrderItem`
- Each design area maps a `DesignPositionType` to `DesignFileInfo | null`
- `null` value = area active (tab shows blue) but no file uploaded yet
- `undefined` (key absent) = area inactive (tab shows gray)
- Update first order's 3 items with varied design area states
- `DesignUploadModal` accepts optional `position` prop, shows in title

### Non-Functional
- Backward compatible - existing items without `designAreas` work unchanged
- Type-safe with `DesignPositionType` from constants

## Architecture

```
MockOrderItem (updated)
  ├── ...existing fields
  └── designAreas?: Partial<Record<DesignPositionType, DesignFileInfo | null>>

DesignUploadModal (updated)
  ├── ...existing props
  └── position?: string  // e.g. "Front", "Back"
      → title: "Upload Design - {Position}"
```

## Related Code Files

### Files to Modify
- `apps/web/src/mocks/types.ts` - Add `designAreas` to `MockOrderItem`
- `apps/web/src/mocks/data/order-items.data.ts` - Update 3 demo items
- `apps/web/src/features/orders/components/order-detail/design-upload-modal.tsx` - Add `position` prop
- `apps/web/src/features/orders/types/order.types.ts` - Re-export `DesignPositionType` if needed

### Files to Create
- None

## Implementation Steps

### 1. Update MockOrderItem interface
In `apps/web/src/mocks/types.ts`:

```typescript
import type { DesignPositionType } from '@/features/orders/lib/constants';

export interface MockOrderItem {
  // ...existing fields
  designAreas?: Partial<Record<DesignPositionType, DesignFileInfo | null>>;
}
```

**Note:** `DesignPositionType` is defined in constants.ts. To avoid circular imports, either:
- (a) Move `DesignPositionType` to mocks/types.ts and re-export from constants, OR
- (b) Define a standalone type in mocks/types.ts: `type DesignPositionType = 'FRONT' | 'BACK' | ...`
- **Recommended:** Option (b) - define directly in mocks/types.ts to keep mock layer self-contained, then re-export from constants.ts

### 2. Update mock data for first 3 items
In `apps/web/src/mocks/data/order-items.data.ts`, after the loop, override first 3 items:

```typescript
// Demo items for Design Areas feature (first order's 3 items)
// Item 1: Classic Black Tee - no design areas → Create Task enabled
// (leave designAreas undefined - default behavior)

// Item 2: Color Changing Magic Mug - Front + Back active
mockOrderItems[1]!.designAreas = {
  FRONT: null,  // active, no file
  BACK: null,   // active, no file
};
mockOrderItems[1]!.designStatus = 'IN_PROGRESS';

// Item 3: Premium Canvas Artwork - Front has file
mockOrderItems[2]!.designAreas = {
  FRONT: {
    url: 'https://example.com/designs/front-canvas.png',
    fileName: 'front-canvas.png',
    fileSize: 2400000,
    uploadedAt: daysAgo(2),
  },
};
mockOrderItems[2]!.designStatus = 'APPROVED';
```

### 3. Update DesignUploadModal
In `design-upload-modal.tsx`:

```typescript
export interface DesignUploadModalProps {
  itemId: string;
  itemName: string;
  trigger: React.ReactNode;
  position?: string;  // NEW - design position label
  onUploadComplete?: (file: File) => void;
}
```

Update title rendering:
```tsx
<DialogTitle>
  Upload Design{position ? ` - ${position}` : ''}
</DialogTitle>
```

### 4. Re-export DesignPositionType
In `apps/web/src/features/orders/types/order.types.ts`, add:
```typescript
export type { DesignPositionType } from '@/features/orders/lib/constants';
```

## Todo List

- [ ] Define `DesignPositionType` in `mocks/types.ts` (standalone string union)
- [ ] Add `designAreas` optional field to `MockOrderItem` interface
- [ ] Update mock data: override items[1] and items[2] with design areas
- [ ] Add `position` prop to `DesignUploadModalProps`
- [ ] Update modal title to show position when provided
- [ ] Re-export `DesignPositionType` from order types
- [ ] Verify compile: `pnpm --filter web build` or `pnpm --filter web typecheck`

## Success Criteria

- `MockOrderItem.designAreas` typed correctly with `DesignPositionType` keys
- First order: item 1 has no designAreas, item 2 has FRONT+BACK (null), item 3 has FRONT (with file)
- `DesignUploadModal` shows "Upload Design - Front" when position="Front"
- No compile errors
- Backward compatible - existing OrderItemCard renders without changes

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Circular import between mocks/types and constants | Build fail | Define DesignPositionType directly in mocks/types.ts |
| Mock data array indices shift if generation logic changes | Wrong demo data | Add explicit check/comment for item identification |

## Security Considerations

- None - mock data only, no user input

## Next Steps

- Phase 02 will consume `designAreas` to render tabs
- Phase 03 will use item data for Create Task modal pre-fill
