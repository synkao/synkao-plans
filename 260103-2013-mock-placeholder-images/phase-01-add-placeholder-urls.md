# Phase 01: Add Placeholder URLs

## Context

- **Parent Plan:** [plan.md](plan.md)
- **Reference:** [Brainstorm Report](reports/brainstorm-260103-2013-mock-images.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-03 |
| Priority | P3 |
| Implementation | completed |
| Review | completed |
| Completed | 2026-01-03 |

## Key Insights

- DiceBear 9.x avataaars style cho user avatars
- Picsum với seed parameter cho deterministic images
- Existing `avatar` field in MockUser already defined

## Requirements

1. Add `productImageUrl` field to MockOrderItem type
2. Populate `avatar` for all mock users
3. Add `productImageUrl` for all order items
4. Update `designUrl` pattern in tasks
5. Update `fileUrl` pattern in design versions

## Related Files

- `apps/web/src/mocks/types.ts`
- `apps/web/src/mocks/data/users.data.ts`
- `apps/web/src/mocks/data/order-items.data.ts`
- `apps/web/src/mocks/data/tasks.data.ts`
- `apps/web/src/mocks/data/design-versions.data.ts`

## Implementation Steps

### Step 1: Update types.ts

Add to MockOrderItem interface:
```typescript
productImageUrl?: string;
```

### Step 2: Update users.data.ts

Add avatar field to each user:
```typescript
{
  id: USER_IDS.admin,
  email: 'admin@synkao.com',
  name: 'John Smith',
  role: 'ADMIN',
  avatar: 'https://api.dicebear.com/9.x/avataaars/svg?seed=John%20Smith'
},
```

### Step 3: Update order-items.data.ts

Add productImageUrl in the push:
```typescript
mockOrderItems.push({
  // ... existing fields
  productImageUrl: `https://picsum.photos/seed/item-${itemIndex}/400/400`,
});
```

### Step 4: Update tasks.data.ts

Change designUrl pattern:
```typescript
designUrl: hasDesign
  ? `https://picsum.photos/seed/design-${n}/800/600`
  : undefined,
```

### Step 5: Update design-versions.data.ts

Change fileUrl pattern:
```typescript
fileUrl: `https://picsum.photos/seed/version-${id}/800/600`,
```

## Todo List

- [ ] Add productImageUrl to MockOrderItem type
- [ ] Add avatar URLs to users
- [ ] Add productImageUrl to order items
- [ ] Update designUrl in tasks
- [ ] Update fileUrl in design versions
- [ ] Verify TypeScript compiles

## Success Criteria

- [ ] 5 files modified
- [ ] ~20 lines changed
- [ ] TypeScript compiles without errors
- [ ] All mock data has image URLs

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Service downtime | Low | External services highly reliable |
| CORS issues | Low | Both services support CORS |

## Security Considerations

None - mock data only, external placeholder services.

## Next Steps

1. Implement changes
2. Verify compilation
3. Visual test in dev
