# Phase 3: Orders Module (List, Detail, Import Wizard)

## Context Links
- **Parent Plan:** [plan.md](./plan.md)
- **Dependencies:** [Phase 1](./phase-01-foundation-layout.md), [Phase 2](./phase-02-design-module.md)
- **Design System:** [synkao-design-system.md](../../docs/design/synkao-design-system.md)
- **UI Spec:** Section 6 - Orders Module

---

## Overview

| Property | Value |
|----------|-------|
| Priority | P2 |
| Status | pending |
| Effort | 1.5 days |
| Description | Build Orders module - Order list with expandable rows, Order detail page, CSV Import wizard |

---

## Key Insights (from Research)

1. **Order Data** - 15 orders in `mockOrders`, linked to 45 order items
2. **Expandable Rows** - Show order items inline when clicked
3. **Order Status** - PENDING, PROCESSING, SHIPPED, DELIVERED, FAILED
4. **Import Wizard** - 3 steps: Upload → Preview → Result (static demo, no real upload)
5. **Timeline** - Show order history events

---

## Requirements

### Functional
- F1: Order list with expandable rows showing items
- F2: Tab bar for quick filter by status
- F3: Order detail page with customer info, items, timeline
- F4: Status stepper (visual progress)
- F5: 3-step import wizard UI (hardcoded success)
- F6: Bulk selection with action bar

### Non-Functional
- NF1: Expandable rows animate smoothly
- NF2: Table pagination for large lists
- NF3: Import wizard steps clearly indicated

---

## Architecture

### Component Tree - Order List
```
OrdersPage
├── PageHeader (title, Import/Export buttons)
├── TabBar (All, Pending, Processing, Shipped, Delivered, Failed)
├── FilterBar (search, status, store, date range)
├── OrdersTable
│   └── OrderRow (expandable)
│       └── OrderItemRow[] (nested)
├── Pagination
└── BulkActionBar (when selected)
```

### Component Tree - Order Detail
```
OrderDetailPage
├── PageHeader (back button, order number, actions)
├── Grid Layout
│   ├── CustomerInfoCard
│   ├── OrderStatusCard (stepper)
│   ├── OrderItemsCard
│   │   └── OrderItemCard[]
│   ├── OrderSummaryCard
│   └── OrderTimeline
```

### Component Tree - Import Wizard
```
ImportPage
├── ImportStepper (1, 2, 3)
└── Step Content
    ├── Step 1: FileDropzone
    ├── Step 2: PreviewTable + ValidationSummary
    └── Step 3: ResultSummary
```

---

## Related Code Files

### Create - Pages
| File Path | Description |
|-----------|-------------|
| `apps/web/src/app/(main)/orders/page.tsx` | Order list với expandable rows |
| `apps/web/src/app/(main)/orders/[id]/page.tsx` | Order detail page |
| `apps/web/src/app/(main)/orders/import/page.tsx` | 3-step import wizard |

### Create - List Components
| File Path | Description |
|-----------|-------------|
| `apps/web/src/features/orders/components/list/orders-table.tsx` | Main table với Collapsible |
| `apps/web/src/features/orders/components/list/order-row.tsx` | Expandable order row |
| `apps/web/src/features/orders/components/list/order-item-row.tsx` | Nested item row |
| `apps/web/src/features/orders/components/list/order-tab-bar.tsx` | Status filter tabs |
| `apps/web/src/features/orders/components/list/orders-filter-bar.tsx` | Search + filters (status, store, date) |
| `apps/web/src/features/orders/components/list/orders-pagination.tsx` | Static pagination component |
| `apps/web/src/features/orders/components/list/index.ts` | Barrel exports |

### Create - Detail Components
| File Path | Description |
|-----------|-------------|
| `apps/web/src/features/orders/components/detail/customer-info-card.tsx` | Customer info |
| `apps/web/src/features/orders/components/detail/order-status-stepper.tsx` | Progress indicator |
| `apps/web/src/features/orders/components/detail/order-items-list.tsx` | Items grid/list |
| `apps/web/src/features/orders/components/detail/order-item-card.tsx` | Single item display |
| `apps/web/src/features/orders/components/detail/order-summary-card.tsx` | Order totals |
| `apps/web/src/features/orders/components/detail/order-timeline.tsx` | Activity history |
| `apps/web/src/features/orders/components/detail/index.ts` | Barrel exports |

