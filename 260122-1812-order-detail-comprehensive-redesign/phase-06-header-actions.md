# Phase 06: UI - Header Actions & Dialogs

**Priority:** P1 - High
**Status:** ✅ Complete
**Depends on:** Phase 02
**Completion Score:** 9/10
**Completed:** 2026-01-22 19:53

## Overview

Add header action buttons and dialogs:
- Edit Notes button → Dialog with textarea
- Cancel Order button → Confirmation dialog with reason

## Related Files

**Create:**
- `apps/web/src/features/orders/components/order-detail/edit-notes-dialog.tsx`
- `apps/web/src/features/orders/components/order-detail/cancel-order-dialog.tsx`

**Modify:**
- `apps/web/src/features/orders/components/order-detail/index.ts` - Export dialogs
- `apps/web/src/app/(main)/orders/[id]/page.tsx` - Add buttons and dialogs

## Design

### Header Actions

```
┌─────────────────────────────────────────────────────────────┐
│ ← Back     ORD-0001                                        │
│            Order from John Smith                            │
│                                                             │
│            [Edit Notes] [Cancel Order] [Create Fulfillment] │
└─────────────────────────────────────────────────────────────┘
```

### Edit Notes Dialog

```
┌─────────────────────────────────────────────────┐
│ Edit Order Notes                            [X] │
├─────────────────────────────────────────────────┤
│                                                 │
│ ┌─────────────────────────────────────────────┐ │
│ │ Customer requests delivery before 12:00 PM.│ │
│ │ Please call 30 minutes before delivery.    │ │
│ │                                             │ │
│ └─────────────────────────────────────────────┘ │
│                                                 │
│                        [Cancel]  [Save Changes] │
└─────────────────────────────────────────────────┘
```

### Cancel Order Dialog

```
┌─────────────────────────────────────────────────┐
│ ⚠️ Cancel Order                             [X] │
├─────────────────────────────────────────────────┤
│                                                 │
│ Are you sure you want to cancel order ORD-0001?│
│ This action cannot be undone.                   │
│                                                 │
│ Reason (optional):                              │
│ ┌─────────────────────────────────────────────┐ │
│ │ Customer requested cancellation            │ │
│ └─────────────────────────────────────────────┘ │
│                                                 │
│                    [Go Back]  [Cancel Order]    │
└─────────────────────────────────────────────────┘
```

## Implementation

### EditNotesDialog

```tsx
// edit-notes-dialog.tsx

'use client';

import { useState, useEffect } from 'react';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
} from '@/components/ui/dialog';
import { Button } from '@/components/ui/button';
import { Textarea } from '@/components/ui/textarea';
import { useUpdateOrderNotes } from '../../hooks';
import { toast } from 'sonner';

interface EditNotesDialogProps {
  orderId: string;
  currentNotes?: string;
  open: boolean;
  onOpenChange: (open: boolean) => void;
}

export function EditNotesDialog({
  orderId,
  currentNotes,
  open,
  onOpenChange,
}: EditNotesDialogProps) {
  const [notes, setNotes] = useState(currentNotes ?? '');
  const { mutate: updateNotes, isPending } = useUpdateOrderNotes(orderId);

  useEffect(() => {
    if (open) {
      setNotes(currentNotes ?? '');
    }
  }, [open, currentNotes]);

  const handleSave = () => {
    updateNotes(notes, {
      onSuccess: () => {
        toast.success('Notes updated');
        onOpenChange(false);
      },
      onError: (err) => {
        toast.error(err.message);
      },
    });
  };

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="sm:max-w-[500px]">
        <DialogHeader>
          <DialogTitle>Edit Order Notes</DialogTitle>
          <DialogDescription>
            Add or update notes for this order. Notes are visible to all team members.
          </DialogDescription>
        </DialogHeader>
        <Textarea
          value={notes}
          onChange={(e) => setNotes(e.target.value)}
          placeholder="Enter order notes..."
          rows={5}
          className="resize-none"
        />
        <DialogFooter>
          <Button variant="outline" onClick={() => onOpenChange(false)}>
            Cancel
          </Button>
          <Button onClick={handleSave} disabled={isPending}>
            {isPending ? 'Saving...' : 'Save Changes'}
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}
```

### CancelOrderDialog

