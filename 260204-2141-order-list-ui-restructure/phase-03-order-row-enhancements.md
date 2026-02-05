# Phase 03: Order Row Enhancements

**Parent:** [plan.md](./plan.md)
**Dependencies:** Phase 01 (Store domain)

## Overview

| Field | Value |
|-------|-------|
| Priority | P2 |
| Status | ✅ completed |
| Effort | 2h |

Add external ID display and platform icon + domain in order row.

## Key Insights

- `externalId` exists in MockOrder (e.g., "WC-1001")
- Need to format as `#WC-xxxxx` next to order number
- Platform icon needs color + letter mapping
- Domain comes from MockStore (added in Phase 01)

## Requirements

### Functional

**External ID:**
- Display format: `ORD-001 #WC-78542`
- External ID in smaller, muted text
- Prefix based on platform: `#WC-`, `#SP-`, `#M-`

**Platform Icon + Domain:**
- Replace text `storeName` with icon + domain
- Icon specs:
  | Platform | Letter | Color |
  |----------|--------|-------|
  | WooCommerce | W | #96588a (purple) |
  | Shopify | S | #96bf48 (green) |
  | Manual | ↓ | #64748b (gray) |
- Size: 20x20px, border-radius: 4px

### Non-functional
- Icons should be crisp at all zoom levels
- Colors must meet contrast requirements

## Architecture

```
Before:
ORD-001
WooCommerce Main

After:
ORD-001  #WC-78542
[W] mystore.com
```

**Platform Icon Component:**
```typescript
interface PlatformIconProps {
  platform: PlatformType;
  className?: string;
}

const PLATFORM_CONFIG = {
  WOOCOMMERCE: { letter: 'W', bgColor: '#96588a' },
  SHOPIFY: { letter: 'S', bgColor: '#96bf48' },
  MANUAL: { letter: '↓', bgColor: '#64748b' },
};
```

## Related Code Files

| File | Action |
|------|--------|
| `apps/web/src/features/orders/components/order-list/order-row.tsx` | Modify |
| `apps/web/src/features/orders/components/order-list/platform-icon.tsx` | Create |
| `apps/web/src/features/orders/lib/constants.ts` | Modify (add platform config) |

## Implementation Steps

1. **Create PlatformIcon component:**
   - New file: `platform-icon.tsx`
   - Props: `platform: PlatformType`
   - Render 20x20 div with letter/icon and bg color

2. **Add platform config to constants:**
   - Add `PLATFORM_CONFIG` with letter and color mappings

3. **Update OrderRow:**
   - Add external ID next to order number: `{order.orderNumber} #{order.externalId}`
   - Replace storeName badge with: `<PlatformIcon platform={store.platform} /> {store.domain}`
   - Need to resolve store from `order.storeId` (pass store data or lookup)

4. **Update OrderListTable:**
   - May need to pass store data to OrderRow

## Todo List

- [x] Create PlatformIcon component
- [x] Add PLATFORM_CONFIG to constants
- [x] Update OrderRow to show external ID
- [x] Update OrderRow to show platform icon + domain
- [x] Handle store lookup/passing
- [x] Test all platform variants

## Success Criteria

- External ID displays as `#WC-xxxxx` format
- Platform icons show correct colors
- Domain displays instead of platform name
- Manual orders show download icon

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Store data not available in OrderRow | Medium | Medium | Pass store via props or use lookup |
| Color contrast issues | Low | Low | Tested colors meet WCAG AA |

## Security Considerations

- External ID is display-only, no XSS risk
- Domain should be sanitized if from user input (mock data is safe)

## Next Steps

→ Phase 04: Item row DESIGN & TASK columns
