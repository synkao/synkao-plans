---
title: "Phase 02: Data Generation Updates"
status: pending
priority: P1
effort: 1h
---

# Phase 02: Data Generation Updates

**Parent:** [plan.md](./plan.md)
**Dependencies:** Phase 01 (types.ts, mock-factory.ts)

## Overview

Update data files để generate mock data theo types mới, maintain referential integrity.

## Key Insights (từ Research)

- Dùng seeded faker cho reproducible results (optional, hiện dùng deterministic generators)
- Referential integrity: parent entities trước, children sau
- Sequential IDs cho test clarity

## Requirements

1. NEW: tenants.data.ts - 1 tenant
2. NEW: columns.data.ts - 10 columns (5 per workspace)
3. UPDATE: users.data.ts - thêm tenantId, timestamps
4. UPDATE: workspaces.data.ts - thêm tenantId, columns ref
5. UPDATE: orders.data.ts - full 25 fields
6. UPDATE: order-items.data.ts - full 15 fields
7. UPDATE: tasks.data.ts - full 26 fields (MockDesignTask)
8. UPDATE: design-versions.data.ts → designs.data.ts

## Related Files

- `apps/web/src/mocks/data/*.data.ts`
- `apps/web/src/mocks/data/index.ts`

## File Creation/Update Order (Dependency Graph)

```
tenants.data.ts (NEW - root)
    ↓
users.data.ts (UPDATE - refs tenant)
    ↓
workspaces.data.ts (UPDATE - refs tenant)
    ↓
columns.data.ts (NEW - refs workspace)
    ↓
orders.data.ts (UPDATE - refs tenant)
    ↓
order-items.data.ts (UPDATE - refs order)
    ↓
tasks.data.ts (UPDATE - refs order, orderItem, workspace, column)
    ↓
designs.data.ts (RENAME + UPDATE - refs task)
    ↓
index.ts (UPDATE exports)
```

## Implementation Steps

### Step 2.1: Create tenants.data.ts

```typescript
// apps/web/src/mocks/data/tenants.data.ts
import type { MockTenant } from '../types';
import { TENANT_IDS, daysAgo, now } from '../factories/mock-factory';

export const mockTenants: MockTenant[] = [
  {
    id: TENANT_IDS.default,
    name: 'Synkao Demo Shop',
    slug: 'synkao-demo',
    plan: 'PRO',
    status: 'ACTIVE',
    settings: {
      timezone: 'America/New_York',
      dateFormat: 'MM/DD/YYYY',
      currency: 'USD',
      autoCreateTask: true,
    },
    createdAt: daysAgo(90),
    updatedAt: daysAgo(7),
  },
];

export const findTenant = (id: string) => mockTenants.find(t => t.id === id);
export const getDefaultTenant = () => mockTenants[0]!;
```

### Step 2.2: Create columns.data.ts

```typescript
// apps/web/src/mocks/data/columns.data.ts
import type { MockColumn, DesignTaskStatus } from '../types';
import { WORKSPACE_IDS, COLUMN_IDS, daysAgo } from '../factories/mock-factory';

const DEFAULT_COLUMNS: Array<{ name: string; status: DesignTaskStatus; color: string }> = [
  { name: 'To Do', status: 'TODO', color: '#3B82F6' },
  { name: 'In Progress', status: 'IN_PROGRESS', color: '#F59E0B' },
  { name: 'Review', status: 'REVIEW', color: '#8B5CF6' },
  { name: 'Revision', status: 'REVISION', color: '#EF4444' },
  { name: 'Approved', status: 'APPROVED', color: '#10B981' },
];

const createColumnsForWorkspace = (
  workspaceId: string,
  prefix: string,
  startIndex: number
): MockColumn[] =>
  DEFAULT_COLUMNS.map((col, idx) => ({
    id: prefix === 'main'
      ? Object.values(COLUMN_IDS)[idx]!
      : Object.values(COLUMN_IDS)[5 + idx]!,
    workspaceId,
    name: col.name,
    status: col.status,
    color: col.color,
    sortOrder: idx + 1,
    wipLimit: col.status === 'IN_PROGRESS' ? 5 : undefined,
    isSystem: true,
    createdAt: daysAgo(90),
  }));

export const mockColumns: MockColumn[] = [
  ...createColumnsForWorkspace(WORKSPACE_IDS.main, 'main', 0),
  ...createColumnsForWorkspace(WORKSPACE_IDS.archive, 'archive', 5),
];

export const findColumn = (id: string) => mockColumns.find(c => c.id === id);
export const getColumnsByWorkspace = (workspaceId: string) =>
  mockColumns.filter(c => c.workspaceId === workspaceId);
export const getColumnByStatus = (workspaceId: string, status: DesignTaskStatus) =>
  mockColumns.find(c => c.workspaceId === workspaceId && c.status === status);
```

