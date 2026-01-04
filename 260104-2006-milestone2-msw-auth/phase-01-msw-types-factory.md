# Phase 1: Types + Factory (~45min)

## Context

- Plan: [plan.md](./plan.md)
- Spec: [synkao-models-spec.md](../../docs/backend/synkao-models-spec.md)
- Issue: #64

## Overview

Update `types.ts` with full spec v1.2 interfaces and enums. Update `mock-factory.ts` with tenant/column ID generators.

## Requirements

1. Add all enums from spec (OrderStatus, DesignTaskStatus, Priority, etc.)
2. Update MockUser to include tenantId
3. Add MockTenant interface
4. Add MockColumn interface
5. Update MockTask with full spec fields
6. Update MockOrder with full spec fields
7. Add tenant/column UUID generators to factory

## Architecture

### Enum Mapping (Spec → Mock)

| Spec Enum | Mock Const |
|-----------|------------|
| `OrderStatus` | Keep all 7 values |
| `DesignTaskStatus` | Map DONE→APPROVED |
| `Priority` | Same 4 values |
| `TaskSource` | NEW→NEW_ORDER, REDO→REDESIGN |
| `UserStatus` | ACTIVE, INACTIVE, INVITED |
| `RoleType` | owner, staff, designer |

### Key Changes

**types.ts:**
```typescript
// Enums as const objects
export const OrderStatus = { ... } as const;
export const DesignTaskStatus = { ... } as const;

// Interfaces with full spec fields
export interface MockTenant { ... }
export interface MockUser { tenantId: string; ... }
export interface MockTask { tenantId, workspaceId, columnId, position, ... }
```

**mock-factory.ts:**
```typescript
export const TENANT_ID = 'aaaaaaaa-0000-4000-8000-000000000001';
export const columnUUID = (n: number) => generateUUID('00000007', n);
```

## Related Files

| File | Action |
|------|--------|
| `apps/web/src/mocks/types.ts` | UPDATE |
| `apps/web/src/mocks/factories/mock-factory.ts` | UPDATE |

## Implementation Steps

### Step 1: Add enums to types.ts

Add const enum objects:
- `TenantStatus`: ACTIVE, SUSPENDED, DELETED
- `PlanType`: FREE, PRO, ENTERPRISE
- `UserStatus`: ACTIVE, INACTIVE, INVITED
- `RoleType`: owner, staff, designer
- `Platform`: WOOCOMMERCE, SHOPIFY, MANUAL
- `OrderStatus`: PENDING, CONFIRMED, IN_DESIGN, READY_TO_FULFILL, SHIPPED, COMPLETED, CANCELLED
- `DesignPhaseStatus`: NONE, PENDING, IN_PROGRESS, COMPLETED
- `OrderSource`: SYNC, CSV_IMPORT, MANUAL
- `ItemDesignStatus`: NOT_REQUIRED, PENDING, IN_PROGRESS, APPROVED
- `DesignTaskStatus`: PENDING, TODO, IN_PROGRESS, REVIEW, REVISION, APPROVED, BLOCKED
- `Priority`: LOW, NORMAL, HIGH, URGENT
- `TaskSource`: NEW_ORDER, REDESIGN, COMPLAINT

### Step 2: Add MockTenant interface

```typescript
export interface MockTenant {
  id: string;
  name: string;
  slug: string;
  plan: typeof PlanType[keyof typeof PlanType];
  status: typeof TenantStatus[keyof typeof TenantStatus];
  settings: {
    timezone: string;
    dateFormat: string;
    currency: string;
    autoCreateTask: boolean;
  };
  createdAt: string;
  updatedAt: string;
}
```

### Step 3: Update MockUser interface

Add fields: `tenantId`, `phone`, `status`, `lastLoginAt`

### Step 4: Add MockColumn interface

```typescript
export interface MockColumn {
  id: string;
  workspaceId: string;
  name: string;
  status: typeof DesignTaskStatus[keyof typeof DesignTaskStatus];
  color: string;
  sortOrder: number;
  wipLimit?: number;
  isSystem: boolean;
}
```

### Step 5: Update MockTask interface

Add full spec fields:
- `tenantId`, `columnId`, `position`
- `assignedBy`, `assignedAt`
- `currentDesignId`, `designCount`
- `title`, `description`, `notes`
- `reviewerId`, `reviewedAt`, `reviewNotes`
- `startedAt`, `completedAt`, `deletedAt`

Update status enum: DONE→APPROVED
Update source enum: NEW→NEW_ORDER, REDO→REDESIGN

### Step 6: Update MockOrder interface

Add full spec fields:
- `tenantId`, `storeId`
- `customerEmail`, `customerPhone`
- `shippingAddress` (Address object)
- `designStatus`
- `subtotal`, `shippingFee`, `discount`, `total`, `currency`
- `fulfillerName`, `trackingNumber`, `carrier`, `shippedAt`
- `notes`, `tags`, `source`
- `orderedAt`, `deletedAt`

Update status enum to match spec: PENDING, CONFIRMED, etc.

### Step 7: Update mock-factory.ts

Add:
```typescript
export const TENANT_ID = 'aaaaaaaa-0000-4000-8000-000000000001';
export const columnUUID = (n: number) => generateUUID('00000007', n);

export const COLUMN_IDS = {
  todo: columnUUID(1),
  inProgress: columnUUID(2),
  review: columnUUID(3),
  revision: columnUUID(4),
  approved: columnUUID(5),
} as const;
```

## Todo List

- [ ] Add all enums as const objects
- [ ] Add MockTenant interface
- [ ] Update MockUser with tenantId, status fields
- [ ] Add MockColumn interface
- [ ] Update MockTask with full spec fields
- [ ] Update MockOrder with full spec fields
- [ ] Update MockOrderItem if needed
- [ ] Add TENANT_ID to mock-factory.ts
- [ ] Add columnUUID generator
- [ ] Add COLUMN_IDS const
- [ ] Run typecheck to verify

## Success Criteria

- [ ] All 12 enums defined as const objects
- [ ] MockTenant interface added
- [ ] MockUser has tenantId field
- [ ] MockColumn interface added
- [ ] MockTask matches spec DesignTask
- [ ] MockOrder matches spec Order
- [ ] TENANT_ID and COLUMN_IDS exported from factory
- [ ] `pnpm typecheck` passes (types only)
