---
title: Phase 3 - Note Edit Popover
status: completed
priority: high
effort: 2
tags: [notes, popover, ui]
completed: 2026-01-25T15:01:00Z
---

# Phase 3: Note Edit Popover

## Context
- [order-row.tsx](../../apps/web/src/features/orders/components/order-list/order-row.tsx) - Lines 215-233 (Note column)
- [status-edit-popover.tsx](../../apps/web/src/features/orders/components/order-list/status-edit-popover.tsx) - Reference pattern
- [use-orders.ts](../../apps/web/src/features/orders/hooks/use-orders.ts) - `useUpdateOrderNotes` hook exists

## Overview
- **Priority:** P2
- **Status:** ✅ Completed
- **Scope:** Click pencil icon → Popover with textarea → Save/Cancel

## Requirements

### Functional
- Pencil icon next to note text
- Click pencil → Opens popover
- Textarea with 255 char limit
- Character counter display
- Save/Cancel buttons
- Manual save (click Save button)

### Non-functional
- Note is Order-level (not OrderItem)
- Use existing `useUpdateOrderNotes` hook
- Follow `StatusEditPopover` pattern

## Architecture

```
OrderRow
  └── Note Column
       ├── Note Text (truncated, ellipsis)
       └── NoteEditPopover
            ├── Trigger: Pencil Icon
            └── Content: Textarea + Counter + Buttons
```

## Related Files

### Create
| File | Purpose |
|------|---------|
| `note-edit-popover.tsx` | Popover with textarea for note editing |

### Modify
| File | Changes |
|------|---------|
| `order-row.tsx` | Add NoteEditPopover, add `onNoteChange` prop |
| `order-list-table.tsx` | Pass `onNoteChange` handler |

## Implementation

### Step 1: Create `note-edit-popover.tsx`

```tsx
// apps/web/src/features/orders/components/order-list/note-edit-popover.tsx
'use client';

import { useState } from 'react';
import {
  Popover,
  PopoverContent,
  PopoverTrigger,
} from '@/components/ui/popover';
import { Button } from '@/components/ui/button';
import { Textarea } from '@/components/ui/textarea';
import { Pencil } from 'lucide-react';
import { cn } from '@/lib/utils';

const MAX_LENGTH = 255;

interface NoteEditPopoverProps {
  orderId: string;
  currentNote: string;
  onSave: (orderId: string, newNote: string) => void;
  isUpdating?: boolean;
}

export function NoteEditPopover({
  orderId,
  currentNote,
  onSave,
  isUpdating = false,
}: NoteEditPopoverProps) {
  const [open, setOpen] = useState(false);
  const [note, setNote] = useState(currentNote);

  const charCount = note.length;
  const isOverLimit = charCount > MAX_LENGTH;
  const hasChanges = note !== currentNote;

  const handleSave = () => {
    if (!isOverLimit && hasChanges) {
      onSave(orderId, note);
      setOpen(false);
    }
  };

  const handleCancel = () => {
    setNote(currentNote);
    setOpen(false);
  };

  const handleOpenChange = (isOpen: boolean) => {
    if (isOpen) {
      setNote(currentNote);
    }
    setOpen(isOpen);
  };

  return (
    <Popover open={open} onOpenChange={handleOpenChange}>
      <PopoverTrigger asChild>
        <button
          className="p-1 hover:bg-gray-100 rounded transition-colors"
          aria-label="Edit note"
          disabled={isUpdating}
        >
          {isUpdating ? (
            <span className="h-4 w-4 animate-spin rounded-full border-2 border-gray-300 border-t-gray-600 inline-block" />
          ) : (
            <Pencil className="h-4 w-4 text-gray-400 hover:text-gray-600" />
          )}
        </button>
      </PopoverTrigger>
      <PopoverContent className="w-80 p-3" align="start">
        <div className="space-y-3">
          <div>
            <Textarea
              value={note}
              onChange={(e) => setNote(e.target.value)}
              placeholder="Add a note..."
              className="min-h-[80px] resize-none"
              maxLength={MAX_LENGTH + 10} // Allow typing slightly over for UX
              disabled={isUpdating}
            />
            <div className={cn(
              "text-xs mt-1 text-right",
              isOverLimit ? "text-red-500" : "text-gray-400"
            )}>
              {charCount}/{MAX_LENGTH}
            </div>
          </div>
          <div className="flex justify-end gap-2">
            <Button
              variant="outline"
              size="sm"
              onClick={handleCancel}
              disabled={isUpdating}
            >
              Cancel
            </Button>
            <Button
              size="sm"
              onClick={handleSave}
              disabled={isUpdating || isOverLimit || !hasChanges}
            >
              {isUpdating ? 'Saving...' : 'Save'}
            </Button>
          </div>
        </div>
      </PopoverContent>
    </Popover>
  );
}
```

### Step 2: Update `order-row.tsx`

Add prop and update Note column (lines 215-233):

```tsx
// Add to OrderRowProps
onNoteChange?: (orderId: string, newNote: string) => void;
isNoteUpdating?: boolean;

// Replace Note column (line ~215)
{/* Note */}
<td className="px-4 py-3" onClick={(e) => e.stopPropagation()}>
  <div className="flex items-center gap-1">
    {order.notes ? (
      <TooltipProvider>
        <Tooltip>
          <TooltipTrigger asChild>
            <span className="text-sm text-gray-600 truncate max-w-[80px] block cursor-help">
              {order.notes}
            </span>
          </TooltipTrigger>
          <TooltipContent side="top" className="max-w-[300px]">
            <p className="text-sm">{order.notes}</p>
          </TooltipContent>
        </Tooltip>
      </TooltipProvider>
    ) : (
      <span className="text-sm text-gray-400">—</span>
    )}
    {onNoteChange && (
      <NoteEditPopover
        orderId={order.id}
        currentNote={order.notes ?? ''}
        onSave={onNoteChange}
        isUpdating={isNoteUpdating}
      />
    )}
  </div>
</td>
```

### Step 3: Update `order-list-table.tsx`

Pass handler to OrderRow:

```tsx
// Add to OrderListTableProps
onNoteChange?: (orderId: string, newNote: string) => void;

// Pass to OrderGroup
<OrderGroup
  ...
  onNoteChange={onNoteChange}
/>

// In OrderGroupProps and OrderGroup component
onNoteChange?: (orderId: string, newNote: string) => void;

// Pass to OrderRow
<OrderRow
  ...
  onNoteChange={onNoteChange}
/>
```

### Step 4: Update exports

```tsx
// index.ts
export { NoteEditPopover } from './note-edit-popover';
```

## Todo List

- [x] Create `note-edit-popover.tsx`
- [x] Add `onNoteChange` prop to `OrderRowProps`
- [x] Update Note column in `order-row.tsx`
- [x] Add import for `NoteEditPopover`
- [x] Pass `onNoteChange` through `order-list-table.tsx`
- [x] Update `index.ts` exports
- [x] Run TypeScript compilation check

## Success Criteria

- [x] Pencil icon visible next to note
- [x] Click pencil opens popover
- [x] Textarea shows current note
- [x] Character counter works (X/255)
- [x] Over-limit shows red counter
- [x] Save disabled when no changes or over limit
- [x] Cancel resets and closes
- [x] Save triggers callback
- [x] No TypeScript errors

## Code Review Notes

**Review Date:** 2026-01-25
**Score:** 8.5/10

**Issues Found:**
- **CRITICAL:** Add input sanitization before save (XSS risk)
- Fix CopyButton useEffect dependencies in `order-row.tsx`
- Consider warning color at 90% character limit

**Status:** ✅ Approved with required fixes before merge