### Step 2.3: Update users.data.ts

```typescript
// apps/web/src/mocks/data/users.data.ts
import type { MockUser } from '../types';
import { USER_IDS, TENANT_IDS, daysAgo } from '../factories/mock-factory';

export const mockUsers: MockUser[] = [
  {
    id: USER_IDS.admin,
    tenantId: TENANT_IDS.default,
    email: 'admin@synkao.com',
    name: 'Admin User',
    phone: '+15551000001',
    role: 'OWNER',
    avatarUrl: 'https://api.dicebear.com/7.x/avataaars/svg?seed=admin',
    status: 'ACTIVE',
    lastLoginAt: daysAgo(0),
    createdAt: daysAgo(90),
    updatedAt: daysAgo(0),
  },
  {
    id: USER_IDS.staff1,
    tenantId: TENANT_IDS.default,
    email: 'staff1@synkao.com',
    name: 'Staff One',
    phone: '+15551000002',
    role: 'STAFF',
    avatarUrl: 'https://api.dicebear.com/7.x/avataaars/svg?seed=staff1',
    status: 'ACTIVE',
    lastLoginAt: daysAgo(1),
    createdAt: daysAgo(60),
    updatedAt: daysAgo(1),
  },
  {
    id: USER_IDS.staff2,
    tenantId: TENANT_IDS.default,
    email: 'staff2@synkao.com',
    name: 'Staff Two',
    role: 'STAFF',
    avatarUrl: 'https://api.dicebear.com/7.x/avataaars/svg?seed=staff2',
    status: 'ACTIVE',
    createdAt: daysAgo(45),
    updatedAt: daysAgo(3),
  },
  {
    id: USER_IDS.designer1,
    tenantId: TENANT_IDS.default,
    email: 'designer1@synkao.com',
    name: 'Alice Designer',
    phone: '+15551000004',
    role: 'DESIGNER',
    avatarUrl: 'https://api.dicebear.com/7.x/avataaars/svg?seed=designer1',
    status: 'ACTIVE',
    lastLoginAt: daysAgo(0),
    createdAt: daysAgo(30),
    updatedAt: daysAgo(0),
  },
  {
    id: USER_IDS.designer2,
    tenantId: TENANT_IDS.default,
    email: 'designer2@synkao.com',
    name: 'Bob Designer',
    role: 'DESIGNER',
    avatarUrl: 'https://api.dicebear.com/7.x/avataaars/svg?seed=designer2',
    status: 'ACTIVE',
    createdAt: daysAgo(25),
    updatedAt: daysAgo(2),
  },
  {
    id: USER_IDS.designer3,
    tenantId: TENANT_IDS.default,
    email: 'designer3@synkao.com',
    name: 'Carol Designer',
    role: 'DESIGNER',
    avatarUrl: 'https://api.dicebear.com/7.x/avataaars/svg?seed=designer3',
    status: 'INVITED',
    createdAt: daysAgo(5),
    updatedAt: daysAgo(5),
  },
];

export const findUser = (id: string) => mockUsers.find(u => u.id === id);
export const getUsersByRole = (role: MockUser['role']) => mockUsers.filter(u => u.role === role);
export const getDesigners = () => getUsersByRole('DESIGNER');
```

### Step 2.4: Update workspaces.data.ts

```typescript
// apps/web/src/mocks/data/workspaces.data.ts
import type { MockWorkspace } from '../types';
import { WORKSPACE_IDS, TENANT_IDS, USER_IDS, daysAgo } from '../factories/mock-factory';

export const mockWorkspaces: MockWorkspace[] = [
  {
    id: WORKSPACE_IDS.main,
    tenantId: TENANT_IDS.default,
    name: 'T-Shirts & Apparel',
    description: 'All apparel design tasks',
    color: '#3B82F6',
    icon: 'shirt',
    isDefault: true,
    sortOrder: 1,
    createdAt: daysAgo(90),
    updatedAt: daysAgo(7),
  },
  {
    id: WORKSPACE_IDS.archive,
    tenantId: TENANT_IDS.default,
    name: 'Posters & Prints',
    description: 'Print products design tasks',
    color: '#10B981',
    icon: 'image',
    isDefault: false,
    sortOrder: 2,
    createdAt: daysAgo(60),
    updatedAt: daysAgo(14),
  },
];

export const findWorkspace = (id: string) => mockWorkspaces.find(w => w.id === id);
export const getDefaultWorkspace = () => mockWorkspaces.find(w => w.isDefault)!;
```

