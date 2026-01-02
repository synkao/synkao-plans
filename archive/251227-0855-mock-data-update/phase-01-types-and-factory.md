---
title: "Phase 01: Types & Factory Updates"
status: pending
priority: P1
effort: 45min
---

# Phase 01: Types & Factory Updates

**Parent:** [plan.md](./plan.md)
**Dependencies:** None (foundation phase)

## Overview

Update TypeScript interfaces và factory generators để match spec v1.2.

## Key Insights (từ Research)

- Dùng TypeScript generics cho type-safe handlers
- Sequential UUIDs pattern đã có sẵn, cần extend
- Value objects (Address, Money) cần separate types

## Requirements

1. Add MockTenant interface
2. Update MockOrder với 25 fields
3. Update MockTask → MockDesignTask với 26 fields
4. Update MockColumn với proper fields
5. Update MockDesign (rename từ MockDesignVersion)
6. Add status enums mới
7. Add value object types (Address, Money)

## Related Files

- `apps/web/src/mocks/types.ts`
- `apps/web/src/mocks/factories/mock-factory.ts`

## Implementation Steps

### Step 1.1: Add Status Enums & Value Objects

```typescript
// types.ts - thêm đầu file

// === STATUS ENUMS ===
export type OrderStatus =
  | 'PENDING' | 'CONFIRMED' | 'IN_DESIGN'
  | 'READY_TO_FULFILL' | 'SHIPPED' | 'COMPLETED' | 'CANCELLED';

export type DesignTaskStatus =
  | 'PENDING' | 'TODO' | 'IN_PROGRESS'
  | 'REVIEW' | 'REVISION' | 'APPROVED' | 'BLOCKED';

export type DesignStatus = 'DRAFT' | 'SUBMITTED' | 'APPROVED' | 'REJECTED';

export type Priority = 'LOW' | 'NORMAL' | 'HIGH' | 'URGENT';
export type TaskSource = 'NEW_ORDER' | 'REDESIGN' | 'COMPLAINT';
export type ItemDesignStatus = 'NOT_REQUIRED' | 'PENDING' | 'IN_PROGRESS' | 'APPROVED';
export type FileType = 'PNG' | 'JPG' | 'PDF' | 'AI' | 'PSD' | 'SVG';
export type Carrier = 'USPS' | 'FEDEX' | 'UPS' | 'DHL' | 'OTHER';

// === VALUE OBJECTS ===
export interface Address {
  fullName: string;
  phone: string;
  line1: string;
  line2?: string;
  city: string;
  state: string;      // 2-letter US state code
  postalCode: string; // 5-digit ZIP
  country: string;    // ISO 3166-1 alpha-2 (US)
}

export interface Money {
  amount: number;    // cents (smallest unit)
  currency: string;  // USD
}
```

### Step 1.2: Add MockTenant Interface

```typescript
export interface MockTenant {
  id: string;
  name: string;
  slug: string;
  plan: 'FREE' | 'PRO' | 'ENTERPRISE';
  status: 'ACTIVE' | 'SUSPENDED' | 'DELETED';
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

### Step 1.3: Update MockUser

```typescript
export interface MockUser {
  id: string;
  tenantId: string;          // +NEW
  email: string;
  name: string;
  phone?: string;            // +NEW
  role: 'OWNER' | 'STAFF' | 'DESIGNER';  // Updated values
  avatarUrl?: string;        // renamed from avatar
  status: 'ACTIVE' | 'INACTIVE' | 'INVITED'; // +NEW
  lastLoginAt?: string;      // +NEW
  createdAt: string;         // +NEW
  updatedAt: string;         // +NEW
}
```

### Step 1.4: Update MockOrder (25 fields)

```typescript
export interface MockOrder {
  id: string;
  tenantId: string;
  storeId?: string;
  externalId: string;
  orderNumber: string;

  // Customer
  customerName: string;
  customerEmail?: string;
  customerPhone?: string;

  // Shipping
  shippingAddress: Address;

  // Status
  status: OrderStatus;
  designStatus: 'NONE' | 'PENDING' | 'IN_PROGRESS' | 'COMPLETED';

  // Money
  subtotal: Money;
  shippingFee: Money;
  discount: Money;
  total: Money;
  currency: string;

  // Fulfillment
  fulfillerName?: string;
  trackingNumber?: string;
  carrier?: Carrier;
  shippedAt?: string;

  // Meta
  notes?: string;
  tags: string[];
  source: 'SYNC' | 'CSV_IMPORT' | 'MANUAL';

  // Timestamps
  orderedAt: string;
  createdAt: string;
  updatedAt: string;
  deletedAt?: string;
}
```

### Step 1.5: Update MockOrderItem

```typescript
export interface MockOrderItem {
  id: string;
  orderId: string;
  externalId: string;
  itemCode: string;

  // Product
  sku: string;
  name: string;
  variantName?: string;
  productType: 'T_SHIRT' | 'MUG' | 'CANVAS' | 'HOODIE' | 'POSTER';
  quantity: number;

  // Money
  unitPrice: Money;
  totalPrice: Money;

  // Images
  imageUrl?: string;
  thumbnailUrl?: string;

  // Design
  needsDesign: boolean;
  designNotes?: string;
  designStatus: ItemDesignStatus;

  // Meta
  attributes?: Record<string, string>;
  createdAt: string;
  updatedAt: string;
}
```

### Step 1.6: Update MockDesignTask (26 fields)

```typescript
export interface MockDesignTask {
  id: string;
  tenantId: string;
  orderId: string;
  orderItemId: string;

