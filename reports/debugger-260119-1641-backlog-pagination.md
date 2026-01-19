# Backlog Pagination Debug Report

**Date:** 2026-01-19
**Component:** `apps/web/src/app/(main)/design/backlog/page.tsx`
**Issue:** Pagination not working - incorrect page display, total pages not showing, cannot navigate

---

## Root Cause Analysis

**PRIMARY ISSUE: Field Name Mismatch Between API Response and Component Props**

The MSW handler returns `total_pages` (snake_case) but the component expects `totalPages` (camelCase).

### Evidence Chain

1. **MSW Handler** (`apps/web/src/mocks/utils/response-helpers.ts:66-71`)
   ```typescript
   export function getPaginationMeta(total: number, page: number, limit: number): PaginationMeta {
     return {
       page,
       limit,
       total,
       total_pages: Math.ceil(total / limit),  // ← snake_case
     };
   }
   ```

2. **API Client Type** (`apps/web/src/features/design/lib/tasks-client.ts:19-27`)
   ```typescript
   export interface TaskListResponse {
     data: MockTask[];
     pagination: {
       page: number;
       limit: number;
       total: number;
       totalPages: number;  // ← camelCase expected
     };
   }
   ```

3. **Component Usage** (`apps/web/src/app/(main)/design/backlog/page.tsx:223-232`)
   ```typescript
   {pagination && (
     <BacklogPagination
       page={page}
       pageSize={pageSize}
       total={pagination.total}
       totalPages={pagination.totalPages}  // ← undefined at runtime
       onPageChange={handlePageChange}
       onPageSizeChange={handlePageSizeChange}
     />
   )}
   ```

4. **Pagination Component** (`apps/web/src/features/design/components/backlog/backlog-pagination.tsx:41-45`)
   ```typescript
   const hasPages = totalPages > 0;  // ← false when totalPages is undefined
   const canPrevious = hasPages && page > 1;
   const canNext = hasPages && page < totalPages;
   const displayPage = hasPages ? page : 0;  // ← displays 0
   const displayTotalPages = hasPages ? totalPages : 0;  // ← displays 0
   ```

---

## Impact Assessment

**Symptoms:**
- Page displays as "Page 0 of 0" instead of "Page 1 of X"
- All navigation buttons disabled (first, prev, next, last)
- Cannot navigate to any page
- Total count displays correctly (uses `pagination.total`)

**Severity:** High - Core functionality completely broken

**Affected Users:** All users viewing backlog page with tasks

---

## Code Locations with Line Numbers

### 1. MSW Response Helper (ROOT CAUSE)
**File:** `apps/web/src/mocks/utils/response-helpers.ts`
**Lines:** 66-71
**Issue:** Returns `total_pages` instead of `totalPages`

### 2. API Client Type Definition
**File:** `apps/web/src/features/design/lib/tasks-client.ts`
**Lines:** 19-27
**Issue:** Defines interface with `totalPages` (camelCase)

### 3. Backlog Page Component
**File:** `apps/web/src/app/(main)/design/backlog/page.tsx`
**Lines:** 223-232
**Issue:** Receives undefined `totalPages` from API response

### 4. Pagination Component Logic
**File:** `apps/web/src/features/design/components/backlog/backlog-pagination.tsx`
**Lines:** 41-45
**Issue:** Evaluates `totalPages > 0` as false, disables all controls

---

## Data Flow Analysis

```
MSW Handler (tasks.ts:58-61)
  ↓ paginateArray() + getPaginationMeta()
  ↓ Returns { data: [...], pagination: { page, limit, total, total_pages } }
  ↓
API Client (tasks-client.ts:56-64)
  ↓ getTasks() returns TaskListResponse
  ↓ Type expects: { data, pagination: { page, limit, total, totalPages } }
  ↓ Runtime receives: { data, pagination: { page, limit, total, total_pages } }
  ↓
React Query Hook (use-tasks.ts:27-32)
  ↓ useTasks() / useBacklogTasks()
  ↓ Returns response with pagination.totalPages = undefined
  ↓
Backlog Page (page.tsx:36-49)
  ↓ const pagination = taskResponse?.pagination
  ↓ pagination.totalPages is undefined
  ↓
BacklogPagination Component (backlog-pagination.tsx:31-46)
  ↓ totalPages prop = undefined
  ↓ hasPages = false (undefined > 0 = false)
  ↓ All navigation disabled, displays "Page 0 of 0"
```

---

## Recommended Fixes

