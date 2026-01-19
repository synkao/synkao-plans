# Phase 1: Types & Hooks

## Context Links

- Spec: `docs/frontend/synkao-frontend-spec.md` sections 3.1, 3.2
- Existing types: `apps/web/src/mocks/types.ts` (MockOrder, MockOrderItem)
- Pattern reference: `apps/web/src/features/design/lib/tasks-client.ts`
- Pattern reference: `apps/web/src/features/design/hooks/use-tasks.ts`

## Overview

| Field | Value |
|-------|-------|
| Priority | P1 |
| Status | pending |
| Effort | 1h |
| Date | 2026-01-19 |

Create TypeScript types, API client functions, and React Query hooks for orders module.

## Key Insights

- Reuse existing `MockOrder`, `MockOrderItem` types from `mocks/types.ts`
- Follow query key factory pattern from `use-tasks.ts`
- API client pattern from `tasks-client.ts` with fetch + error handling
- Orders API already exists: `GET /api/v1/orders/`, `GET /api/v1/orders/:id`, `GET /api/v1/orders/:id/items`

## Requirements

### Functional
- Export response types for order list (with pagination)
- Export params type for filtering orders
- Hooks for: `useOrders()`, `useOrder(id)`, `useOrderItems(orderId)`
- Query key factory for cache invalidation

### Non-functional
- TypeScript strict mode compliant
- Consistent with existing patterns

## Architecture

```
lib/
├── orders-client.ts    # HTTP functions
└── constants.ts        # Order status config

hooks/
├── use-orders.ts       # React Query hooks

types/
└── order.types.ts      # Re-exports + extended types
```

## Related Code Files

**To Create:**
- `features/orders/types/order.types.ts`
- `features/orders/lib/orders-client.ts`
- `features/orders/lib/constants.ts`
- `features/orders/hooks/use-orders.ts`
- `features/orders/hooks/index.ts`
- `features/orders/lib/index.ts`
- `features/orders/types/index.ts`

**Reference:**
- `mocks/types.ts` - MockOrder, MockOrderItem, OrderStatus, DesignPhaseStatus
- `mocks/handlers/orders.ts` - API structure
- `features/design/hooks/use-tasks.ts` - Query pattern

## Implementation Steps

### 1. Create types file

**File:** `features/orders/types/order.types.ts`

```typescript
// Re-export from mocks for convenience
export type {
  MockOrder,
  MockOrderItem,
  MockAddress,
  OrderStatusType,
  DesignPhaseStatusType,
  ItemDesignStatusType,
} from '@/mocks/types';

export { OrderStatus, DesignPhaseStatus, ItemDesignStatus } from '@/mocks/types';

// Extended types for grouped display
export interface OrderWithItems extends MockOrder {
  items: MockOrderItem[];
}

// API response types
export interface OrderListParams {
  page?: number;
  limit?: number;
  status?: OrderStatusType;
  design_status?: DesignPhaseStatusType;
  fulfiller_name?: string;
  search?: string;
  from?: string;  // ISO date
  to?: string;    // ISO date
}

export interface OrderListResponse {
  data: MockOrder[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

export interface OrderItemsResponse {
  items: MockOrderItem[];
  total: number;
}
```

### 2. Create constants file

**File:** `features/orders/lib/constants.ts`

