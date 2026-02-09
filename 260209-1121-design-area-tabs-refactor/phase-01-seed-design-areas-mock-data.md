# Phase 01: Seed All Order Items with designAreas

## Context
- Parent plan: [plan.md](plan.md)
- File: `apps/web/src/mocks/data/order-items.data.ts`

## Overview
- Priority: P2
- Status: ✅ completed
- Description: Every MockOrderItem must have `designAreas` populated so DesignAreaTabs renders for all items

## Key Insights
- Currently only items[1] and items[2] have designAreas (lines 144-160)
- Other items have `designAreas: undefined` → DesignAreaTabs returns null
- Need product-type-based position assignment (T_SHIRT: FRONT/BACK, MUG: FRONT, etc.)
- Mix of null (awaiting upload) and DesignFileInfo (already uploaded) for realistic demo

## Requirements
- Every item gets `designAreas` based on product type
- Realistic position mapping per product type
- Some positions have files (DesignFileInfo), some null (awaiting upload)
- Reuse existing `getFileSize()` and `daysAgo()` helpers

## Architecture

### Position mapping by product type
```typescript
const designPositionsByType: Record<MockOrderItem['productType'], DesignPositionType[]> = {
  T_SHIRT: ['FRONT', 'BACK', 'LEFT_CHEST'],
  MUG: ['FRONT', 'BACK'],
  CANVAS: ['FRONT'],
  HOODIE: ['FRONT', 'BACK', 'HOOD', 'POCKET'],
  POSTER: ['FRONT'],
};
```

### File assignment logic
- Items with `designStatus === 'APPROVED'` → all positions have DesignFileInfo
- Items with `designStatus === 'IN_PROGRESS'` → first position has file, rest null
- Items with `designStatus === 'PENDING'` → all positions null (awaiting upload)
- Items with `designStatus === 'NOT_REQUIRED'` → minimal positions, all null

## Related Code Files
- **Modify:** `apps/web/src/mocks/data/order-items.data.ts`

## Implementation Steps
1. Add `designPositionsByType` mapping constant
2. Import `DesignPositionType` from types
3. In the item generation loop, assign `designAreas` based on `productType` and `designStatus`
4. Remove the manual demo assignments at lines 141-160
5. Verify all 60 items get designAreas

## Todo
- [x] Add position-by-type mapping
- [x] Generate designAreas in loop
- [x] Remove manual demo data (lines 141-160)
- [x] Verify mock data renders correctly

## Success Criteria
- Every `mockOrderItems[i].designAreas` is defined (not undefined)
- Positions match product type logically
- Mix of file/null values for realistic demo
- No TypeScript errors

## Risk Assessment
- Low risk: only mock data changes, no production impact