### Step 2.5: Update orders.data.ts

```typescript
// apps/web/src/mocks/data/orders.data.ts
import type { MockOrder, OrderStatus } from '../types';
import {
  orderUUID, orderCode, daysAgo,
  TENANT_IDS, generateAddress, generateMoney
} from '../factories/mock-factory';

const customers = [
  { name: 'John Smith', email: 'john.smith@email.com', phone: '+15551234001' },
  { name: 'Emily Johnson', email: 'emily.j@email.com', phone: '+15551234002' },
  { name: 'Michael Brown', email: 'mike.brown@email.com', phone: '+15551234003' },
  { name: 'Sarah Davis', email: 'sarah.d@email.com', phone: '+15551234004' },
  { name: 'David Wilson', email: 'david.w@email.com', phone: '+15551234005' },
  { name: 'Jessica Martinez', email: 'jess.m@email.com', phone: '+15551234006' },
  { name: 'Christopher Lee', email: 'chris.lee@email.com', phone: '+15551234007' },
  { name: 'Amanda Taylor', email: 'amanda.t@email.com', phone: '+15551234008' },
  { name: 'Daniel Anderson', email: 'dan.a@email.com', phone: '+15551234009' },
  { name: 'Ashley Thomas', email: 'ashley.t@email.com', phone: '+15551234010' },
  { name: 'Matthew Jackson', email: 'matt.j@email.com', phone: '+15551234011' },
  { name: 'Jennifer White', email: 'jen.white@email.com', phone: '+15551234012' },
  { name: 'Andrew Harris', email: 'andrew.h@email.com', phone: '+15551234013' },
  { name: 'Stephanie Martin', email: 'steph.m@email.com', phone: '+15551234014' },
  { name: 'Joshua Thompson', email: 'josh.t@email.com', phone: '+15551234015' },
];

const statusDistribution: OrderStatus[] = [
  'PENDING', 'PENDING', 'PENDING',          // 3
  'CONFIRMED', 'CONFIRMED',                  // 2
  'IN_DESIGN', 'IN_DESIGN', 'IN_DESIGN',    // 3
  'READY_TO_FULFILL', 'READY_TO_FULFILL',   // 2
  'SHIPPED', 'SHIPPED',                      // 2
  'COMPLETED', 'COMPLETED',                  // 2
  'CANCELLED',                               // 1
];

const carriers = ['USPS', 'FEDEX', 'UPS'] as const;

export const mockOrders: MockOrder[] = Array.from({ length: 15 }, (_, i) => {
  const n = i + 1;
  const customer = customers[i]!;
  const status = statusDistribution[i]!;
  const basePrice = 25 + (i * 5);
  const isShipped = ['SHIPPED', 'COMPLETED'].includes(status);

  return {
    id: orderUUID(n),
    tenantId: TENANT_IDS.default,
    storeId: undefined, // Manual import for now
    externalId: `EXT-${String(n).padStart(4, '0')}`,
    orderNumber: orderCode(n),

    // Customer
    customerName: customer.name,
    customerEmail: customer.email,
    customerPhone: customer.phone,

    // Shipping
    shippingAddress: generateAddress(n),

    // Status
    status,
    designStatus: status === 'IN_DESIGN' ? 'IN_PROGRESS'
                : ['READY_TO_FULFILL', 'SHIPPED', 'COMPLETED'].includes(status) ? 'COMPLETED'
                : 'PENDING',

    // Money
    subtotal: generateMoney(basePrice),
    shippingFee: generateMoney(5.99),
    discount: generateMoney(i % 3 === 0 ? 5 : 0),
    total: generateMoney(basePrice + 5.99 - (i % 3 === 0 ? 5 : 0)),
    currency: 'USD',

    // Fulfillment
    fulfillerName: isShipped ? 'PrintfulPOD' : undefined,
    trackingNumber: isShipped ? `1Z${String(n).padStart(16, '0')}` : undefined,
    carrier: isShipped ? carriers[i % 3] : undefined,
    shippedAt: isShipped ? daysAgo(3 - (i % 3)) : undefined,

    // Meta
    notes: i % 4 === 0 ? 'Rush order - priority handling' : undefined,
    tags: i % 2 === 0 ? ['vip', 'repeat'] : [],
    source: 'MANUAL',

    // Timestamps
    orderedAt: daysAgo(15 - i),
    createdAt: daysAgo(15 - i),
    updatedAt: daysAgo(Math.max(0, 7 - i)),
    deletedAt: undefined,
  };
});

export const findOrder = (id: string) => mockOrders.find(o => o.id === id);
export const findOrderByNumber = (orderNumber: string) =>
  mockOrders.find(o => o.orderNumber === orderNumber);
export const getOrdersByStatus = (status: OrderStatus) =>
  mockOrders.filter(o => o.status === status);
```

