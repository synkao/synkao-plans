---
title: Phase 1 - Image Lightbox Dialog
status: completed
priority: high
effort: 3
tags: [image, dialog, ui]
completed: 2026-01-25T15:01:00Z
---

# Phase 1: Image Lightbox Dialog

## Context
- [order-item-row.tsx](../../apps/web/src/features/orders/components/order-list/order-item-row.tsx) - Lines 48-62 contain thumbnail
- [dialog.tsx](../../apps/web/src/components/ui/dialog.tsx) - Radix Dialog wrapper

## Overview
- **Priority:** P1
- **Status:** ✅ Completed
- **Scope:** Click product thumbnail → Dialog shows large image

## Requirements

### Functional
- Click 56x56 thumbnail opens dialog with large image
- Dialog max width ~800px for image display
- Esc key closes dialog
- Click outside closes dialog

### Non-functional
- No zoom/pan controls needed
- Simple display only

## Architecture

```
OrderListTable (state: selectedImage)
  └── OrderGroup
       └── OrderItemRow
            └── Thumbnail (onClick → lift state)

ImageLightboxDialog (controlled by OrderListTable)
```

## Related Files

### Create
| File | Purpose |
|------|---------|
| `image-lightbox-dialog.tsx` | Dialog component for fullscreen image |

### Modify
| File | Changes |
|------|---------|
| `order-item-row.tsx` | Add `onImageClick` prop, cursor-pointer |
| `order-list-table.tsx` | Add state + pass handler + render dialog |
| `index.ts` | Export new component |

## Implementation

### Step 1: Create `image-lightbox-dialog.tsx`

```tsx
// apps/web/src/features/orders/components/order-list/image-lightbox-dialog.tsx
'use client';

import {
  Dialog,
  DialogContent,
} from '@/components/ui/dialog';
import Image from 'next/image';

interface ImageLightboxDialogProps {
  imageUrl: string | null;
  alt: string;
  open: boolean;
  onOpenChange: (open: boolean) => void;
}

export function ImageLightboxDialog({
  imageUrl,
  alt,
  open,
  onOpenChange,
}: ImageLightboxDialogProps) {
  if (!imageUrl) return null;

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="max-w-3xl p-2 bg-black/90">
        <div className="relative w-full aspect-square max-h-[80vh]">
          <Image
            src={imageUrl}
            alt={alt}
            fill
            className="object-contain"
            sizes="(max-width: 768px) 100vw, 800px"
            unoptimized
          />
        </div>
      </DialogContent>
    </Dialog>
  );
}
```

### Step 2: Update `order-item-row.tsx`

Add prop and click handler:

```tsx
// Add to OrderItemRowProps
onImageClick?: (imageUrl: string, alt: string) => void;

// Update thumbnail div (line ~48)
<div
  className="h-14 w-14 flex-shrink-0 overflow-hidden rounded border border-gray-200 bg-gray-100 relative cursor-pointer hover:ring-2 hover:ring-blue-500 transition-all"
  onClick={(e) => {
    e.stopPropagation();
    if (item.productImageUrl && onImageClick) {
      onImageClick(item.productImageUrl, item.productName);
    }
  }}
  role="button"
  aria-label={`View ${item.productName} image`}
>
```

### Step 3: Update `order-list-table.tsx`

Add state management:

```tsx
// Add state
const [lightboxImage, setLightboxImage] = useState<{url: string; alt: string} | null>(null);

// Handler
const handleImageClick = (url: string, alt: string) => {
  setLightboxImage({ url, alt });
};

// Pass to OrderItemRow
<OrderItemRow
  ...
  onImageClick={handleImageClick}
/>

// Render dialog at component end
<ImageLightboxDialog
  imageUrl={lightboxImage?.url ?? null}
  alt={lightboxImage?.alt ?? ''}
  open={!!lightboxImage}
  onOpenChange={(open) => !open && setLightboxImage(null)}
/>
```

### Step 4: Update exports

```tsx
// index.ts
export { ImageLightboxDialog } from './image-lightbox-dialog';
```

## Todo List

- [x] Create `image-lightbox-dialog.tsx`
- [x] Add `onImageClick` prop to `OrderItemRowProps`
- [x] Add click handler to thumbnail in `order-item-row.tsx`
- [x] Add state management in `order-list-table.tsx`
- [x] Render `ImageLightboxDialog` in `order-list-table.tsx`
- [x] Update `index.ts` exports
- [x] Run TypeScript compilation check

## Success Criteria

- [x] Click thumbnail opens dialog
- [x] Large image displays correctly
- [x] Esc/outside click closes dialog
- [x] No TypeScript errors
- [x] Consistent with existing UI patterns

## Code Review Notes

**Review Date:** 2026-01-25
**Score:** 8.5/10

**Issues Found:**
- Remove `unoptimized` flag or configure Next.js image domains
- Consider lazy loading for performance

**Status:** ✅ Approved with minor improvements recommended