### Create - Import Components
| File Path | Description |
|-----------|-------------|
| `apps/web/src/features/orders/components/import/import-stepper.tsx` | Step indicator 1-2-3 |
| `apps/web/src/features/orders/components/import/file-dropzone.tsx` | Drag-drop upload area |
| `apps/web/src/features/orders/components/import/import-preview-table.tsx` | CSV preview |
| `apps/web/src/features/orders/components/import/import-validation-summary.tsx` | Validation errors |
| `apps/web/src/features/orders/components/import/import-result.tsx` | Success summary |
| `apps/web/src/features/orders/components/import/index.ts` | Barrel exports |

### Create - Shared Components
| File Path | Description |
|-----------|-------------|
| `apps/web/src/features/orders/components/order-status-badge.tsx` | Status with color |
| `apps/web/src/features/orders/components/index.ts` | Feature barrel exports |

### Create - Hooks & Utils
| File Path | Description |
|-----------|-------------|
| `apps/web/src/features/orders/lib/constants.ts` | Status configs, step configs |
| `apps/web/src/features/orders/lib/utils.ts` | Helpers (format, calculate) |
| `apps/web/src/features/orders/hooks/use-orders.ts` | Filter/group orders |
| `apps/web/src/features/orders/hooks/index.ts` | Hooks barrel exports |
| `apps/web/src/features/orders/index.ts` | Feature barrel exports |

### Create - Mock Data
| File Path | Description |
|-----------|-------------|
| `apps/web/src/mocks/data/order-timeline.data.ts` | Timeline events cho orders |

### Modify
| File Path | Change Description |
|-----------|-------------------|
| `apps/web/src/mocks/data/order-items.data.ts` | Add designStatus, fulfillerId fields |
| `apps/web/src/mocks/types.ts` | Add MockOrderTimelineEvent interface |
| `apps/web/src/mocks/index.ts` | Export timeline data |
| `apps/web/src/lib/navigation.ts` | Add Orders routes |

### Delete
| File Path | Reason |
|-----------|--------|
| `apps/web/src/features/orders/.gitkeep` | Folder sẽ có files |

---

## Implementation Steps

### 1. Create Order Constants (15 min)
```typescript
// apps/web/src/features/orders/lib/constants.ts
export const ORDER_STATUS_CONFIG = {
  PENDING: { label: 'Chờ xử lý', color: 'gray', step: 0 },
  PROCESSING: { label: 'Đang xử lý', color: 'amber', step: 1 },
  SHIPPED: { label: 'Đã gửi', color: 'blue', step: 2 },
  DELIVERED: { label: 'Đã giao', color: 'emerald', step: 3 },
  FAILED: { label: 'Thất bại', color: 'red', step: -1 },
};

export const ORDER_STEPS = [
  { label: 'Mới', status: 'PENDING' },
  { label: 'Xử lý', status: 'PROCESSING' },
  { label: 'Gửi hàng', status: 'SHIPPED' },
  { label: 'Hoàn thành', status: 'DELIVERED' },
];
```

### 2. Create OrderStatusBadge (20 min)
Badge showing order status with color

### 3. Create OrderTabBar (30 min)
```tsx
<Tabs defaultValue="all">
  <TabsList>
    <TabsTrigger value="all">Tất cả (15)</TabsTrigger>
    <TabsTrigger value="PENDING">Chờ xử lý (3)</TabsTrigger>
    <TabsTrigger value="PROCESSING">Đang xử lý (3)</TabsTrigger>
    ...
  </TabsList>
</Tabs>
```

### 4. Create OrderRow Component (45 min)
Expandable row with Collapsible:
```tsx
<Collapsible>
  <TableRow>
    <TableCell><Checkbox /></TableCell>
    <TableCell>{orderNumber}</TableCell>
    <TableCell>{customerName}</TableCell>
    <TableCell>{items.length} items</TableCell>
    <TableCell><OrderStatusBadge status={status} /></TableCell>
    <TableCell>{relativeTime}</TableCell>
    <CollapsibleTrigger asChild>
      <Button variant="ghost"><ChevronDown /></Button>
    </CollapsibleTrigger>
  </TableRow>
  <CollapsibleContent>
    {items.map(item => <OrderItemRow key={item.id} item={item} />)}
  </CollapsibleContent>
</Collapsible>
```

### 5. Create OrderItemRow (30 min)
Nested row for each order item:
```tsx
<TableRow className="bg-muted/30">
  <TableCell></TableCell>
  <TableCell><Thumbnail /></TableCell>
  <TableCell>{productName} x{quantity}</TableCell>
  <TableCell><DesignStatusBadge /></TableCell>
  <TableCell>{fulfiller || '—'}</TableCell>
  <TableCell><Button size="icon"><Eye /></Button></TableCell>
</TableRow>
```