### Step 2.6: Update order-items.data.ts

```typescript
// apps/web/src/mocks/data/order-items.data.ts
import type { MockOrderItem, ItemDesignStatus } from '../types';
import { itemUUID, itemCode, daysAgo, generateMoney } from '../factories/mock-factory';
import { mockOrders } from './orders.data';

const productTypes = ['T_SHIRT', 'MUG', 'CANVAS', 'HOODIE', 'POSTER'] as const;

const products = [
  { type: 'T_SHIRT', name: 'Classic Cotton Tee', sku: 'TEE-001', price: 24.99 },
  { type: 'HOODIE', name: 'Premium Hoodie', sku: 'HOD-001', price: 49.99 },
  { type: 'MUG', name: 'Ceramic Mug 11oz', sku: 'MUG-001', price: 14.99 },
  { type: 'CANVAS', name: 'Canvas Print 16x20', sku: 'CNV-001', price: 39.99 },
  { type: 'POSTER', name: 'Poster 18x24', sku: 'PST-001', price: 19.99 },
];

const variants = ['Size S - Black', 'Size M - White', 'Size L - Navy', 'Size XL - Gray', 'One Size'];

// 3 items per order = 45 items total
export const mockOrderItems: MockOrderItem[] = mockOrders.flatMap((order, orderIdx) =>
  Array.from({ length: 3 }, (_, itemIdx) => {
    const n = orderIdx * 3 + itemIdx + 1;
    const product = products[n % 5]!;
    const quantity = 1 + (n % 3);
    const needsDesign = product.type !== 'MUG'; // Mugs don't need custom design

    // Determine design status based on order status
    let designStatus: ItemDesignStatus = 'NOT_REQUIRED';
    if (needsDesign) {
      if (order.status === 'PENDING' || order.status === 'CONFIRMED') {
        designStatus = 'PENDING';
      } else if (order.status === 'IN_DESIGN') {
        designStatus = 'IN_PROGRESS';
      } else {
        designStatus = 'APPROVED';
      }
    }

    return {
      id: itemUUID(n),
      orderId: order.id,
      externalId: `EXT-ITEM-${String(n).padStart(4, '0')}`,
      itemCode: itemCode(n),

      // Product
      sku: `${product.sku}-${String(n).padStart(3, '0')}`,
      name: product.name,
      variantName: variants[n % 5],
      productType: product.type as MockOrderItem['productType'],
      quantity,

      // Money
      unitPrice: generateMoney(product.price),
      totalPrice: generateMoney(product.price * quantity),

      // Images
      imageUrl: `https://cdn.synkao.com/products/${product.type.toLowerCase()}-${n}.jpg`,
      thumbnailUrl: `https://cdn.synkao.com/products/${product.type.toLowerCase()}-${n}-thumb.jpg`,

      // Design
      needsDesign,
      designNotes: needsDesign && n % 4 === 0
        ? 'Customer wants bold colors and vintage style'
        : undefined,
      designStatus,

      // Meta
      attributes: n % 3 === 0 ? { customText: 'Happy Birthday!' } : undefined,
      createdAt: order.createdAt,
      updatedAt: order.updatedAt,
    };
  })
);

export const findOrderItem = (id: string) => mockOrderItems.find(i => i.id === id);
export const findOrderItems = (orderId: string) => mockOrderItems.filter(i => i.orderId === orderId);
export const getItemsNeedingDesign = () => mockOrderItems.filter(i => i.needsDesign);
```

### Step 2.7: Update tasks.data.ts → MockDesignTask

```typescript
// apps/web/src/mocks/data/tasks.data.ts
import type { MockDesignTask, DesignTaskStatus } from '../types';
import {
  taskUUID, taskCode, daysAgo,
  USER_IDS, TENANT_IDS, WORKSPACE_IDS, COLUMN_IDS
} from '../factories/mock-factory';
import { mockOrderItems } from './order-items.data';
import { mockOrders } from './orders.data';
import { getColumnByStatus } from './columns.data';

