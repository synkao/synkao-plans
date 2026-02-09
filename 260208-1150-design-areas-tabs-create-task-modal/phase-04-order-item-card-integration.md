# Phase 04: OrderItemCard Integration & Modularization

## Context Links

- [OrderItemCard](../../apps/web/src/features/orders/components/order-detail/order-item-card.tsx) - 324 lines, needs modularization
- [Order detail page](../../apps/web/src/app/(main)/orders/[id]/page.tsx) - Consumer of OrderItemCard
- [Phase 02 - DesignAreaTabs](./phase-02-design-area-tabs-component.md)
- [Phase 03 - CreateTaskModal](./phase-03-create-task-modal-component.md)

## Overview

- **Priority:** P2
- **Status:** completed
- **Description:** Integrate DesignAreaTabs and CreateTaskModal into OrderItemCard; modularize OrderItemCard (324 lines > 200 line limit)

## Key Insights

- `OrderItemCard` is 324 lines, exceeding 200 line limit per project rules
- Current `onCreateTask` callback only fires `toast.info` on the page level
- Need to replace callback-only approach with modal integration
- Card layout has clear sections: thumbnail, content (header, status, attributes, personalization, design file, notes, upload trigger, actions)
- Integration points: DesignAreaTabs goes after personalization/notes section; CreateTaskModal wired to "+ Create Task" button

## Requirements

### Functional
- Insert `DesignAreaTabs` between notes/upload-trigger section and actions bar
- Wire `CreateTaskModal` to the "+ Create Task" button
- Modal needs `orderNumber` - must pass through from page
- Keep existing action button logic intact
- Modularize card into smaller sub-components

### Non-Functional
- Main `order-item-card.tsx` under 200 lines after modularization
- Extracted modules also under 200 lines each
- No visual regressions - same rendering as before plus new features
- Maintain all existing props backward compatible

## Architecture

### Modularization Plan

```
order-item-card.tsx (main, ~120 lines)
  ├── Imports sub-components
  ├── OrderItemCard component
  │   ├── Thumbnail (inline, small)
  │   ├── Header (inline)
  │   ├── OrderItemCardDetails (extracted)
  │   ├── DesignAreaTabs (from Phase 02)
  │   ├── OrderItemCardActions (extracted)
  │   └── CreateTaskModal (from Phase 03, controlled)

order-item-card-details.tsx (~80 lines)
  ├── Attributes display
  ├── Personalization display
  ├── Design file info
  ├── Notes
  └── Empty design upload trigger

order-item-card-actions.tsx (~80 lines)
  ├── renderTaskButton logic (moved here)
  ├── View/Download buttons
  ├── Upload/Replace buttons
  └── Task status badge
```

### Updated Props

```typescript
export interface OrderItemCardProps {
  item: MockOrderItem;
  orderNumber: string;        // NEW - for CreateTaskModal message
  designerName?: string;
  taskStatus?: DesignTaskStatusType | null;
  onViewDesign?: (item: MockOrderItem) => void;
  onDownloadDesign?: (item: MockOrderItem) => void;
  onCreateTask?: (item: MockOrderItem) => void;   // Keep for external handler
  onUploadDesign?: (item: MockOrderItem) => void;
  onReplaceDesign?: (item: MockOrderItem) => void;
}
```

## Related Code Files

### Files to Create
- `apps/web/src/features/orders/components/order-detail/order-item-card-details.tsx`
- `apps/web/src/features/orders/components/order-detail/order-item-card-actions.tsx`

### Files to Modify
- `apps/web/src/features/orders/components/order-detail/order-item-card.tsx` - Refactor, integrate new components
- `apps/web/src/features/orders/components/order-detail/index.ts` - Export new sub-components if needed
- `apps/web/src/app/(main)/orders/[id]/page.tsx` - Pass `orderNumber` prop to OrderItemCard

## Implementation Steps

### 1. Extract OrderItemCardDetails

File: `order-item-card-details.tsx`

Extract these sections from current OrderItemCard:
- Attributes display (lines 197-207)
- Personalization display (lines 209-223)
- Design file info display (lines 225-235)
- Notes display (lines 237-242)
- Empty design upload trigger (lines 244-262)

Props:
```typescript
interface OrderItemCardDetailsProps {
  item: MockOrderItem;
  hasDesign: boolean;
  hasDesignFile: boolean;
}
```

### 2. Extract OrderItemCardActions

