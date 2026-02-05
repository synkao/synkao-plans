# Scout Report: Order List UI Restructure (#76)

## Summary

Scouted Order List components, data models, and API for Issue #76 implementation.

---

## 1. UI Components

**Location:** `apps/web/src/features/orders/components/order-list/`

| File | Purpose |
|------|---------|
| `order-list-table.tsx` | Main table, manages `expandedIds: Set<string>`, lazy-loads items |
| `order-row.tsx` | Order header row - Expand, Checkbox, Order, Buyer, Status, Cost, FF, Note, Actions |
| `order-item-row.tsx` | Item sub-row - Tree indicator, thumbnail, SKU, attributes, design badges |
| `constants.ts` | Column widths (expand: w-8, checkbox: w-10, etc.) in `lib/` |

**Key Patterns:**
- Expandable: `Set<string>` tracks expanded order IDs
- Lazy loading: `useOrderItems(orderId, { enabled: expanded })`
- No "expand all" button exists currently

---

## 2. Data Model

**Location:** `apps/web/src/mocks/types.ts`

### MockOrder Fields (relevant)
```typescript
externalId: string        // ✅ EXISTS - "WC-1001", etc.
storeId?: string          // Links to MockStore
source: OrderSourceType   // SYNC | CSV_IMPORT | MANUAL
```

### MockOrderItem Fields (relevant)
```typescript
designStatus: ItemDesignStatusType  // NOT_REQUIRED | PENDING | IN_PROGRESS | APPROVED
designFileInfo?: DesignFileInfo     // url, fileName, fileSize, uploadedAt
```

### MockStore Fields
```typescript
id: string
name: string              // "WooCommerce Main", "Shopify US"
platform: PlatformType    // WOOCOMMERCE | SHOPIFY | MANUAL
```

### Missing Fields
- ❌ `store_domain` - NOT in model (only `storeId` + `name`)
- ❌ Task relationship on item - Tasks linked via `orderId` in separate entity

---

## 3. API/Services

**Location:** `apps/web/src/features/orders/`

| File | Purpose |
|------|---------|
| `lib/orders-client.ts` | HTTP client - getOrders, getOrderItems, etc. |
| `hooks/use-orders.ts` | React Query hooks - useOrders, useOrderItems |
| `types/order.types.ts` | API request/response types |

**Mock Handlers:** `apps/web/src/mocks/handlers/orders.ts`

**Task Data:** `apps/web/src/mocks/data/tasks.data.ts` - 60 tasks (1:1 with items)

---

## 4. Implementation Gap Analysis

| Issue Requirement | Current State | Action Needed |
|-------------------|---------------|---------------|
| Toggle Expand All | ❌ Not exists | Add button + state in `order-list-table.tsx` |
| External ID display | ✅ `externalId` exists | Add display in `order-row.tsx` |
| Platform Icon + Domain | ⚠️ Platform exists, domain NOT | Add domain to Store model or extract from name |
| Remove Order DESIGN/TASK cols | ✅ Not there | No change needed |
| Item DESIGN column | ⚠️ Has `designStatus` | Add "Missing Design" badge logic |
| Item TASK column | ❌ Task not linked | Need to fetch/link tasks to items |

---

## 5. Files to Modify

### Primary
1. `apps/web/src/features/orders/components/order-list/order-list-table.tsx` - Add expand all toggle
2. `apps/web/src/features/orders/components/order-list/order-row.tsx` - Add external ID, platform icon
3. `apps/web/src/features/orders/components/order-list/order-item-row.tsx` - Add DESIGN & TASK columns

### Secondary
4. `apps/web/src/mocks/types.ts` - Add `domain` to MockStore if needed
5. `apps/web/src/mocks/data/stores.data.ts` - Add domain data
6. `apps/web/src/features/orders/hooks/use-orders.ts` - May need task fetching hook

---

## Unresolved Questions

1. **Store Domain Source:** Add new `domain` field to Store model, or parse from existing `name`?
2. **Task-Item Linking:** How to efficiently fetch task status per order item? Batch endpoint?
3. **Create Task Flow:** `+ Create Task` button opens modal or navigates to task page?
