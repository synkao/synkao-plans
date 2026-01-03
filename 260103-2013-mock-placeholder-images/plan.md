---
title: "Mock Placeholder Images"
description: "Thêm avatar, product image, design preview URLs cho mock data"
status: completed
priority: P3
effort: 15m
branch: main
tags: [mock, images, placeholder]
created: 2026-01-03
completed: 2026-01-03
---

# Mock Placeholder Images

## Overview

Thêm placeholder image URLs cho mock data sử dụng:
- **DiceBear** cho user avatars (avataaars style)
- **Picsum** cho product images và design previews

## References

- [Brainstorm Report](reports/brainstorm-260103-2013-mock-images.md)

## Implementation Phases

| Phase | Description | Status | Progress |
|-------|-------------|--------|----------|
| [Phase 01](phase-01-add-placeholder-urls.md) | Add placeholder URLs to mock data | completed | 100% |
| [Phase 02](phase-02-render-images-in-components.md) | Render images in UI components | completed | 100% |

## URL Patterns

```
User Avatar:    https://api.dicebear.com/9.x/avataaars/svg?seed={name}
Product Image:  https://picsum.photos/seed/item-{id}/400/400
Design Preview: https://picsum.photos/seed/design-{taskId}/800/600
Design Version: https://picsum.photos/seed/version-{id}/800/600
```

## Files to Modify

1. `mocks/types.ts` - Add productImageUrl field
2. `mocks/data/users.data.ts` - Populate avatar
3. `mocks/data/order-items.data.ts` - Add productImageUrl
4. `mocks/data/tasks.data.ts` - Update designUrl
5. `mocks/data/design-versions.data.ts` - Update fileUrl

## Success Criteria

- [x] All users have avatar URLs
- [x] All order items have product image URLs
- [x] Tasks with designs have preview URLs
- [x] TypeScript compiles without errors

## Results

**Status:** ✅ Completed successfully

- Phase 01: Placeholder URLs added to mock data
- Phase 02: Images rendered in task-card and user-menu
- Build: Successful (compiled in 12.8s)
- Review: [Code Review Report](reports/code-reviewer-260103-2143-phase02-images.md)

### Key Achievements

✅ User avatars using DiceBear API
✅ Design thumbnails using Picsum
✅ Proper fallbacks implemented
✅ Type-safe implementation
✅ Zero breaking changes

### Minor Notes

- Next.js Image optimization warning (acceptable for mock phase)
- Consider migrating `<img>` to `<Image />` when production CDN ready