### Option 1: Update MSW Handler (Preferred)
**Why:** Frontend expects camelCase, align mock to match production API

**File:** `apps/web/src/mocks/utils/response-helpers.ts`

**Change line 70:**
```typescript
// Before
total_pages: Math.ceil(total / limit),

// After
totalPages: Math.ceil(total / limit),
```

**Also update type definition (line 23):**
```typescript
// Before
total_pages: number;

// After
totalPages: number;
```

**Impact:** Minimal - single source change, aligns with frontend convention

---

### Option 2: Update Frontend to Use Snake Case
**Why:** If production API returns snake_case, frontend should adapt

**Files to change:**
1. `apps/web/src/features/design/lib/tasks-client.ts` (line 25)
2. `apps/web/src/app/(main)/design/backlog/page.tsx` (line 228)

**Changes:**
```typescript
// tasks-client.ts interface
totalPages: number;  // → total_pages: number;

// page.tsx component
totalPages={pagination.totalPages}  // → totalPages={pagination.total_pages}
```

**Impact:** Higher - multiple file changes, but matches backend convention

---

### Option 3: Add Runtime Transformation
**Why:** Defensive programming for API inconsistencies

**File:** `apps/web/src/app/(main)/design/backlog/page.tsx`

**Add after line 49:**
```typescript
const tasks = taskResponse?.data ?? [];
const pagination = taskResponse?.pagination;

// Normalize pagination field names
const normalizedPagination = pagination ? {
  ...pagination,
  totalPages: pagination.totalPages ?? (pagination as any).total_pages
} : undefined;
```

**Then use `normalizedPagination` instead of `pagination` (line 228)**

**Impact:** Medium - adds transformation layer, handles both formats

---

## Testing Recommendations

### Unit Tests Needed
1. **MSW Handler Test**
   - Verify pagination response structure
   - Assert `totalPages` field exists (not `total_pages`)
   - Test with various page/limit combinations

2. **Component Test**
   - Test BacklogPagination with valid pagination data
   - Verify navigation buttons enable/disable correctly
   - Test page display text formatting

### Integration Tests Needed
1. **API Response Validation**
   - Verify real API returns expected field names
   - Test pagination across different endpoints
   - Validate type safety between client/server

### Manual Testing Steps
1. Navigate to `/design/backlog`
2. Verify page displays "Page 1 of X" (not "Page 0 of 0")
3. Click next/prev buttons to navigate
4. Change page size dropdown
5. Apply filters (search, urgent, source)
6. Verify pagination resets to page 1 on filter changes

---

## Related Issues to Check

### Potential Similar Issues
1. **Other endpoints using pagination:**
   - Check `workspaces.ts` handler (line 6 imports `getPaginationMeta`)
   - Check `orders.ts` handler (line 14 imports pagination utils)
   - Verify all return `totalPages` not `total_pages`

2. **Type Safety Gap:**
   - TypeScript interface defines `totalPages`
   - Runtime returns `total_pages`
   - No compile-time error because MSW handlers untyped
   - Consider typing MSW handlers with response interfaces

### Prevention Strategies
1. Add runtime validation for API responses
2. Create typed MSW handler wrappers
3. Add E2E tests covering pagination flows
4. Implement API contract testing (schema validation)

---

## Performance Considerations

**Current Performance:** Not applicable - feature completely broken

**After Fix:**
- No performance impact expected
- Pagination logic already optimized
- Server-side filtering implemented correctly

---

## Security Considerations

**None identified** - This is a field name mapping issue with no security implications.

---

## Additional Context

### Why This Wasn't Caught Earlier
1. **Type safety gap:** MSW handlers return untyped responses
2. **No runtime validation:** API responses not validated against types
3. **No E2E tests:** Pagination flow not covered by automated tests
4. **Silent failure:** Component gracefully degrades to "0 of 0" instead of crashing

### Backend API Convention
Need to verify production API response format:
- If backend returns `total_pages` → Use Option 2
- If backend returns `totalPages` → Use Option 1 (recommended)
- If inconsistent → Use Option 3 (safest)

---

## Unresolved Questions

1. **Production API format:** Does real backend return `total_pages` or `totalPages`?
2. **API convention documentation:** Is there a documented standard for field naming?
3. **Other MSW handlers:** Are there similar mismatches in other handlers?
4. **Type validation:** Should we add runtime type validation for all API responses?
5. **Contract testing:** Should we implement API schema validation between frontend/backend?