```typescript
import type { OrderStatusType, DesignPhaseStatusType } from '../types';
import { OrderStatus, DesignPhaseStatus } from '../types';

export interface StatusConfig {
  label: string;
  color: string;
  bgClass: string;
  textClass: string;
}

export const ORDER_STATUS_CONFIG: Record<OrderStatusType, StatusConfig> = {
  PENDING: { label: 'Pending', color: 'gray', bgClass: 'bg-gray-50', textClass: 'text-gray-600' },
  CONFIRMED: { label: 'Confirmed', color: 'blue', bgClass: 'bg-blue-50', textClass: 'text-blue-600' },
  IN_DESIGN: { label: 'In Design', color: 'amber', bgClass: 'bg-amber-50', textClass: 'text-amber-600' },
  READY_TO_FULFILL: { label: 'Ready', color: 'violet', bgClass: 'bg-violet-50', textClass: 'text-violet-600' },
  SHIPPED: { label: 'Shipped', color: 'cyan', bgClass: 'bg-cyan-50', textClass: 'text-cyan-600' },
  COMPLETED: { label: 'Completed', color: 'emerald', bgClass: 'bg-emerald-50', textClass: 'text-emerald-600' },
  CANCELLED: { label: 'Cancelled', color: 'red', bgClass: 'bg-red-50', textClass: 'text-red-600' },
};

export const DESIGN_PHASE_CONFIG: Record<DesignPhaseStatusType, StatusConfig> = {
  NONE: { label: 'N/A', color: 'gray', bgClass: 'bg-gray-50', textClass: 'text-gray-400' },
  PENDING: { label: 'Pending', color: 'gray', bgClass: 'bg-gray-50', textClass: 'text-gray-600' },
  IN_PROGRESS: { label: 'In Progress', color: 'amber', bgClass: 'bg-amber-50', textClass: 'text-amber-600' },
  COMPLETED: { label: 'Completed', color: 'emerald', bgClass: 'bg-emerald-50', textClass: 'text-emerald-600' },
};

export const ITEM_DESIGN_STATUS_CONFIG: Record<string, StatusConfig> = {
  NOT_REQUIRED: { label: '—', color: 'gray', bgClass: 'bg-gray-50', textClass: 'text-gray-400' },
  PENDING: { label: 'Pending', color: 'blue', bgClass: 'bg-blue-50', textClass: 'text-blue-600' },
  IN_PROGRESS: { label: 'In Progress', color: 'amber', bgClass: 'bg-amber-50', textClass: 'text-amber-600' },
  APPROVED: { label: 'Approved', color: 'emerald', bgClass: 'bg-emerald-50', textClass: 'text-emerald-600' },
};

// Order status flow for stepper
export const ORDER_STATUS_STEPS = [
  { status: OrderStatus.PENDING, label: 'New' },
  { status: OrderStatus.IN_DESIGN, label: 'Design' },
  { status: OrderStatus.READY_TO_FULFILL, label: 'Ready' },
  { status: OrderStatus.COMPLETED, label: 'Done' },
];
```

### 3. Create API client

**File:** `features/orders/lib/orders-client.ts`

```typescript
import type {
  MockOrder,
  OrderListParams,
  OrderListResponse,
  OrderItemsResponse,
} from '../types';

const API_BASE = '/api/v1';

function buildQueryString(params: OrderListParams): string {
  const searchParams = new URLSearchParams();

  if (params.page) searchParams.set('page', String(params.page));
  if (params.limit) searchParams.set('limit', String(params.limit));
  if (params.status) searchParams.set('status', params.status);
  if (params.design_status) searchParams.set('design_status', params.design_status);
  if (params.fulfiller_name) searchParams.set('fulfiller_name', params.fulfiller_name);
  if (params.search) searchParams.set('search', params.search);
  if (params.from) searchParams.set('from', params.from);
  if (params.to) searchParams.set('to', params.to);

  const qs = searchParams.toString();
  return qs ? `?${qs}` : '';
}

// GET /api/v1/orders/
export async function getOrders(params: OrderListParams = {}): Promise<OrderListResponse> {
  const response = await fetch(`${API_BASE}/orders/${buildQueryString(params)}`);

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Failed to fetch orders');
  }

  return response.json();
}

// GET /api/v1/orders/:id
export async function getOrder(id: string): Promise<MockOrder> {
  const response = await fetch(`${API_BASE}/orders/${id}`);

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Failed to fetch order');
  }

  const result = await response.json();
  return result.data;
}

// GET /api/v1/orders/:id/items
export async function getOrderItems(orderId: string): Promise<OrderItemsResponse> {
  const response = await fetch(`${API_BASE}/orders/${orderId}/items`);

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Failed to fetch order items');
  }

  const result = await response.json();
  return result.data;
}
```

