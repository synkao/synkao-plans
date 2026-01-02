# Phase 02: Create Data Files

## Context Links
- Parent: [plan.md](./plan.md)
- Dependency: [phase-01-update-types.md](./phase-01-update-types.md)

## Overview
- **Priority:** High
- **Status:** Done
- **Effort:** 2h
- **Completed:** 2025-12-24T22:51Z
- **Description:** Tạo factory helpers và data files với realistic Vietnamese data

## Key Insights
- UUID cho internal IDs (realistic, production-like)
- Human-readable codes (PREFIX-XXX) cho display: orderNumber, taskCode, itemCode
- Data relationships phải maintain referential integrity
- Vietnamese names cho realistic testing
- Keep files < 200 lines → split by entity

## Requirements

### Functional
- ID generation helpers
- 6 users (1 Admin, 2 Staff, 3 Designer)
- 2 workspaces với 5 columns mỗi cái
- 15 orders với mixed statuses
- 45 order items (~3/order)
- 45 tasks với distribution: TODO:15, IN_PROGRESS:10, REVIEW:8, REVISION:5, DONE:7
- 20 design versions (cho tasks REVIEW/DONE)

### Non-functional
- Type-safe exports
- Easy to extend
- DRY with factory functions

## Architecture

```
apps/web/src/mocks/
├── data/
│   ├── index.ts              # Re-export all
│   ├── users.data.ts         # 6 users
│   ├── workspaces.data.ts    # 2 workspaces + columns
│   ├── orders.data.ts        # 15 orders
│   ├── order-items.data.ts   # 45 items
│   ├── tasks.data.ts         # 45 tasks
│   └── design-versions.data.ts # 20 versions
└── factories/
    └── mock-factory.ts       # ID generators + helpers
```

## Related Code Files

| Action | File | Description |
|--------|------|-------------|
| Create | `apps/web/src/mocks/factories/mock-factory.ts` | ID generators |
| Create | `apps/web/src/mocks/data/index.ts` | Re-exports |
| Create | `apps/web/src/mocks/data/users.data.ts` | 6 users |
| Create | `apps/web/src/mocks/data/workspaces.data.ts` | 2 workspaces |
| Create | `apps/web/src/mocks/data/orders.data.ts` | 15 orders |
| Create | `apps/web/src/mocks/data/order-items.data.ts` | 45 items |
| Create | `apps/web/src/mocks/data/tasks.data.ts` | 45 tasks |
| Create | `apps/web/src/mocks/data/design-versions.data.ts` | 20 versions |

## Implementation Steps

### Step 1: Create Factory Functions
```typescript
// apps/web/src/mocks/factories/mock-factory.ts

// Pre-generated UUIDs for deterministic mock data
// Use crypto.randomUUID() or online generator to create these

// Date helpers
export const daysAgo = (days: number): string =>
  new Date(Date.now() - days * 24 * 60 * 60 * 1000).toISOString();

// Export pre-defined UUIDs for reference consistency
export const USER_IDS = {
  admin: 'a1b2c3d4-e5f6-4a5b-8c9d-0e1f2a3b4c5d',
  staff1: 'b2c3d4e5-f6a7-5b6c-9d0e-1f2a3b4c5d6e',
  staff2: 'c3d4e5f6-a7b8-6c7d-0e1f-2a3b4c5d6e7f',
  designer1: 'd4e5f6a7-b8c9-7d8e-1f2a-3b4c5d6e7f8a',
  designer2: 'e5f6a7b8-c9d0-8e9f-2a3b-4c5d6e7f8a9b',
  designer3: 'f6a7b8c9-d0e1-9f0a-3b4c-5d6e7f8a9b0c',
} as const;

// Human-readable code generators
export const orderCode = (n: number) => `ORD-${String(n).padStart(3, '0')}`;
export const itemCode = (n: number) => `ITM-${String(n).padStart(3, '0')}`;
export const taskCode = (n: number) => `TSK-${String(n).padStart(3, '0')}`;
```

### Step 2: Create Users Data
```typescript
// apps/web/src/mocks/data/users.data.ts
import type { MockUser } from '../types';
import { USER_IDS } from '../factories/mock-factory';

export const mockUsers: MockUser[] = [
  { id: USER_IDS.admin, email: 'admin@synkao.com', name: 'Trần Văn Admin', role: 'ADMIN' },
  { id: USER_IDS.staff1, email: 'staff1@synkao.com', name: 'Nguyễn Thị Hoa', role: 'STAFF' },
  { id: USER_IDS.staff2, email: 'staff2@synkao.com', name: 'Lê Văn Dũng', role: 'STAFF' },
  { id: USER_IDS.designer1, email: 'designer1@synkao.com', name: 'Phạm Minh Tuấn', role: 'DESIGNER' },
  { id: USER_IDS.designer2, email: 'designer2@synkao.com', name: 'Hoàng Thị Lan', role: 'DESIGNER' },
  { id: USER_IDS.designer3, email: 'designer3@synkao.com', name: 'Vũ Đức Anh', role: 'DESIGNER' },
];

// Helper to find user
export const findUser = (id: string) => mockUsers.find(u => u.id === id);
export const getDesigners = () => mockUsers.filter(u => u.role === 'DESIGNER');
```