```tsx
// cancel-order-dialog.tsx

'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
} from '@/components/ui/alert-dialog';
import { Textarea } from '@/components/ui/textarea';
import { Label } from '@/components/ui/label';
import { useCancelOrder } from '../../hooks';
import { toast } from 'sonner';

interface CancelOrderDialogProps {
  orderId: string;
  orderNumber: string;
  open: boolean;
  onOpenChange: (open: boolean) => void;
}

export function CancelOrderDialog({
  orderId,
  orderNumber,
  open,
  onOpenChange,
}: CancelOrderDialogProps) {
  const [reason, setReason] = useState('');
  const router = useRouter();
  const { mutate: cancelOrder, isPending } = useCancelOrder(orderId);

  const handleCancel = () => {
    cancelOrder(reason || undefined, {
      onSuccess: () => {
        toast.success('Order cancelled');
        onOpenChange(false);
        router.push('/orders');
      },
      onError: (err) => {
        toast.error(err.message);
      },
    });
  };

  return (
    <AlertDialog open={open} onOpenChange={onOpenChange}>
      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle className="text-red-600">
            Cancel Order
          </AlertDialogTitle>
          <AlertDialogDescription>
            Are you sure you want to cancel order <strong>{orderNumber}</strong>?
            This action cannot be undone.
          </AlertDialogDescription>
        </AlertDialogHeader>
        <div className="py-4">
          <Label htmlFor="reason" className="text-sm font-medium">
            Reason (optional)
          </Label>
          <Textarea
            id="reason"
            value={reason}
            onChange={(e) => setReason(e.target.value)}
            placeholder="Customer requested cancellation..."
            rows={3}
            className="mt-2 resize-none"
          />
        </div>
        <AlertDialogFooter>
          <AlertDialogCancel>Go Back</AlertDialogCancel>
          <AlertDialogAction
            onClick={handleCancel}
            disabled={isPending}
            className="bg-red-600 hover:bg-red-700"
          >
            {isPending ? 'Cancelling...' : 'Cancel Order'}
          </AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  );
}
```

### Update page.tsx

```tsx
// Add state
const [editNotesOpen, setEditNotesOpen] = useState(false);
const [cancelOrderOpen, setCancelOrderOpen] = useState(false);

// Check if order can be cancelled
const canCancel = order && !['SHIPPED', 'COMPLETED', 'CANCELLED'].includes(order.status);

// Update header actions
<div className="flex gap-2">
  <Button variant="outline" onClick={handleBack}>
    <ArrowLeft className="mr-2 h-4 w-4" />
    Back
  </Button>
  <Button variant="outline" onClick={() => setEditNotesOpen(true)}>
    <Edit className="mr-2 h-4 w-4" />
    Edit Notes
  </Button>
  {canCancel && (
    <Button
      variant="outline"
      className="text-red-600 hover:text-red-700 hover:bg-red-50"
      onClick={() => setCancelOrderOpen(true)}
    >
      <X className="mr-2 h-4 w-4" />
      Cancel Order
    </Button>
  )}
  <Button onClick={handleCreateFulfillment}>
    <Truck className="mr-2 h-4 w-4" />
    Create Fulfillment
  </Button>
</div>

// Add dialogs at end of component
<EditNotesDialog
  orderId={id}
  currentNotes={order.notes}
  open={editNotesOpen}
  onOpenChange={setEditNotesOpen}
/>
<CancelOrderDialog
  orderId={id}
  orderNumber={order.orderNumber}
  open={cancelOrderOpen}
  onOpenChange={setCancelOrderOpen}
/>
```

## Todo

- [x] Create EditNotesDialog component ✅
- [x] Create CancelOrderDialog component ✅
- [x] Export from order-detail/index.ts ✅
- [x] Add dialog state to page.tsx ✅
- [x] Add Edit Notes button to header ✅
- [x] Add Cancel Order button (conditional) ✅
- [x] Wire up dialogs to mutation hooks ✅
- [x] Add maxLength validation for textareas ✅
- [x] Add character counters ✅
- [x] Test edit notes flow ✅
- [x] Test cancel order flow ✅

## Success Criteria

- Edit Notes opens dialog with current notes
- Save updates notes and shows toast
- Cancel Order shows confirmation
- Cancel with reason works
- Cannot cancel shipped/completed orders
- Redirect to /orders after cancel

## Implementation Notes

### Key Features Implemented

1. **EditNotesDialog**
   - Character limit: 500 characters for notes
   - Character counter display
   - maxLength attribute on textarea
   - State reset on dialog open/close
   - Mutation hook integration with toast notifications

2. **CancelOrderDialog**
   - Character limit: 200 characters for cancel reason
   - Character counter display
   - maxLength attribute on textarea
   - Confirmation warning with order number
   - Conditional rendering (cannot cancel shipped/completed/cancelled)
   - Redirect to /orders on successful cancel

3. **Enhancements Applied**
   - Standardized error handling in both dialogs
   - Explicit state cleanup on dialog close
   - Proper type safety for error messages
   - Toast notifications for user feedback

## Next Phase

→ [Phase 07: Customer Info Enhancement](phase-07-customer-info-enhancement.md)
