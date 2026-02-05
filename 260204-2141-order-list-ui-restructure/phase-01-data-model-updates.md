# Phase 01: Data Model Updates

**Parent:** [plan.md](./plan.md)
**Dependencies:** None

## Overview

| Field | Value |
|-------|-------|
| Priority | P1 |
| Status | ✅ completed |
| Effort | 1h |

Add `domain` field to MockStore for platform icon + domain display.

## Key Insights

- Current `MockStore` only has: `id`, `name`, `platform`
- Need `domain` for display like `[W] mystore.com`
- `externalId` already exists in MockOrder - no changes needed

## Requirements

### Functional
- Add `domain` field to MockStore interface
- Update mock stores with domain values
- Ensure backward compatibility

### Non-functional
- No API changes required (mock data only)

## Related Code Files

| File | Action |
|------|--------|
| `apps/web/src/mocks/data/stores.data.ts` | Modify |

## Implementation Steps

1. Update `MockStore` interface to add `domain: string` field
2. Update `mockStores` array with domain values:
   - `store-woo-main`: `mystore.com`
   - `store-shopify-us`: `shopify-us.com`
   - `store-manual`: `Manual Import` (special case)

## Todo List

- [x] Add `domain` field to MockStore interface
- [x] Update mockStores with domain values
- [x] Verify no TypeScript errors

## Success Criteria

- MockStore has `domain` field
- All existing stores have valid domain values
- No compilation errors

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Breaking existing code | Low | Low | Interface extension is additive |

## Next Steps

→ Phase 02: Use domain in OrderRow component
