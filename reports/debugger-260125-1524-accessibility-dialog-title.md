# Debug Report: Dialog Accessibility Warning

**Date:** 2026-01-25 15:24
**Issue:** Missing DialogTitle for screen reader accessibility
**Severity:** Medium (Accessibility violation)

## Executive Summary

**Root Cause:** `ImageLightboxDialog` component missing required `DialogTitle` within `DialogContent`

**Impact:** Screen reader users cannot identify dialog purpose, violating WCAG accessibility standards

**Status:** ✅ Identified | 📋 Solution Ready

---

## Technical Analysis

### Affected Components

#### ❌ ImageLightboxDialog (Missing DialogTitle)
**File:** `apps/web/src/features/orders/components/order-list/image-lightbox-dialog.tsx`

```tsx
// Lines 29-42 - Current implementation
<Dialog open={open} onOpenChange={onOpenChange}>
  <DialogContent className="max-w-3xl p-2 bg-black/90 border-none">
    <div className="relative w-full aspect-square max-h-[80vh]">
      <Image src={imageUrl} alt={alt} fill ... />
    </div>
  </DialogContent>
</Dialog>
```

**Problem:** No `DialogTitle` component present. Radix UI requires title for screen reader announcements.

#### ✅ ImportOrdersDialog (Compliant)
**File:** `apps/web/src/features/orders/components/order-list/import-orders-dialog.tsx`

```tsx
// Lines 48-58 - Correct implementation
<DialogContent className="sm:max-w-lg">
  <DialogHeader>
    <DialogTitle className="flex items-center gap-2">
      <Upload className="h-5 w-5" />
      Import Orders
    </DialogTitle>
    <DialogDescription>...</DialogDescription>
  </DialogHeader>
  ...
</DialogContent>
```

**Status:** ✅ Already has DialogTitle, no changes needed

---

## Solution: Hidden Title Pattern

### Implementation Strategy

Use `sr-only` class (screen reader only) to add visually hidden title for lightbox dialogs where visible title would harm UX.

### Evidence: SR-Only Pattern Available

Found in existing codebase:
- `apps/web/src/components/ui/dialog.tsx` line 75: `<span className="sr-only">Close</span>`
- Pattern already used for accessibility without visual impact

### Recommended Fix

```tsx
// apps/web/src/features/orders/components/order-list/image-lightbox-dialog.tsx

'use client';

import {
  Dialog,
  DialogContent,
  DialogTitle, // ADD THIS IMPORT
} from '@/components/ui/dialog';
import Image from 'next/image';

// ... interface unchanged ...

export function ImageLightboxDialog({
  imageUrl,
  alt,
  open,
  onOpenChange,
}: ImageLightboxDialogProps) {
  if (!imageUrl) return null;

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="max-w-3xl p-2 bg-black/90 border-none">
        {/* ADD HIDDEN TITLE FOR SCREEN READERS */}
        <DialogTitle className="sr-only">{alt}</DialogTitle>

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

---

## Implementation Steps

1. **Open file:** `apps/web/src/features/orders/components/order-list/image-lightbox-dialog.tsx`
2. **Add import:** Include `DialogTitle` in import statement (line 3-6)
3. **Add hidden title:** Insert `<DialogTitle className="sr-only">{alt}</DialogTitle>` inside `DialogContent` before image div (after line 30)
4. **Verify:** Run dev server and check console for accessibility warnings
5. **Test:** Confirm screen readers announce dialog purpose correctly

---

## Validation Criteria

- [ ] No accessibility warnings in console
- [ ] Screen reader announces image alt text when dialog opens
- [ ] Visual appearance unchanged (title remains hidden)
- [ ] Dialog still functions correctly (open/close behavior)

---

## Related Files

### Modified
- `apps/web/src/features/orders/components/order-list/image-lightbox-dialog.tsx`

### Reference (No changes needed)
- `apps/web/src/features/orders/components/order-list/import-orders-dialog.tsx` (correct pattern)
- `apps/web/src/components/ui/dialog.tsx` (sr-only class available)

---

## Risk Assessment

**Risk Level:** Low

**Considerations:**
- Non-breaking change (additive only)
- Uses existing sr-only pattern from codebase
- No visual impact to users
- Improves WCAG compliance

**Mitigation:**
- Test with screen reader (VoiceOver/NVDA) after implementation
- Verify alt text provides meaningful context

---

## Additional Context

### Why Alt Text Works as Title
The `alt` prop already contains descriptive text for the image (e.g., "Product XYZ image"). Using it as the hidden DialogTitle provides meaningful context to screen reader users without redundancy.

### Alternative Approach (Not Recommended)
Creating custom VisuallyHidden component would be over-engineering since `sr-only` class already exists and is Tailwind CSS standard.

---

## Unresolved Questions

None. Solution is straightforward and follows established patterns.