  // Workspace
  workspaceId?: string;
  columnId?: string;
  position: number;

  // Assignment
  assigneeId?: string;
  assignedBy?: string;
  assignedAt?: string;

  // Status
  status: DesignTaskStatus;
  priority: Priority;
  source: TaskSource;

  // Design
  currentDesignId?: string;
  designCount: number;

  // Content
  taskCode: string;
  title: string;
  description?: string;
  notes?: string;

  // Deadline
  dueDate?: string;

  // Review
  reviewerId?: string;
  reviewedAt?: string;
  reviewNotes?: string;

  // Timestamps
  createdAt: string;
  updatedAt: string;
  startedAt?: string;
  completedAt?: string;
  deletedAt?: string;

  // Optimistic Lock
  version: number;
}
```

### Step 1.7: Update MockColumn

```typescript
export interface MockColumn {
  id: string;
  workspaceId: string;
  name: string;
  status: DesignTaskStatus;
  color: string;           // hex color
  sortOrder: number;
  wipLimit?: number;
  isSystem: boolean;
  createdAt: string;
}
```

### Step 1.8: Update MockDesign (rename từ MockDesignVersion)

```typescript
export interface MockDesign {
  id: string;
  taskId: string;
  version: number;

  // File
  fileName: string;
  fileUrl: string;
  fileType: FileType;
  fileSize: number;        // bytes
  thumbnailUrl?: string;

  // Upload
  uploadedBy: string;
  uploadedAt: string;

  // Review
  status: DesignStatus;
  reviewerId?: string;
  reviewedAt?: string;
  reviewNotes?: string;

  // Metadata
  metadata?: {
    width?: number;
    height?: number;
    dpi?: number;
    colorSpace?: string;
  };
}
```

### Step 1.9: Update MockWorkspace

```typescript
export interface MockWorkspace {
  id: string;
  tenantId: string;
  name: string;
  description?: string;
  color: string;
  icon?: string;
  isDefault: boolean;
  sortOrder: number;
  createdAt: string;
  updatedAt: string;
}
```

### Step 1.10: Update mock-factory.ts

```typescript
// Thêm vào mock-factory.ts

export const TENANT_IDS = {
  default: 'tenant-0001-0000-0000-000000000001',
} as const;

export const COLUMN_IDS = {
  // Workspace 1 (main)
  main_todo: 'col-main-todo-0000-000000000001',
  main_inprogress: 'col-main-inpr-0000-000000000002',
  main_review: 'col-main-revw-0000-000000000003',
  main_revision: 'col-main-revn-0000-000000000004',
  main_approved: 'col-main-appr-0000-000000000005',
  // Workspace 2 (archive)
  archive_todo: 'col-arch-todo-0000-000000000006',
  archive_inprogress: 'col-arch-inpr-0000-000000000007',
  archive_review: 'col-arch-revw-0000-000000000008',
  archive_revision: 'col-arch-revn-0000-000000000009',
  archive_approved: 'col-arch-appr-0000-000000000010',
} as const;

// UUID generators
export const tenantUUID = (n: number) => generateUUID('00000005', n);
export const columnUUID = (n: number) => generateUUID('00000006', n);
export const designUUID = (n: number) => generateUUID('00000007', n);

// Address generator (US)
export const generateAddress = (seed: number): Address => {
  const states = ['CA', 'TX', 'NY', 'FL', 'WA', 'OR', 'IL', 'PA'];
  const cities = ['Los Angeles', 'Houston', 'New York', 'Miami', 'Seattle', 'Portland', 'Chicago', 'Philadelphia'];

  return {
    fullName: `Customer ${seed}`,
    phone: `+1555${String(seed).padStart(7, '0')}`,
    line1: `${100 + seed} Main Street`,
    line2: seed % 3 === 0 ? `Apt ${seed}` : undefined,
    city: cities[seed % cities.length]!,
    state: states[seed % states.length]!,
    postalCode: String(10000 + seed * 111).slice(0, 5),
    country: 'US',
  };
};

// Money generator (USD cents)
export const generateMoney = (dollars: number): Money => ({
  amount: Math.round(dollars * 100),
  currency: 'USD',
});

// Status color mapping
export const TASK_STATUS_COLORS: Record<DesignTaskStatus, string> = {
  PENDING: '#9CA3AF',
  TODO: '#3B82F6',
  IN_PROGRESS: '#F59E0B',
  REVIEW: '#8B5CF6',
  REVISION: '#EF4444',
  APPROVED: '#10B981',
  BLOCKED: '#6B7280',
};
```

### Step 1.11: Keep Backward Compatibility

```typescript
// types.ts - bottom of file

// DEPRECATED: Aliases for backward compatibility
/** @deprecated Use MockDesignTask instead */
export type MockTask = MockDesignTask;
/** @deprecated Use MockDesign instead */
export type MockDesignVersion = MockDesign;
```

## Success Criteria

- [ ] All interfaces compile without errors
- [ ] Factory functions generate valid data
- [ ] Import statements work in data files
- [ ] No breaking changes to existing code (aliases added)

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Breaking existing imports | HIGH | Add type aliases for deprecated names |
| Missing fields at runtime | MEDIUM | Make new fields optional where possible |
| Type mismatches | LOW | Run tsc --noEmit after changes |
