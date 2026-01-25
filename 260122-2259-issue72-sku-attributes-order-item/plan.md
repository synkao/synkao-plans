# Plan: Issue #72 - SKU & Attributes in Order Item Row

**Issue:** https://github.com/synkao/synkao/issues/72
**Status:** Ready to implement
**Approach:** Two-line Layout

## Overview

Add SKU code and product attributes (Color, Size, etc.) to order item rows in the sales order list.

## Current State

```
[IMG] Classic Black Tee                                    x2
```

## Target State (Two-line Layout)

```
[IMG] Classic Black Tee                                    x2
      SKU: ITEM-001 | Black | L | Unisex
```

## Data Availability

Mock API already provides all required fields in `MockOrderItem`:
- `itemCode` → SKU code ✅
- `attributes[]` → Color, Size, Gender ✅

**No API changes needed.**

## Implementation

### File to Modify

`apps/web/src/features/orders/components/order-list/order-item-row.tsx`

### Changes Required

Update lines 43-66 (Product Name cell):

**Before:**
```tsx
{/* Thumbnail + Product Name (spans 2 columns) */}
<td className="px-4 py-2" colSpan={2}>
  <div className="flex items-center gap-3">
    {/* Thumbnail */}
    <div className="h-10 w-10 ...">...</div>
    {/* Product Name */}
    <span className="text-sm text-gray-700">{item.productName}</span>
  </div>
</td>
```

**After:**
```tsx
{/* Thumbnail + Product Info (spans 2 columns) */}
<td className="px-4 py-2" colSpan={2}>
  <div className="flex items-start gap-3">
    {/* Thumbnail */}
    <div className="h-10 w-10 ...">...</div>
    {/* Product Info - Two lines */}
    <div className="flex flex-col min-w-0">
      {/* Line 1: Product Name */}
      <span className="text-sm text-gray-700 truncate">{item.productName}</span>
      {/* Line 2: SKU + Attributes */}
      <span className="text-xs text-gray-400 truncate">
        SKU: {item.itemCode}
        {item.attributes?.map((attr) => ` | ${attr.value}`).join('')}
      </span>
    </div>
  </div>
</td>
```

### Key Changes

1. Change `items-center` → `items-start` (align top for two-line)
2. Wrap product info in `flex-col` container
3. Add second line for SKU + attributes
4. Add `truncate` + `min-w-0` for overflow handling
5. Style: `text-xs text-gray-400` for secondary info

## Acceptance Criteria

- [x] SKU code displayed for each order item
- [x] Product attributes visible in row
- [x] Clearly formatted, no UI clutter
- [x] Works on all order items

## Testing

1. Navigate to `/orders`
2. Expand any order row
3. Verify each item shows:
   - Product name (line 1)
   - SKU + attributes (line 2, smaller gray text)
4. Test with items having different attribute counts

## Effort

~15 min - Single file, straightforward UI change.
