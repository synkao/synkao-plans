# Brainstorm Report: Mock Data với Images

**Date:** 2026-01-03
**Objective:** Thêm placeholder images cho mock data

---

## Problem Statement

Mock data hiện tại thiếu images, cần:
- User avatars
- Product images cho order items
- Design previews cho tasks
- Design version files

---

## Evaluated Approaches

### Option A: Local Static Files
- Download images vào `public/`
- **Rejected**: Tăng repo size, phải maintain

### Option B: AI Generated Images
- Dùng Imagen/DALL-E tạo images
- **Rejected**: Overkill cho mock data, tốn API credits

### Option C: Placeholder Services ✅ Selected
- DiceBear cho avatars
- Picsum cho product/design images
- **Selected**: Zero maintenance, consistent, free

---

## Final Solution

### Placeholder Services

| Data Type | Service | URL Pattern |
|-----------|---------|-------------|
| User Avatar | DiceBear 9.x | `https://api.dicebear.com/9.x/avataaars/svg?seed={name}` |
| Product Image | Picsum | `https://picsum.photos/seed/item-{id}/400/400` |
| Design Preview | Picsum | `https://picsum.photos/seed/design-{taskId}/800/600` |
| Design Version | Picsum | `https://picsum.photos/seed/version-{id}/800/600` |

### Files to Modify

| File | Change |
|------|--------|
| `mocks/types.ts` | Add `productImageUrl?: string` to MockOrderItem |
| `mocks/data/users.data.ts` | Populate `avatar` field |
| `mocks/data/order-items.data.ts` | Add `productImageUrl` |
| `mocks/data/tasks.data.ts` | Update `designUrl` pattern |
| `mocks/data/design-versions.data.ts` | Update `fileUrl` pattern |

---

## Implementation

### 1. types.ts

```typescript
export interface MockOrderItem {
  // ... existing fields
  productImageUrl?: string; // ADD THIS
}
```

### 2. users.data.ts

```typescript
export const mockUsers: MockUser[] = [
  {
    id: USER_IDS.admin,
    email: 'admin@synkao.com',
    name: 'John Smith',
    role: 'ADMIN',
    avatar: 'https://api.dicebear.com/9.x/avataaars/svg?seed=John%20Smith'
  },
  // ... etc
];
```

### 3. order-items.data.ts

```typescript
mockOrderItems.push({
  // ... existing fields
  productImageUrl: `https://picsum.photos/seed/item-${itemIndex}/400/400`,
});
```

### 4. tasks.data.ts

```typescript
designUrl: hasDesign
  ? `https://picsum.photos/seed/design-${n}/800/600`
  : undefined,
```

---

## Success Criteria

- [x] All mock users have avatar URLs
- [x] All order items have product image URLs
- [x] Tasks with designs have preview URLs
- [x] Design versions have file URLs
- [x] URLs use deterministic seeds (same data = same image)

---

## Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Service downtime | Low | Placeholder services highly reliable |
| CORS issues | Low | Both services support CORS |
| Image load time | Low | Services have CDN caching |

---

## Effort Estimate

- Total: ~10 minutes implementation
- Files: 5
- Lines changed: ~20