const designerIds = [USER_IDS.designer1, USER_IDS.designer2, USER_IDS.designer3];
const priorities: MockDesignTask['priority'][] = ['LOW', 'NORMAL', 'HIGH', 'URGENT'];

// Status distribution: PENDING:5, TODO:10, IN_PROGRESS:10, REVIEW:8, REVISION:5, APPROVED:7 = 45
const statusDistribution: DesignTaskStatus[] = [
  ...Array(5).fill('PENDING'),
  ...Array(10).fill('TODO'),
  ...Array(10).fill('IN_PROGRESS'),
  ...Array(8).fill('REVIEW'),
  ...Array(5).fill('REVISION'),
  ...Array(7).fill('APPROVED'),
] as DesignTaskStatus[];

// Only create tasks for items that need design
const itemsNeedingDesign = mockOrderItems.filter(item => item.needsDesign);

export const mockTasks: MockDesignTask[] = itemsNeedingDesign.map((item, idx) => {
  const n = idx + 1;
  const status = statusDistribution[idx % statusDistribution.length]!;
  const order = mockOrders.find(o => o.id === item.orderId)!;

  // Workspace assignment: PENDING = no workspace, others = main workspace
  const inWorkspace = status !== 'PENDING';
  const workspaceId = inWorkspace ? WORKSPACE_IDS.main : undefined;
  const column = inWorkspace ? getColumnByStatus(WORKSPACE_IDS.main, status) : undefined;

  // Assignment: TODO+ statuses have assignee
  const needsAssignee = !['PENDING', 'TODO'].includes(status);
  const designerIdx = idx % 3;

  // Review info for REVIEW/REVISION/APPROVED statuses
  const needsReview = ['REVIEW', 'REVISION', 'APPROVED'].includes(status);

  return {
    id: taskUUID(n),
    tenantId: TENANT_IDS.default,
    orderId: order.id,
    orderItemId: item.id,

    // Workspace
    workspaceId,
    columnId: column?.id,
    position: inWorkspace ? (idx % 5) + 1 : 0,

    // Assignment
    assigneeId: needsAssignee ? designerIds[designerIdx] : undefined,
    assignedBy: needsAssignee ? USER_IDS.staff1 : undefined,
    assignedAt: needsAssignee ? daysAgo(7 - (idx % 7)) : undefined,

    // Status
    status,
    priority: priorities[idx % 4]!,
    source: idx % 10 === 0 ? 'REDESIGN' : 'NEW_ORDER',

    // Design
    currentDesignId: ['REVIEW', 'APPROVED'].includes(status)
      ? `design-${n}-v1` : undefined,
    designCount: ['REVIEW', 'APPROVED'].includes(status) ? 1
               : status === 'REVISION' ? 2 : 0,

    // Content
    taskCode: taskCode(n),
    title: `Design for ${item.name}`,
    description: `Create custom design for order ${order.orderNumber}`,
    notes: idx % 5 === 0 ? 'Customer prefers minimalist style' : undefined,

    // Deadline
    dueDate: daysAgo(-7 - (idx % 14)),

    // Review
    reviewerId: needsReview ? USER_IDS.staff1 : undefined,
    reviewedAt: status === 'APPROVED' ? daysAgo(1) : undefined,
    reviewNotes: status === 'REVISION'
      ? 'Please adjust the color contrast and font size' : undefined,

    // Timestamps
    createdAt: daysAgo(14 - (idx % 14)),
    updatedAt: daysAgo(idx % 7),
    startedAt: needsAssignee ? daysAgo(6 - (idx % 6)) : undefined,
    completedAt: status === 'APPROVED' ? daysAgo(1) : undefined,
    deletedAt: undefined,

    // Version
    version: 1,
  };
});

// Helper exports
export const findTask = (id: string) => mockTasks.find(t => t.id === id);
export const findTaskByCode = (code: string) => mockTasks.find(t => t.taskCode === code);
export const getTasksByStatus = (status: DesignTaskStatus) =>
  mockTasks.filter(t => t.status === status);
