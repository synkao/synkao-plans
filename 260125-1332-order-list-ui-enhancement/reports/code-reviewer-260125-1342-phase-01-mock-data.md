# Code Review Report: Phase 1 - Mock Data Preparation

**Date**: 2026-01-25
**Reviewer**: code-reviewer
**Score**: 8.5/10

## Scope

**Files Reviewed**:
- `apps/web/src/mocks/types.ts` (Modified)
- `apps/web/src/mocks/data/stores.data.ts` (Created)
- `apps/web/src/mocks/data/index.ts` (Modified)
- `apps/web/src/mocks/data/orders.data.ts` (Modified)
- `apps/web/src/mocks/handlers/orders.ts` (Modified)

**Lines of Code**: ~150 added, 5 modified
**Review Focus**: Recent changes for mock data enhancement
**Build Status**: ✅ ESLint passed (no errors)

## Overall Assessment

Implementation successfully adds 3 new optional fields to `MockOrder` type and populates them with realistic data. Code follows YAGNI/KISS/DRY principles with helper functions for data generation. Type safety maintained throughout. Mock data distribution realistic (50% null values for notes, varied fulfillers).

Clean separation of concerns: types → data → handlers. Good documentation with JSDoc comments and inline explanations.

## Critical Issues

None.

## High Priority Findings

### H1: Revenue Calculation Business Logic Should Be Configurable

**Location**: `apps/web/src/mocks/data/orders.data.ts:151-153`

```typescript
const calculateRevenue = (totalAmount: number): number => {
  return Math.round(totalAmount * 0.7);
};
```

**Issue**: Hardcoded 30% cost margin (70% revenue) may not match actual business model.

**Impact**: If real POD cost margin differs significantly, mock data won't represent production scenarios accurately, leading to misleading UI testing.

**Recommendation**: Add constant or comment explaining assumption:
```typescript
// POD industry standard: ~30% production cost, ~70% revenue
const POD_COST_MARGIN = 0.3;
const calculateRevenue = (totalAmount: number): number => {
  return Math.round(totalAmount * (1 - POD_COST_MARGIN));
};
```

### H2: Fulfill Deadline Logic Too Simplistic

**Location**: `apps/web/src/mocks/data/orders.data.ts:158-162`

```typescript
const getFulfillDeadline = (createdAt: string): string => {
  const date = new Date(createdAt);
  date.setDate(date.getDate() + 3);
  return date.toISOString();
};
```

**Issue**:
- Fixed 3-day deadline doesn't account for weekends/holidays
- Applies deadline to ALL orders regardless of status (even CANCELLED)
- No business day calculation

**Impact**: Unrealistic deadline data may hide edge cases in production (weekend deadlines, holiday handling).

**Recommendation**:
```typescript
/**
 * Calculate fulfill deadline (3 business days from order creation)
 * Only applicable for orders requiring fulfillment
 */
const getFulfillDeadline = (createdAt: string, status: OrderStatusType): string | undefined => {
  const needsDeadline = [
    OrderStatus.CONFIRMED,
    OrderStatus.IN_DESIGN,
    OrderStatus.READY_TO_FULFILL,
  ];
  if (!needsDeadline.includes(status)) return undefined;

  const date = new Date(createdAt);
  date.setDate(date.getDate() + 3); // TODO: Add business day calculation
  return date.toISOString();
};
```

Then update usage:
```typescript
fulfillDeadline: getFulfillDeadline(createdAtDate, status),
```

## Medium Priority Improvements

### M1: Missing Type Safety for Store Distribution

**Location**: `apps/web/src/mocks/data/orders.data.ts:179`

```typescript
const store = mockStores[i % mockStores.length]!;
```

**Issue**: Non-null assertion (`!`) bypasses TypeScript safety. If `mockStores` empty, runtime error.

**Recommendation**: Add guard or default:
```typescript
const store = mockStores[i % mockStores.length] ?? {
  id: 'store-default',
  name: 'Default Store',
  platform: Platform.MANUAL
};
```

### M2: Tracking Number Pattern Could Cause Collisions

**Location**: `apps/web/src/mocks/data/orders.data.ts:144`

```typescript
const trackingNumber = `${trackingPatterns[carrier]}${index}`;
```

**Issue**: Appending index to base pattern creates predictable sequences. For 20 orders rotating through 4 carriers, tracking numbers like `9400111899223384769<0-4>` may not validate if UI implements carrier-specific validation.