### 4. Create React Query hooks

**File:** `features/orders/hooks/use-orders.ts`

```typescript
import { useQuery, useQueries } from '@tanstack/react-query';
import {
  getOrders,
  getOrder,
  getOrderItems,
} from '../lib/orders-client';
import type { OrderListParams, OrderWithItems } from '../types';

// Query Key Factory
export const orderKeys = {
  all: ['orders'] as const,
  lists: () => [...orderKeys.all, 'list'] as const,
  list: (filters: OrderListParams) => [...orderKeys.lists(), filters] as const,
  details: () => [...orderKeys.all, 'detail'] as const,
  detail: (id: string) => [...orderKeys.details(), id] as const,
  items: (orderId: string) => [...orderKeys.all, 'items', orderId] as const,
};

/**
 * Hook to fetch orders with filters and pagination
 */
export function useOrders(params: OrderListParams = {}) {
  return useQuery({
    queryKey: orderKeys.list(params),
    queryFn: () => getOrders(params),
    staleTime: 1000 * 60 * 2, // 2 minutes
  });
}

/**
 * Hook to fetch single order by ID
 */
export function useOrder(id: string) {
  return useQuery({
    queryKey: orderKeys.detail(id),
    queryFn: () => getOrder(id),
    enabled: !!id,
  });
}

/**
 * Hook to fetch order items
 */
export function useOrderItems(orderId: string) {
  return useQuery({
    queryKey: orderKeys.items(orderId),
    queryFn: () => getOrderItems(orderId),
    enabled: !!orderId,
  });
}

/**
 * Hook to fetch orders with their items (for grouped display)
 */
export function useOrdersWithItems(params: OrderListParams = {}) {
  const ordersQuery = useOrders(params);
  const orders = ordersQuery.data?.data ?? [];

  const itemsQueries = useQueries({
    queries: orders.map((order) => ({
      queryKey: orderKeys.items(order.id),
      queryFn: () => getOrderItems(order.id),
      enabled: ordersQuery.isSuccess,
      staleTime: 1000 * 60 * 2,
    })),
  });

  const isLoading = ordersQuery.isLoading || itemsQueries.some((q) => q.isLoading);
  const error = ordersQuery.error || itemsQueries.find((q) => q.error)?.error;

  const ordersWithItems: OrderWithItems[] = orders.map((order, idx) => ({
    ...order,
    items: itemsQueries[idx]?.data?.items ?? [],
  }));

  return {
    data: ordersWithItems,
    pagination: ordersQuery.data?.pagination,
    isLoading,
    error,
  };
}
```

### 5. Create index files

**File:** `features/orders/types/index.ts`
```typescript
export * from './order.types';
```

**File:** `features/orders/lib/index.ts`
```typescript
export * from './orders-client';
export * from './constants';
```

**File:** `features/orders/hooks/index.ts`
```typescript
export * from './use-orders';
```

## Todo List

- [ ] Create `features/orders/types/order.types.ts`
- [ ] Create `features/orders/lib/constants.ts`
- [ ] Create `features/orders/lib/orders-client.ts`
- [ ] Create `features/orders/hooks/use-orders.ts`
- [ ] Create index files for types, lib, hooks
- [ ] Verify TypeScript compilation
- [ ] Test hooks with MSW in browser

## Success Criteria

- All types export without errors
- `useOrders()` returns paginated order list
- `useOrder(id)` returns single order
- `useOrderItems(orderId)` returns items array
- `useOrdersWithItems()` returns orders with nested items

## Risk Assessment

| Risk | Mitigation |
|------|------------|
| MSW response format mismatch | Verify against `mocks/handlers/orders.ts` |
| React Query version differences | Follow existing patterns in `use-tasks.ts` |

## Security Considerations

- No auth handling in client (handled by MSW/backend)
- No sensitive data exposed in types

## Next Steps

Proceed to Phase 2: Shared Components (OrderStatusBadge, DesignPhaseBadge, FulfillerCell)