export const getTasksByAssignee = (assigneeId: string) =>
  mockTasks.filter(t => t.assigneeId === assigneeId);
export const getTasksByWorkspace = (workspaceId: string) =>
  mockTasks.filter(t => t.workspaceId === workspaceId);
export const getBacklogTasks = () => mockTasks.filter(t => !t.workspaceId);
export const getTasksByColumn = (columnId: string) =>
  mockTasks.filter(t => t.columnId === columnId).sort((a, b) => a.position - b.position);
```

### Step 2.8: Rename design-versions.data.ts → designs.data.ts

```typescript
// apps/web/src/mocks/data/designs.data.ts
import type { MockDesign, DesignStatus } from '../types';
import { designUUID, daysAgo, USER_IDS } from '../factories/mock-factory';
import { mockTasks } from './tasks.data';

const fileTypes = ['PNG', 'JPG', 'PDF'] as const;

// Create designs for tasks that have currentDesignId
const tasksWithDesigns = mockTasks.filter(t => t.designCount > 0);

export const mockDesigns: MockDesign[] = tasksWithDesigns.flatMap((task, taskIdx) =>
  Array.from({ length: task.designCount }, (_, versionIdx) => {
    const version = versionIdx + 1;
    const n = taskIdx * 2 + version;
    const isLatest = version === task.designCount;

    // Status: earlier versions are REJECTED, latest matches task status
    let status: DesignStatus = 'DRAFT';
    if (!isLatest) {
      status = 'REJECTED';
    } else if (task.status === 'APPROVED') {
      status = 'APPROVED';
    } else if (['REVIEW', 'REVISION'].includes(task.status)) {
      status = 'SUBMITTED';
    }

    const fileType = fileTypes[n % 3]!;

    return {
      id: designUUID(n),
      taskId: task.id,
      version,

      // File
      fileName: `${task.taskCode.toLowerCase()}-v${version}.${fileType.toLowerCase()}`,
      fileUrl: `https://cdn.synkao.com/designs/${task.taskCode.toLowerCase()}-v${version}.${fileType.toLowerCase()}`,
      fileType,
      fileSize: 1024 * 1024 * (1 + (n % 5)), // 1-5 MB
      thumbnailUrl: `https://cdn.synkao.com/designs/${task.taskCode.toLowerCase()}-v${version}-thumb.jpg`,

      // Upload
      uploadedBy: task.assigneeId || USER_IDS.designer1,
      uploadedAt: daysAgo(7 - version),

      // Review
      status,
      reviewerId: status !== 'DRAFT' ? USER_IDS.staff1 : undefined,
      reviewedAt: ['APPROVED', 'REJECTED'].includes(status) ? daysAgo(5 - version) : undefined,
      reviewNotes: status === 'REJECTED'
        ? 'Colors need adjustment for print quality' : undefined,

      // Metadata
      metadata: fileType !== 'PDF' ? {
        width: 2000 + (n * 100),
        height: 2000 + (n * 100),
        dpi: 300,
        colorSpace: 'sRGB',
      } : undefined,
    };
  })
);

export const findDesign = (id: string) => mockDesigns.find(d => d.id === id);
export const findDesignVersions = (taskId: string) =>
  mockDesigns.filter(d => d.taskId === taskId).sort((a, b) => b.version - a.version);
export const getLatestDesign = (taskId: string) => {
  const versions = findDesignVersions(taskId);
  return versions[0];
};
```

### Step 2.9: Update data/index.ts

```typescript
// apps/web/src/mocks/data/index.ts

// Entities
export * from './tenants.data';
export * from './users.data';
export * from './workspaces.data';
export * from './columns.data';
export * from './orders.data';
export * from './order-items.data';
export * from './tasks.data';
export * from './designs.data';

// Re-export types for convenience
export type {
  MockTenant,
  MockUser,
  MockWorkspace,
  MockColumn,
  MockOrder,
  MockOrderItem,
  MockDesignTask,
  MockDesign,
} from '../types';
```

## Success Criteria

- [ ] All data files import without errors
- [ ] `npm run typecheck` passes
- [ ] Referential integrity: all FKs point to valid parent IDs
- [ ] Data counts match spec: 1 tenant, 6 users, 2 workspaces, 10 columns, 15 orders, 45 items, ~35 tasks, ~20 designs

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Circular imports | HIGH | Follow dependency order strictly |
| Missing parent refs | MEDIUM | Create parent data before children |
| Type errors | LOW | Run typecheck after each file |
