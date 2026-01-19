# Code Review: Order List & Detail Implementation

**Date:** 2026-01-19
**Reviewer:** code-reviewer agent
**Scope:** Order List & Detail feature implementation
**Score:** 7.5/10

---

## Code Review Summary

### Scope
- Files reviewed: 21 files (types, hooks, components, pages, MSW handlers)
- Lines of code analyzed: ~1,416 lines
- Review focus: Full feature implementation
- Updated plans: plan.md

### Overall Assessment

Good implementation following established patterns. Clean component composition, proper TypeScript usage, functional React Query integration.

**Main concerns:** Performance issue with N+1 queries, accessibility gaps, missing error boundaries, no loading skeletons.

---

## Critical Issues (MUST FIX)

### 1. **N+1 Query Problem in useOrdersWithItems Hook**

**File:** `hooks/use-orders.ts:56-84`

**Issue:** Hook triggers separate API call for EACH order's items, causing cascade of requests.

```typescript
// Current: If 10 orders → 1 orders query + 10 items queries = 11 requests
const itemsQueries = useQueries({
  queries: orders.map((order) => ({
    queryKey: orderKeys.items(order.id),
    queryFn: () => getOrderItems(order.id),
    enabled: ordersQuery.isSuccess,
  })),
});
```

**Impact:**
- Page 1 with 10 orders = 11 HTTP requests
- Slow on high-latency networks
- React Query waterfall effect

**Fix Options:**

**Option A (Recommended):** Backend batch endpoint
```typescript
// orders-client.ts
export async function getOrdersWithItems(params: OrderListParams) {
  const response = await fetch(`${API_BASE}/orders/?include=items&${buildQueryString(params)}`);
  // Returns orders with items nested
}
```

**Option B:** Client-side batch fetcher
```typescript
// Batch all item requests into single call
export async function getBatchOrderItems(orderIds: string[]) {
  const response = await fetch(`${API_BASE}/orders/items/batch?ids=${orderIds.join(',')}`);
  return response.json();
}
```

**Temporary workaround:** Increase staleTime, reduce page size

---

### 2. **Missing Keyboard Navigation**

**File:** `components/order-list/order-list-table.tsx:22-79`

**Issue:** Order rows clickable but no keyboard support

```typescript
<tr onClick={() => onClick?.(order)}>  // No onKeyDown
```

**Impact:** Keyboard users cannot navigate to order detail

**Fix:**
```typescript
<tr
  role="button"
  tabIndex={0}
  onClick={() => onClick?.(order)}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      onClick?.(order);
    }
  }}
>
```

---

### 3. **Unsafe Type Assertion in useOrdersWithItems**

**File:** `hooks/use-orders.ts:73-76`

**Issue:** Assumes parallel array indices match

```typescript
const ordersWithItems: OrderWithItems[] = orders.map((order, idx) => ({
  ...order,
  items: itemsQueries[idx]?.data?.items ?? [],  // idx could mismatch
}));
```

**Impact:** If queries reorder, items assigned to wrong order

**Fix:**
```typescript
const ordersWithItems: OrderWithItems[] = orders.map((order) => {
  const itemQuery = itemsQueries.find(q =>
    q.data && orderKeys.items(order.id).toString() === q.queryKey.toString()
  );
  return {
    ...order,
    items: itemQuery?.data?.items ?? [],
  };
});
```

---

## High Priority Findings

### 4. **No Error Boundaries**

**Files:** `app/(main)/orders/page.tsx`, `app/(main)/orders/[id]/page.tsx`

**Issue:** Unhandled component errors crash entire page

**Fix:** Wrap routes with error boundary
```typescript
// app/(main)/orders/error.tsx
'use client';
export default function OrdersError({ error, reset }) {
  return <ErrorDisplay error={error} onRetry={reset} />;
}
```

---

### 5. **Missing Loading Skeletons**

**File:** `app/(main)/orders/page.tsx:117-126`

**Issue:** Plain text "Loading orders..." poor UX

**Fix:** Add skeleton component
```typescript
if (isLoading) {
  return (
    <>
      <OrdersPageHeader />
      <OrderListSkeleton rows={10} />
    </>
  );
}
```

---

### 6. **Checkbox Accessibility Issues**

**File:** `components/order-list/order-list-table.tsx:58-64`

**Issue:** "Select all" checkbox has indeterminate state not set

**Fix:**
```typescript
<input
  type="checkbox"
  ref={checkboxRef}
  checked={selectedIds.size === orders.length && orders.length > 0}
  onChange={handleSelectAll}
  className="h-4 w-4 rounded border-gray-300"
  aria-label="Select all orders"
/>

// In useEffect
useEffect(() => {
  if (checkboxRef.current) {
    const isIndeterminate = selectedIds.size > 0 && selectedIds.size < orders.length;
    checkboxRef.current.indeterminate = isIndeterminate;
  }
}, [selectedIds, orders]);
```

---

### 7. **Image Optimization Disabled**

**File:** `components/order-list/order-item-row.tsx:48-56`

**Issue:** `unoptimized` prop bypasses Next.js optimization

```typescript
<Image
  src={item.productImageUrl}
  alt={item.productName}
  fill
  unoptimized  // ← BAD
/>
```

**Impact:** Larger bundle, slower page loads

**Fix:** Remove `unoptimized`, configure image domains in `next.config.js`

---

### 8. **React Query Cache Invalidation Missing**

**File:** `hooks/use-orders.ts`

**Issue:** No mutation hooks for order updates, no cache invalidation strategy

**Needed:**
```typescript
export function useUpdateOrderStatus() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: { orderId: string; status: OrderStatusType }) =>
      updateOrderStatus(data.orderId, data.status),
    onSuccess: (_, variables) => {
      queryClient.invalidateQueries({ queryKey: orderKeys.detail(variables.orderId) });
      queryClient.invalidateQueries({ queryKey: orderKeys.lists() });
    },
  });
}
```