File: `order-item-card-actions.tsx`

Extract the actions bar (lines 264-318) and `renderTaskButton` logic (lines 84-145).

Props:
```typescript
interface OrderItemCardActionsProps {
  item: MockOrderItem;
  taskStatus?: DesignTaskStatusType | null;
  hasDesign: boolean;
  hasDesignFile: boolean;
  isDesignApproved: boolean;
  onViewDesign?: (item: MockOrderItem) => void;
  onDownloadDesign?: (item: MockOrderItem) => void;
  onCreateTask?: (item: MockOrderItem) => void;
  onUploadDesign?: (item: MockOrderItem) => void;
  onReplaceDesign?: (item: MockOrderItem) => void;
}
```

### 3. Refactor OrderItemCard

After extraction, main file:
- Imports + type definitions
- Derived state (hasDesign, hasDesignFile, etc.)
- Thumbnail + header section
- `<OrderItemCardDetails />`
- `<DesignAreaTabs />` (from Phase 02)
- `<OrderItemCardActions />` with CreateTask wired to modal
- `<CreateTaskModal />` (controlled, from Phase 03)

Add state for modal:
```typescript
const [createTaskOpen, setCreateTaskOpen] = useState(false);
```

Wire "+ Create Task" button: instead of calling `onCreateTask` directly, open the modal:
```typescript
onCreateTask={(item) => {
  setCreateTaskOpen(true);
  onCreateTask?.(item); // still call external handler if provided
}}
```

### 4. Update Order Detail page

In `apps/web/src/app/(main)/orders/[id]/page.tsx`, add `orderNumber` prop:

```tsx
<OrderItemCard
  key={item.id}
  item={item}
  orderNumber={order.orderNumber}   // NEW
  onViewDesign={handleViewDesign}
  onDownloadDesign={handleDownloadDesign}
  onCreateTask={handleCreateTask}
  onUploadDesign={handleUploadDesign}
  onReplaceDesign={handleReplaceDesign}
/>
```

### 5. Move utility functions

Move `formatFileSize` and `formatDate` from `order-item-card.tsx` to either:
- `order-item-card-details.tsx` (used only there), OR
- A shared utils file if used elsewhere

**Recommended:** Keep in `order-item-card-details.tsx` since only used there.

## Todo List

- [ ] Create `order-item-card-details.tsx` with attributes, personalization, design file, notes, upload trigger
- [ ] Create `order-item-card-actions.tsx` with task button logic and action buttons
- [ ] Move `formatFileSize` and `formatDate` to details component
- [ ] Refactor `order-item-card.tsx` to use extracted sub-components
- [ ] Add `orderNumber` prop to `OrderItemCardProps`
- [ ] Add `createTaskOpen` state and wire to modal
- [ ] Integrate `DesignAreaTabs` into card layout
- [ ] Integrate `CreateTaskModal` (controlled)
- [ ] Update `index.ts` exports
- [ ] Update order detail page to pass `orderNumber`
- [ ] Verify each file under 200 lines
- [ ] Verify compile
- [ ] Visual regression check - same appearance as before + new features

## Success Criteria

- `order-item-card.tsx` under 200 lines
- `order-item-card-details.tsx` under 200 lines
- `order-item-card-actions.tsx` under 200 lines
- DesignAreaTabs visible for items with `designAreas`
- "+ Create Task" button opens Design Service modal
- Modal pre-fills with item data
- All existing functionality preserved
- Order detail page passes `orderNumber` prop
- No compile errors

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Modularization breaks existing styling/layout | Visual regression | Test visually after each extraction step |
| Prop drilling deep (item + callbacks) | Complexity | Keep flat - 2 levels max (card -> sub-component) |
| CreateTaskModal and external onCreateTask conflict | Confusing UX | Modal takes priority; external callback for analytics/logging |

## Security Considerations

- No new security concerns - same data flow as existing
- CreateTaskModal message content from mock data only

## Next Steps

- Run compile check and visual verification
- Future: connect CreateTaskModal to real API endpoint
- Future: connect DesignAreaTabs to real upload/view endpoints

## Unresolved Questions

1. Should active design area tabs show a popover with file info, or just toast? (Defaulting to toast for MVP)
2. Should the DESIGN_POSITION_OPTIONS list be product-type-dependent? (e.g., mugs don't have sleeves) - Defaulting to show all positions for all items per screenshot