### 6. Create OrdersTable (45 min)
```tsx
<Table>
  <TableHeader>
    <TableRow>
      <TableHead><Checkbox /></TableHead>
      <TableHead>Mã đơn</TableHead>
      <TableHead>Khách hàng</TableHead>
      <TableHead>Items</TableHead>
      <TableHead>Trạng thái</TableHead>
      <TableHead>Thời gian</TableHead>
    </TableRow>
  </TableHeader>
  <TableBody>
    {orders.map(order => <OrderRow key={order.id} order={order} />)}
  </TableBody>
</Table>
```

### 7. Create OrdersFilterBar Component (30 min)
```tsx
// apps/web/src/features/orders/components/list/orders-filter-bar.tsx
export function OrdersFilterBar({ filters, onChange }) {
  return (
    <div className="flex gap-3 py-3">
      <div className="flex-1">
        <Input
          placeholder="Tìm mã đơn, khách hàng..."
          value={filters.search}
          onChange={(e) => onChange({ ...filters, search: e.target.value })}
          className="max-w-xs"
        />
      </div>
      <Select value={filters.store} onValueChange={(v) => onChange({ ...filters, store: v })}>
        <SelectTrigger className="w-[180px]">
          <SelectValue placeholder="Tất cả store" />
        </SelectTrigger>
        <SelectContent>
          <SelectItem value="all">Tất cả store</SelectItem>
          <SelectItem value="woo">WooCommerce</SelectItem>
          <SelectItem value="shopify">Shopify</SelectItem>
        </SelectContent>
      </Select>
      <DateRangePicker value={filters.dateRange} onChange={(v) => onChange({ ...filters, dateRange: v })} />
    </div>
  );
}
```

### 8. Create OrdersPagination Component (20 min)
```tsx
// apps/web/src/features/orders/components/list/orders-pagination.tsx
interface OrdersPaginationProps {
  currentPage: number;
  totalPages: number;
  onPageChange: (page: number) => void;
}

export function OrdersPagination({ currentPage, totalPages, onPageChange }: OrdersPaginationProps) {
  return (
    <div className="flex items-center justify-between px-2 py-4">
      <p className="text-sm text-muted-foreground">
        Trang {currentPage} / {totalPages}
      </p>
      <div className="flex gap-2">
        <Button
          variant="outline"
          size="sm"
          disabled={currentPage <= 1}
          onClick={() => onPageChange(currentPage - 1)}
        >
          Trước
        </Button>
        <Button
          variant="outline"
          size="sm"
          disabled={currentPage >= totalPages}
          onClick={() => onPageChange(currentPage + 1)}
        >
          Sau
        </Button>
      </div>
    </div>
  );
}
```

### 9. Create Orders List Page (30 min)
```tsx
export default function OrdersPage() {
  const [filters, setFilters] = useState({ search: '', store: 'all', dateRange: null });
  const [currentPage, setCurrentPage] = useState(1);
  const totalPages = 3; // Static for demo

  return (
    <>
      <PageHeader title="Đơn hàng" actions={<><ImportButton /><ExportButton /></>} />
      <OrderTabBar />
      <OrdersFilterBar filters={filters} onChange={setFilters} />
      <OrdersTable orders={mockOrders} />
      <OrdersPagination currentPage={currentPage} totalPages={totalPages} onPageChange={setCurrentPage} />
    </>
  );
}
```

### 10. Create Order Detail Components (1.5 hours)
- `CustomerInfoCard` - Customer name, email, phone, address
- `OrderStatusStepper` - Progress indicator
- `OrderItemCard` - Item with design status, view design button
- `OrderSummaryCard` - Subtotal, shipping, discount, total
- `OrderTimeline` - Event history

### 11. Create Order Detail Page (45 min)
```tsx
export function generateStaticParams() {
  return mockOrders.map(o => ({ id: o.id }));
}

export default function OrderDetailPage({ params }) {
  const order = findOrder(params.id);
  const items = getOrderItems(order.id);

  return (
    <>
      <PageHeader title={order.orderNumber} backHref="/orders" />
      <div className="grid grid-cols-3 gap-6">
        <CustomerInfoCard customer={order} />
        <OrderStatusStepper status={order.status} />
      </div>
      <OrderItemsCard items={items} />
      <div className="grid grid-cols-3 gap-6">
        <OrderSummaryCard order={order} />
        <div className="col-span-2">
          <OrderTimeline orderId={order.id} />
        </div>
      </div>
    </>
  );
}
```

### 12. Create Import Wizard Components (1 hour)
- `ImportStepper` - Step indicator (1, 2, 3)
- `FileDropzone` - Drag & drop area (static)
- `ImportPreviewTable` - Sample preview data
- `ImportResult` - Success summary