---

### 9. **Timeline Generation in Page Component**

**File:** `app/(main)/orders/[id]/page.tsx:174-239`

**Issue:** 66-line timeline generator in page component, violates SRP

**Fix:** Extract to `lib/timeline-generator.ts`

---

### 10. **Memory Leak Risk: Uncontrolled State Growth**

**File:** `app/(main)/orders/page.tsx:35`

**Issue:** selectedOrderIds Set persists across pagination without cleanup

**Fix:**
```typescript
// Clear on unmount
useEffect(() => {
  return () => setSelectedOrderIds(new Set());
}, []);
```

---

## Medium Priority Improvements

### 11. Pagination Component Duplication
**File:** `app/(main)/orders/page.tsx:216-260`
Pagination component defined inline. Extract to `components/shared/pagination.tsx`

### 12. Hardcoded API Base URL
**File:** `lib/orders-client.ts:9`
`const API_BASE = '/api/v1'` should use env var

### 13. Magic Numbers
**File:** `app/(main)/orders/page.tsx:27`
`const pageSize = 10` should be constant `DEFAULT_PAGE_SIZE`

### 14. Missing ARIA Live Regions
**File:** `components/order-list/order-list-table.tsx:99-103`
"No orders found" should have `aria-live="polite"`

### 15. Insufficient Error Messages
**File:** `lib/orders-client.ts:38`
Generic "Failed to fetch orders" - include status code

---

## Low Priority Suggestions

### 16. Unused Props in Components
- `BulkActionBar.itemCount` - calculated but optional usage
- Consider making required if always calculated

### 17. Inconsistent Size Props
- Some badges default to 'md', ItemDesignStatusBadge defaults to 'sm'
- Standardize default sizing

### 18. CSS Class Duplication
- Card backdrop blur pattern repeated 7+ times
- Extract to Tailwind @apply or component

### 19. Date Formatting Inconsistency
- `formatRelativeTime` vs inline `toLocaleDateString`
- Centralize date formatting utilities

### 20. No Empty State Illustrations
- Table shows plain text "No orders found"
- Add empty state SVG/illustration

---

## Positive Observations

**Excellent:**
1. ✅ Type safety - strict TypeScript, no `any` types
2. ✅ Component composition - clean separation of concerns
3. ✅ Consistent naming - kebab-case files, descriptive names
4. ✅ Query key factory pattern - proper cache management structure
5. ✅ Error handling - try-catch in API clients
6. ✅ Accessibility basics - aria-labels on interactive elements
7. ✅ No XSS vulnerabilities - no dangerouslySetInnerHTML
8. ✅ Clean imports - organized, no circular deps
9. ✅ Proper React hooks - no ESLint rule violations
10. ✅ MSW integration - good mock data structure

---

## Recommended Actions

### Immediate (Before Merge)
1. **Fix N+1 query problem** - Implement batch endpoint or temporary mitigation
2. **Add keyboard navigation** - OrderRow onKeyDown handlers
3. **Fix type safety** - useOrdersWithItems index mapping
4. **Add error boundaries** - Prevent full page crashes

### Before Production
5. **Add loading skeletons** - Better perceived performance
6. **Implement mutations** - Cache invalidation for updates
7. **Extract timeline generator** - Move to lib/
8. **Fix image optimization** - Remove unoptimized prop
9. **Add indeterminate checkboxes** - Proper selection UI
10. **Memory leak cleanup** - useEffect cleanup functions

### Tech Debt
11. Extract pagination component
12. Centralize constants
13. Improve error messages
14. Add empty states
15. Refactor duplicate CSS

---

## Metrics

- **Type Coverage:** 100% (all files TypeScript)
- **Test Coverage:** 0% (no tests found)
- **Linting Issues:** 0 (tsc passed without errors)
- **Security Issues:** 0 (no XSS, injection vulnerabilities)
- **Performance Issues:** 1 critical (N+1 queries)
- **Accessibility Issues:** 3 (keyboard nav, indeterminate, live regions)

---

## Plan Status Update

**Phase Completion:**
- ✅ Phase 1: Types & Hooks - Complete
- ✅ Phase 2: Shared Components - Complete
- ✅ Phase 3: Order List Components - Complete
- ✅ Phase 4: Order List Page - Complete
- ✅ Phase 5: Order Detail Components - Complete
- ✅ Phase 6: Order Detail Page - Complete
- ✅ Phase 7: MSW Mock Enhancement - Complete

**Success Criteria:**
- ✅ Order List displays grouped rows
- ✅ Checkbox selection works at order level
- ✅ Filters work: search, status, design, fulfiller
- ✅ Bulk action bar appears when orders selected
- ✅ Server-side pagination functional
- ✅ Order Detail shows customer info, status stepper, items, timeline
- ✅ Navigation between list and detail works

**Outstanding:**
- ⚠️ Performance optimization needed (N+1 queries)
- ⚠️ Accessibility improvements required
- ⚠️ Error boundaries missing
- ⚠️ Tests not implemented

---

## Unresolved Questions

1. **Timeline API:** Should timeline be server-generated or client-generated? Current implementation has both (MSW endpoint exists but unused, page generates locally)

2. **Batch Items Endpoint:** Does backend support `?include=items` or `/orders/items/batch`? Need backend team confirmation for N+1 fix

3. **Image CDN:** What domains should be added to Next.js image config for optimization?

4. **Page Size:** Is 10 orders/page optimal? Should be configurable by user?

5. **Mutation Strategy:** What order operations need mutations? (status update, assign designer, create fulfillment)

6. **Test Coverage:** What testing strategy - unit, integration, or E2E priority?
