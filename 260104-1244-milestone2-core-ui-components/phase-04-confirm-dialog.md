# Phase 04: Confirm Dialog

## Context

- **Parent Plan:** [plan.md](plan.md)
- **Issue:** #11 Modal, Drawer, Toast
- **Dependencies:** Phase 01 (Toast for feedback)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P0 |
| Effort | 45 min |
| Implementation | ⬜ Pending |
| Review | ⬜ Pending |

## Key Insights

- shadcn has `alert-dialog` component for confirmations
- Need wrapper component with simpler API
- Support destructive variant for delete actions

## Requirements

1. Add shadcn alert-dialog
2. Create ConfirmDialog wrapper with clean props API
3. Support default and destructive variants

## Related Files

| File | Action |
|------|--------|
| `apps/web/src/components/ui/alert-dialog.tsx` | Create (via shadcn) |
| `apps/web/src/components/ui/confirm-dialog.tsx` | Create |

## Implementation Steps

### Step 1: Add shadcn alert-dialog

```bash
pnpm dlx shadcn@latest add alert-dialog --cwd apps/web
```

This creates `components/ui/alert-dialog.tsx` with:
- AlertDialog, AlertDialogTrigger, AlertDialogContent
- AlertDialogHeader, AlertDialogFooter
- AlertDialogTitle, AlertDialogDescription
- AlertDialogAction, AlertDialogCancel

### Step 2: Create confirm-dialog.tsx

**File:** `apps/web/src/components/ui/confirm-dialog.tsx`

```tsx
"use client"

import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
} from "@/components/ui/alert-dialog"
import { cn } from "@/lib/utils"

export interface ConfirmDialogProps {
  /** Control dialog visibility */
  open: boolean
  /** Callback when open state changes */
  onOpenChange: (open: boolean) => void
  /** Dialog title */
  title: string
  /** Optional description text */
  description?: string
  /** Confirm button text */
  confirmText?: string
  /** Cancel button text */
  cancelText?: string
  /** Callback when confirm is clicked */
  onConfirm: () => void
  /** Visual variant - destructive for delete actions */
  variant?: "default" | "destructive"
  /** Disable confirm button (e.g., during loading) */
  confirmDisabled?: boolean
}

/**
 * Confirmation dialog for destructive or important actions
 *
 * @example
 * <ConfirmDialog
 *   open={showDelete}
 *   onOpenChange={setShowDelete}
 *   title="Delete order?"
 *   description="This action cannot be undone."
 *   confirmText="Delete"
 *   variant="destructive"
 *   onConfirm={handleDelete}
 * />
 */
export function ConfirmDialog({
  open,
  onOpenChange,
  title,
  description,
  confirmText = "Confirm",
  cancelText = "Cancel",
  onConfirm,
  variant = "default",
  confirmDisabled = false,
}: ConfirmDialogProps) {
  const handleConfirm = () => {
    onConfirm()
    onOpenChange(false)
  }

  return (
    <AlertDialog open={open} onOpenChange={onOpenChange}>
      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle>{title}</AlertDialogTitle>
          {description && (
            <AlertDialogDescription>{description}</AlertDialogDescription>
          )}
        </AlertDialogHeader>
        <AlertDialogFooter>
          <AlertDialogCancel>{cancelText}</AlertDialogCancel>
          <AlertDialogAction
            onClick={handleConfirm}
            disabled={confirmDisabled}
            className={cn(
              variant === "destructive" &&
                "bg-destructive text-destructive-foreground hover:bg-destructive/90"
            )}
          >
            {confirmText}
          </AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  )
}
```

### Step 3: Usage example

```tsx
import { ConfirmDialog } from "@/components/ui/confirm-dialog"
import { toast } from "sonner"

function OrderActions() {
  const [showDelete, setShowDelete] = useState(false)

  const handleDelete = async () => {
    await deleteOrder(orderId)
    toast.success("Order deleted")
  }

  return (
    <>
      <Button variant="destructive" onClick={() => setShowDelete(true)}>
        Delete
      </Button>

      <ConfirmDialog
        open={showDelete}
        onOpenChange={setShowDelete}
        title="Delete this order?"
        description="This action cannot be undone. All order data will be permanently removed."
        confirmText="Delete Order"
        variant="destructive"
        onConfirm={handleDelete}
      />
    </>
  )
}
```

## Todo

- [ ] Add shadcn alert-dialog
- [ ] Create confirm-dialog.tsx wrapper
- [ ] Test with destructive variant
- [ ] Verify build passes

## Success Criteria

- [ ] AlertDialog components available
- [ ] ConfirmDialog exported with clean API
- [ ] Destructive variant has red styling
- [ ] Auto-closes after confirm
- [ ] Build passes

## Risk Assessment

| Risk | Level | Mitigation |
|------|-------|------------|
| Missing alert-dialog | Low | Install via shadcn CLI |
| Focus trap issues | Low | shadcn handles accessibility |

## Security Considerations

- Ensure confirm action is intentional (require click, not just Enter)
- Consider loading state to prevent double-submit

## Next Steps

After completion → All phases done, run final verification