### Step 3: Create Workspaces Data
```typescript
// apps/web/src/mocks/data/workspaces.data.ts
import type { MockWorkspace, MockKanbanColumn } from '../types';
import { USER_IDS } from '../factories/mock-factory';

// Pre-generated workspace UUIDs
const WKS_MAIN = '11111111-1111-4111-8111-111111111111';
const WKS_ARCHIVE = '22222222-2222-4222-8222-222222222222';

export const mockWorkspaces: MockWorkspace[] = [
  {
    id: WKS_MAIN,
    name: 'Workspace Chính',
    ownerId: USER_IDS.admin,
    memberIds: [USER_IDS.staff1, USER_IDS.staff2, USER_IDS.designer1, USER_IDS.designer2, USER_IDS.designer3],
  },
  {
    id: WKS_ARCHIVE,
    name: 'Archive',
    ownerId: USER_IDS.admin,
    memberIds: [USER_IDS.staff1],
  },
];

// Default columns for kanban board
export const defaultColumns: Omit<MockKanbanColumn, 'tasks'>[] = [
  { id: 'todo', name: 'TODO' },
  { id: 'in-progress', name: 'IN PROGRESS' },
  { id: 'review', name: 'REVIEW' },
  { id: 'revision', name: 'REVISION' },
  { id: 'done', name: 'DONE' },
];
```

### Step 4: Create Orders Data
- 15 orders với Vietnamese customer names
- Each order có `orderNumber`: ORD-001 → ORD-015
- Mix of statuses: PENDING, PROCESSING, SHIPPED, DELIVERED, FAILED
- Use daysAgo() for realistic createdAt dates

### Step 5: Create Order Items Data
- 45 items linked to orders (~3 per order)
- Each item có `itemCode`: ITM-001 → ITM-045
- Product types: T_SHIRT, MUG, CANVAS, HOODIE, POSTER
- Vietnamese product names

### Step 6: Create Tasks Data
- 45 tasks linked to order items (1:1)
- Each task có `taskCode`: TSK-001 → TSK-045
- Distribution: TODO:15, IN_PROGRESS:10, REVIEW:8, REVISION:5, DONE:7
- Assign designers for non-TODO tasks
- Priority mix

### Step 7: Create Design Versions Data
- 20 versions for tasks in REVIEW/DONE status
- Multiple versions per task (1-3)
- Some with REJECTED status + reason

### Step 8: Create Index Re-export
```typescript
// apps/web/src/mocks/data/index.ts
export * from './users.data';
export * from './workspaces.data';
export * from './orders.data';
export * from './order-items.data';
export * from './tasks.data';
export * from './design-versions.data';
```

## Todo List

- [x] Create `factories/mock-factory.ts` với UUID constants + date helpers
- [x] Create `data/users.data.ts` (6 users)
- [x] Create `data/workspaces.data.ts` (2 workspaces)
- [x] Create `data/orders.data.ts` (15 orders)
- [x] Create `data/order-items.data.ts` (45 items)
- [x] Create `data/tasks.data.ts` (45 tasks)
- [x] Create `data/design-versions.data.ts` (20 versions)
- [x] Create `data/index.ts` re-export
- [x] Verify referential integrity
- [x] TypeScript compilation passes

## Success Criteria

- [x] All data files created và exported
- [x] ID format consistent (UUID)
- [x] Referential integrity (valid foreign keys)
- [x] Data volume matches spec
- [x] Vietnamese names/context

## Completion Summary

**Files Created:**
- apps/web/src/mocks/factories/mock-factory.ts
- apps/web/src/mocks/data/users.data.ts
- apps/web/src/mocks/data/workspaces.data.ts
- apps/web/src/mocks/data/orders.data.ts
- apps/web/src/mocks/data/order-items.data.ts
- apps/web/src/mocks/data/tasks.data.ts
- apps/web/src/mocks/data/design-versions.data.ts
- apps/web/src/mocks/data/index.ts

**Data Volume Delivered:**
- 6 users (1 Admin, 2 Staff, 3 Designer)
- 2 workspaces with 5 columns each
- 15 orders with mixed statuses
- 45 order items (~3 per order)
- 45 tasks with proper status distribution
- 20 design versions for REVIEW/DONE tasks

**Quality:**
- Type-safe exports with full TypeScript support
- UUID-based IDs with pre-generated constants
- Human-readable codes (ORD-XXX, ITM-XXX, TSK-XXX)
- Referential integrity maintained across entities
- Vietnamese names/context for realistic testing
- DRY factory pattern for easy extension

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Inconsistent foreign keys | High | Use factory functions, manual verify |
| File > 200 lines | Medium | Split by entity, use loops |

## Security Considerations
- N/A (mock data only)

## Next Steps
- Proceed to Phase 03: Update Handlers