### 13. Create Import Page (45 min)
```tsx
export default function ImportPage() {
  const [step, setStep] = useState(1);

  return (
    <>
      <PageHeader title="Import đơn hàng" />
      <ImportStepper currentStep={step} />

      {step === 1 && (
        <FileDropzone onUpload={() => setStep(2)} />
      )}

      {step === 2 && (
        <ImportPreviewTable
          onConfirm={() => setStep(3)}
          onBack={() => setStep(1)}
        />
      )}

      {step === 3 && (
        <ImportResult
          summary={{ orders: 48, tasks: 67, skipped: 2 }}
        />
      )}
    </>
  );
}
```

---

## Todo List

### Setup & Utils
- [ ] Create `apps/web/src/features/orders/lib/constants.ts`
- [ ] Create `apps/web/src/features/orders/lib/utils.ts`

### List Components
- [ ] Create `apps/web/src/features/orders/components/list/order-row.tsx`
- [ ] Create `apps/web/src/features/orders/components/list/order-item-row.tsx`
- [ ] Create `apps/web/src/features/orders/components/list/orders-table.tsx`
- [ ] Create `apps/web/src/features/orders/components/list/order-tab-bar.tsx`
- [ ] Create `apps/web/src/features/orders/components/list/orders-filter-bar.tsx`
- [ ] Create `apps/web/src/features/orders/components/list/orders-pagination.tsx`
- [ ] Create `apps/web/src/features/orders/components/list/index.ts`

### Detail Components
- [ ] Create `apps/web/src/features/orders/components/detail/customer-info-card.tsx`
- [ ] Create `apps/web/src/features/orders/components/detail/order-status-stepper.tsx`
- [ ] Create `apps/web/src/features/orders/components/detail/order-items-list.tsx`
- [ ] Create `apps/web/src/features/orders/components/detail/order-item-card.tsx`
- [ ] Create `apps/web/src/features/orders/components/detail/order-summary-card.tsx`
- [ ] Create `apps/web/src/features/orders/components/detail/order-timeline.tsx`
- [ ] Create `apps/web/src/features/orders/components/detail/index.ts`

### Import Components
- [ ] Create `apps/web/src/features/orders/components/import/import-stepper.tsx`
- [ ] Create `apps/web/src/features/orders/components/import/file-dropzone.tsx`
- [ ] Create `apps/web/src/features/orders/components/import/import-preview-table.tsx`
- [ ] Create `apps/web/src/features/orders/components/import/import-validation-summary.tsx`
- [ ] Create `apps/web/src/features/orders/components/import/import-result.tsx`
- [ ] Create `apps/web/src/features/orders/components/import/index.ts`

### Shared & Exports
- [ ] Create `apps/web/src/features/orders/components/order-status-badge.tsx`
- [ ] Create `apps/web/src/features/orders/components/index.ts`
- [ ] Create `apps/web/src/features/orders/hooks/use-orders.ts`
- [ ] Create `apps/web/src/features/orders/hooks/index.ts`
- [ ] Create `apps/web/src/features/orders/index.ts`

### Mock Data
- [ ] Create `apps/web/src/mocks/data/order-timeline.data.ts`
- [ ] Update `apps/web/src/mocks/data/order-items.data.ts`
- [ ] Update `apps/web/src/mocks/types.ts`

### Pages
- [ ] Create `apps/web/src/app/(main)/orders/page.tsx`
- [ ] Create `apps/web/src/app/(main)/orders/[id]/page.tsx`
- [ ] Create `apps/web/src/app/(main)/orders/import/page.tsx`

### Integration
- [ ] Update `apps/web/src/lib/navigation.ts`

### Testing
- [ ] Test order list với expandable rows
- [ ] Test tab filtering by status
- [ ] Test filter bar (search, store, date) works
- [ ] Test pagination navigation works
- [ ] Test order detail page renders correctly
- [ ] Test status stepper shows correct step
- [ ] Test import wizard step navigation

---

## Success Criteria

- [ ] Order list shows 15 orders
- [ ] Click row expands to show items
- [ ] Tab bar filters by status
- [ ] Filter bar allows search, store, date filtering
- [ ] Pagination shows page navigation
- [ ] Order detail shows customer info, status stepper
- [ ] Items list shows design status
- [ ] Timeline shows mock events
- [ ] Import wizard has 3 steps
- [ ] Step navigation works

---

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Expandable table complex | Medium | Use Collapsible from shadcn |
| Timeline data missing | Low | Generate mock events |
| Import wizard state | Low | Simple useState |

---

## Next Steps

After completion, proceed to [Phase 4: Dashboard + Settings](./phase-04-dashboard-settings.md)