**Impact**: Low - only affects mock data realism.

**Recommendation**: Use more realistic pattern or note limitation:
```typescript
// NOTE: Simplified tracking numbers for mock data
// Real carriers have checksum validation (USPS Mod10, UPS check digit)
const trackingNumber = `${trackingPatterns[carrier]}${String(index).padStart(3, '0')}`;
```

### M3: Inconsistent Comment Style

**Location**: Throughout files

Mix of JSDoc (`/** */`) and inline (`//`) comments for similar purposes.

**Recommendation**: Use JSDoc for exported functions/interfaces, inline for internal logic.

## Low Priority Suggestions

### L1: Consider Adding Store Lookup Optimization

**Location**: `apps/web/src/mocks/data/stores.data.ts:24`

```typescript
export const findStore = (id: string) => mockStores.find((s) => s.id === id);
```

**Note**: Linear search O(n) acceptable for 3 stores. If store count grows, consider Map.

### L2: Sample Notes Distribution Could Be More Varied

50% null notes realistic, but remaining 50% rotate through only 5 messages. Consider adding more variety or weighted distribution (e.g., "Rush order" less common than "Gift order").

### L3: Type Annotation for `counts` Object

**Location**: `apps/web/src/mocks/handlers/orders.ts:155`

```typescript
const counts: Record<string, number> = {};
```

Could be more specific:
```typescript
const counts: Record<OrderStatusType, number> = {} as Record<OrderStatusType, number>;
```

## Positive Observations

✅ **Clean separation of concerns**: Types, data, handlers well-organized
✅ **Good documentation**: JSDoc comments explain business logic
✅ **Realistic data distribution**: Varied null values, status-based logic
✅ **Type safety**: All new fields properly typed as optional
✅ **YAGNI compliance**: No over-engineering, focused implementation
✅ **DRY principle**: Helper functions eliminate duplication
✅ **Backward compatibility**: All new fields optional, no breaking changes
✅ **Consistent naming**: camelCase, descriptive names
✅ **ESLint compliance**: No linting errors

## Recommended Actions

**Priority Order**:

1. **Add business logic constants** (H1) - 5 min
   Extract magic number `0.7` to named constant with comment

2. **Fix fulfill deadline logic** (H2) - 15 min
   Only set deadline for relevant statuses, add status parameter

3. **Remove non-null assertion** (M1) - 5 min
   Add fallback store or empty check

4. **Document tracking pattern limitation** (M2) - 2 min
   Add comment about simplified mock tracking

5. **Standardize comments** (M3) - 10 min
   Convert exported function comments to JSDoc

**Total Effort**: ~40 minutes

## Task Completeness Verification

Checking against plan file TODO list:

- ✅ 1.1 Add storeName, revenue, fulfillDeadline to MockOrder type
- ✅ 1.2 Create stores.data.ts with mock stores
- ✅ 1.3 Export stores from data/index.ts
- ✅ 1.4 Update orders.data.ts with new helper functions
- ✅ 1.5 Populate all new fields in mockOrders array
- ✅ 1.6 Add /api/v1/orders/counts endpoint
- ⚠️ 1.7 Verify TypeScript compiles without errors (Build failed - Node.js version issue, not code issue)

**Status**: 6/7 tasks complete. Task 1.7 blocked by environment (Node 18 vs required >=20.9).

## Security Considerations

✅ No security issues identified
✅ No sensitive data in mock files
✅ No SQL injection vectors (MSW handlers)
✅ No XSS vulnerabilities

## Performance Analysis

✅ No performance issues
✅ Array operations O(n) acceptable for 20 mock orders
✅ No memory leaks
✅ Handlers synchronous, appropriate for MSW

## Metrics

- **Type Coverage**: 100% (all fields typed)
- **Test Coverage**: N/A (mock data)
- **Linting Issues**: 0
- **Build Status**: ⚠️ Blocked by Node.js version (env issue, not code)

## Next Steps

1. Address H1 & H2 (high priority) before Phase 2
2. Upgrade Node.js to >=20.9.0 or update Next.js config
3. Proceed to Phase 2: UI Implementation
4. Consider adding E2E test for counts endpoint

## Unresolved Questions

1. **What is actual POD cost margin?** (Currently assumes 30%)
2. **Should fulfill deadline account for business days?** (Currently calendar days)
3. **Do carriers have specific tracking format validation?** (Mock uses simplified patterns)
4. **Are there plans to increase mock store count?** (Affects findStore optimization)
