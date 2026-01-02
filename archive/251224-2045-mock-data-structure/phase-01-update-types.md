# Phase 01: Update Types

## Context Links
- Parent: [plan.md](./plan.md)
- Brainstorm: [brainstorm-251224-2025-mock-data-structure.md](../reports/brainstorm-251224-2025-mock-data-structure.md)

## Overview
- **Priority:** High (blocks other phases)
- **Status:** Done
- **Effort:** 30m
- **Description:** Thêm 3 interfaces mới vào types.ts
- **Completed:** 2025-12-24T22:18Z

## Key Insights
- Existing types đã có pattern tốt với clear field naming
- ID format: UUID cho internal reference
- Human-readable codes: PREFIX-XXX cho display (orderNumber, taskCode, etc.)
- Optional fields sử dụng `?` syntax

## Requirements

### Functional
- MockOrderItem: link Order → Items → Task
- MockWorkspace: container cho columns và members
- MockDesignVersion: track design iterations cho tasks

### Non-functional
- Type-safe với TypeScript
- Consistent với existing type patterns

## Architecture

```typescript
// Entity relationships
Order (1) ──→ OrderItem (N) ──→ Task (1)
                                    ↓
Workspace (1) ──→ Column (N)    DesignVersion (N)
     ↓
  User (N)
```

## Related Code Files

| Action | File | Description |
|--------|------|-------------|
| Modify | `apps/web/src/mocks/types.ts` | Add 3 new interfaces |

## Implementation Steps

### Step 1: Add MockOrderItem
```typescript
export interface MockOrderItem {
  id: string;                    // UUID
  itemCode: string;              // ITM-001 (human-readable)
  orderId: string;               // UUID ref
  productType: 'T_SHIRT' | 'MUG' | 'CANVAS' | 'HOODIE' | 'POSTER';
  productName: string;
  designNotes?: string;
  quantity: number;
}
```

### Step 2: Add MockWorkspace
```typescript
export interface MockWorkspace {
  id: string;                    // UUID
  name: string;
  ownerId: string;               // UUID ref (Admin)
  memberIds: string[];           // UUID refs
}
```

### Step 3: Add MockDesignVersion
```typescript
export interface MockDesignVersion {
  id: string;                    // UUID
  taskId: string;                // UUID ref
  version: number;               // 1, 2, 3...
  fileUrl: string;
  status: 'DRAFT' | 'SUBMITTED' | 'APPROVED' | 'REJECTED';
  rejectionReason?: string;
  createdAt: string;
}
```

### Step 4: Update MockTask
```typescript
// Add new fields
orderItemId?: string;  // UUID ref
taskCode: string;      // TSK-001 (human-readable)
```

### Step 5: Update MockOrder
```typescript
// Add human-readable field
orderNumber: string;   // ORD-001 (display)
```

## Todo List

- [x] Add MockOrderItem interface (with itemCode)
- [x] Add MockWorkspace interface
- [x] Add MockDesignVersion interface
- [x] Update MockTask: add orderItemId, taskCode
- [x] Update MockOrder: add orderNumber
- [x] Verify no TypeScript errors

## Success Criteria

- [x] All 3 new interfaces exported
- [x] TypeScript compilation passes
- [x] Field naming consistent với existing types

## Files Modified

- `apps/web/src/mocks/types.ts` - Added MockOrderItem, MockWorkspace, MockDesignVersion; updated MockTask, MockOrder
- `apps/web/src/mocks/handlers/tasks.ts` - Updated for new task fields
- `apps/web/src/mocks/handlers/orders.ts` - Updated for new order fields

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Breaking existing imports | Low | Add only, không modify existing |

## Security Considerations
- N/A (mock data only)

## Next Steps
- Proceed to Phase 02: Create Data Files
