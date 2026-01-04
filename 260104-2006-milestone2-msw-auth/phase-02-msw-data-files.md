# Phase 2: Data Files (~1h)

## Context

- Plan: [plan.md](./plan.md)
- Depends on: [Phase 1](./phase-01-msw-types-factory.md)
- Issue: #64

## Overview

Create/update mock data files to use new spec-aligned types. Add tenant.data.ts and columns.data.ts.

## Requirements

1. Create tenant.data.ts with 1 hardcoded tenant
2. Create columns.data.ts with default Kanban columns
3. Update users.data.ts - add tenantId to all users
4. Update orders.data.ts - use new OrderStatus, add full fields
5. Update tasks.data.ts - use new DesignTaskStatus, add columnId/position

## Architecture

### Data Dependency

```
tenant.data.ts (root)
    ↓
users.data.ts (tenantId from tenant)
    ↓
workspaces.data.ts (tenantId, memberIds from users)
    ↓
columns.data.ts (workspaceId from workspaces)
    ↓
orders.data.ts (tenantId)
    ↓
order-items.data.ts (orderId from orders)
    ↓
tasks.data.ts (orderId, orderItemId, columnId, assigneeId)
```

### Hardcoded Tenant

```typescript
export const mockTenant: MockTenant = {
  id: TENANT_ID,
  name: 'Synkao POD',
  slug: 'synkao-pod',
  plan: 'PRO',
  status: 'ACTIVE',
  settings: {
    timezone: 'Asia/Ho_Chi_Minh',
    dateFormat: 'DD/MM/YYYY',
    currency: 'VND',
    autoCreateTask: true,
  },
  createdAt: '2024-01-01T00:00:00Z',
  updatedAt: '2024-01-01T00:00:00Z',
};
```

### Default Columns (match spec)

| Name | Status | Color | SortOrder |
|------|--------|-------|-----------|
| To Do | TODO | #3B82F6 | 1 |
| In Progress | IN_PROGRESS | #F59E0B | 2 |
| Review | REVIEW | #8B5CF6 | 3 |
| Revision | REVISION | #EF4444 | 4 |
| Approved | APPROVED | #10B981 | 5 |

## Related Files

| File | Action |
|------|--------|
| `apps/web/src/mocks/data/tenant.data.ts` | CREATE |
| `apps/web/src/mocks/data/columns.data.ts` | CREATE |
| `apps/web/src/mocks/data/users.data.ts` | UPDATE |
| `apps/web/src/mocks/data/orders.data.ts` | UPDATE |
| `apps/web/src/mocks/data/tasks.data.ts` | UPDATE |
| `apps/web/src/mocks/data/index.ts` | UPDATE |

## Implementation Steps

### Step 1: Create tenant.data.ts

```typescript
import { MockTenant } from '../types';
import { TENANT_ID } from '../factories/mock-factory';

export const mockTenant: MockTenant = {
  id: TENANT_ID,
  name: 'Synkao POD',
  slug: 'synkao-pod',
  plan: 'PRO',
  status: 'ACTIVE',
  settings: { ... },
  createdAt: '2024-01-01T00:00:00Z',
  updatedAt: '2024-01-01T00:00:00Z',
};
```

### Step 2: Create columns.data.ts

```typescript
import { MockColumn } from '../types';
import { COLUMN_IDS, WORKSPACE_IDS } from '../factories/mock-factory';

export const mockColumns: MockColumn[] = [
  {
    id: COLUMN_IDS.todo,
    workspaceId: WORKSPACE_IDS.main,
    name: 'To Do',
    status: 'TODO',
    color: '#3B82F6',
    sortOrder: 1,
    isSystem: true,
  },
  // ... 4 more columns
];

export const findColumn = (id: string) => mockColumns.find(c => c.id === id);
export const findColumnByStatus = (status: string) => mockColumns.find(c => c.status === status);
```

### Step 3: Update users.data.ts

Add `tenantId: TENANT_ID` to all users.
Add `status: 'ACTIVE'` to all users.
Add optional `phone`, `lastLoginAt` fields.

### Step 4: Update orders.data.ts

Replace status values:
- `PENDING` → `PENDING`
- `PROCESSING` → `CONFIRMED` or `IN_DESIGN`
- `SHIPPED` → `SHIPPED`
- `DELIVERED` → `COMPLETED`
- `FAILED` → `CANCELLED`

Add new fields:
- `tenantId`
- `designStatus`: derive from items (NONE if no items need design)
- `shippingAddress`: mock Address object
- Money fields: `subtotal`, `total` (amount + currency)
- `source: 'MANUAL'`

### Step 5: Update tasks.data.ts

Replace status values:
- `TODO` → `TODO`
- `IN_PROGRESS` → `IN_PROGRESS`
- `REVIEW` → `REVIEW`
- `REVISION` → `REVISION`
- `DONE` → `APPROVED`

Replace source values:
- `NEW` → `NEW_ORDER`
- `REDO` → `REDESIGN`
- `COMPLAINT` → `COMPLAINT`

Add new fields:
- `tenantId`
- `columnId`: map from status using COLUMN_IDS
- `position`: sequential within column
- `title`: generate from productName
- `description`, `notes`: empty or placeholder

### Step 6: Update data/index.ts

Export new data:
```typescript
export * from './tenant.data';
export * from './columns.data';
```

## Todo List

- [ ] Create tenant.data.ts
- [ ] Create columns.data.ts with 5 columns
- [ ] Update users.data.ts - add tenantId, status
- [ ] Update orders.data.ts - new status enum, full fields
- [ ] Update tasks.data.ts - new status enum, columnId, position
- [ ] Update order-items.data.ts if needed
- [ ] Update data/index.ts exports
- [ ] Verify imports work

## Success Criteria

- [ ] tenant.data.ts exports mockTenant
- [ ] columns.data.ts exports mockColumns, findColumn, findColumnByStatus
- [ ] All users have tenantId = TENANT_ID
- [ ] Orders use spec OrderStatus values
- [ ] Tasks use spec DesignTaskStatus values
- [ ] Tasks have columnId matching their status
- [ ] All data files import successfully in index.ts
