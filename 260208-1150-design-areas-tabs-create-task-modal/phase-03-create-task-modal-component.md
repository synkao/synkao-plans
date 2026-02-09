# Phase 03: Create Task Modal Component

## Context Links

- [Reference screenshot 2](/tmp/comment2.png) - "Design Service" modal with fields
- [Reference screenshot 1](/tmp/comment1.png) - Overview context
- [MockOrderItem type](../../apps/web/src/mocks/types.ts)
- [MockOrder type](../../apps/web/src/mocks/types.ts)
- [DatePicker component](../../apps/web/src/components/ui/date-picker.tsx)
- [shadcn Dialog](../../apps/web/src/components/ui/dialog.tsx)

## Overview

- **Priority:** P2
- **Status:** completed
- **Description:** Create "Design Service" modal dialog for creating design tasks from order items, pre-filled with order/item context

## Key Insights

From reference screenshot 2, the modal contains:
- **Title:** "Design Service" (dialog header)
- **Title field:** Input pre-filled with `{ProductName} - Custom Design`
- **Message field:** Textarea auto-generated with:
  ```
  Order ID #{orderNumber}
  Product Id #{itemCode}
  Meta: {attributes joined by " | "}

  * Personalized text:
  {personalization entries}
  ```
- **Item Image:** Product thumbnail (small preview)
- **Due Date:** Date picker showing date + time
- **Buttons:** "Send Request" (blue filled) + "Cancel" (red outline border)

## Requirements

### Functional
- Dialog title: "Design Service"
- **Title input:** Pre-filled `{productName} - Custom Design`, editable
- **Message textarea:** Auto-generated from order/item data, editable
  - Line 1: `Order ID #{orderNumber}`
  - Line 2: `Product Id #{itemCode}`
  - Line 3: `Meta: {Color} | {Size} | {Gender}` (from attributes)
  - Empty line
  - Line 5: `* Personalized text:` (if personalization exists)
  - Lines 6+: `{key}: {value}` for each personalization entry
- **Item Image:** Show `productImageUrl` thumbnail
- **Due Date:** DatePicker component, defaults to 7 days from now
- **Send Request:** Submit button (blue) - toast success for now (no API)
- **Cancel:** Red outline button, closes dialog
- Dialog controlled via `open`/`onOpenChange` pattern

### Non-Functional
- Component under 200 lines
- Accessible: proper labels, keyboard nav
- Responsive: modal width `sm:max-w-lg`

## Architecture

```
CreateTaskModal
  Props:
    open: boolean
    onOpenChange: (open: boolean) => void
    item: MockOrderItem
    orderNumber: string  // for message generation
    onSubmit?: (data: CreateTaskData) => void

  State:
    title: string (pre-filled)
    message: string (auto-generated)
    dueDate: Date | undefined

  CreateTaskData:
    title: string
    message: string
    dueDate: Date | undefined
    itemId: string

  Renders:
    <Dialog>
      <DialogContent>
        <DialogHeader title="Design Service" />
        <form>
          <Input label="Title" value={title} />
          <Textarea label="Message" value={message} rows={6} />
          <div label="Item Image">
            <Image src={item.productImageUrl} />
          </div>
          <DatePicker label="Due Date" value={dueDate} />
        </form>
        <DialogFooter>
          <Button "Send Request" blue />
          <Button "Cancel" red outline />
        </DialogFooter>
      </DialogContent>
    </Dialog>
```

## Related Code Files

### Files to Create
- `apps/web/src/features/orders/components/order-detail/create-task-modal.tsx`

### Files to Modify
- `apps/web/src/features/orders/components/order-detail/index.ts` - Add export

## Implementation Steps

### 1. Create helper: generate default message

```typescript
function generateTaskMessage(item: MockOrderItem, orderNumber: string): string {
  const lines: string[] = [];
  lines.push(`Order ID #${orderNumber}`);
  lines.push(`Product Id #${item.itemCode}`);

  // Meta from attributes
  if (item.attributes?.length) {
    const meta = item.attributes.map(a => a.value).join(' | ');
    lines.push(`Meta: ${meta}`);
  }

  // Personalization
  if (item.personalization?.length) {
    lines.push('');
    lines.push('* Personalized text:');
    item.personalization.forEach(p => {
      lines.push(`${p.key}: ${p.value}`);
    });
  }

  return lines.join('\n');
}
```

### 2. Create CreateTaskModal component

File: `apps/web/src/features/orders/components/order-detail/create-task-modal.tsx`

```tsx
'use client';

import { useState, useEffect } from 'react';
import Image from 'next/image';
import { toast } from 'sonner';
import {
  Dialog, DialogContent, DialogHeader,
  DialogTitle, DialogFooter,
} from '@/components/ui/dialog';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { DatePicker } from '@/components/ui/date-picker';
import { Label } from '@/components/ui/label';
import type { MockOrderItem } from '../../types';

export interface CreateTaskData {
  title: string;
  message: string;
  dueDate: Date | undefined;
  itemId: string;
}

export interface CreateTaskModalProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  item: MockOrderItem;
  orderNumber: string;
  onSubmit?: (data: CreateTaskData) => void;
}

// generateTaskMessage helper (see step 1)

export function CreateTaskModal({ ... }) {
  const [title, setTitle] = useState('');
  const [message, setMessage] = useState('');
  const [dueDate, setDueDate] = useState<Date | undefined>();

  // Reset form when item changes or modal opens
  useEffect(() => {
    if (open) {
      setTitle(`${item.productName} - Custom Design`);
      setMessage(generateTaskMessage(item, orderNumber));
      setDueDate(new Date(Date.now() + 7 * 24 * 60 * 60 * 1000));
    }
  }, [open, item, orderNumber]);

  const handleSubmit = () => {
    onSubmit?.({ title, message, dueDate, itemId: item.id });
    toast.success('Design task request sent');
    onOpenChange(false);
  };

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="sm:max-w-lg">
        <DialogHeader>
          <DialogTitle>Design Service</DialogTitle>
        </DialogHeader>

        <div className="space-y-4">
          {/* Title */}
          <div className="space-y-1.5">
            <Label htmlFor="task-title">Title</Label>
            <Input id="task-title" value={title} onChange={e => setTitle(e.target.value)} />
          </div>

          {/* Message */}
          <div className="space-y-1.5">
            <Label htmlFor="task-message">Message</Label>
            <Textarea id="task-message" value={message} onChange={e => setMessage(e.target.value)} rows={6} />
          </div>

          {/* Item Image */}
          <div className="space-y-1.5">
            <Label>Item Image</Label>
            <div className="w-20 h-20 border rounded-lg overflow-hidden relative bg-gray-100">
              {item.productImageUrl ? (
                <Image src={item.productImageUrl} alt={item.productName} fill className="object-cover" sizes="80px" unoptimized />
              ) : (
                <div className="flex h-full w-full items-center justify-center text-gray-400 text-xs">No image</div>
              )}
            </div>
          </div>

          {/* Due Date */}
          <div className="space-y-1.5">
            <Label>Due Date</Label>
            <DatePicker value={dueDate} onValueChange={setDueDate} />
          </div>
        </div>

        <DialogFooter className="gap-2">
          <Button onClick={handleSubmit}>Send Request</Button>
          <Button variant="outline" className="border-red-300 text-red-600 hover:bg-red-50" onClick={() => onOpenChange(false)}>
            Cancel
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}
```

### 3. Export from index
In `apps/web/src/features/orders/components/order-detail/index.ts`, add:
```typescript
export { CreateTaskModal } from './create-task-modal';
export type { CreateTaskModalProps, CreateTaskData } from './create-task-modal';
```

## Todo List

- [ ] Create `generateTaskMessage` helper function
- [ ] Create `CreateTaskData` interface
- [ ] Create `CreateTaskModalProps` interface
- [ ] Create `CreateTaskModal` component with controlled open state
- [ ] Pre-fill title with `{productName} - Custom Design`
- [ ] Auto-generate message from order/item data
- [ ] Show product thumbnail image
- [ ] Add DatePicker defaulting to +7 days
- [ ] "Send Request" blue button with toast
- [ ] "Cancel" red outline button
- [ ] Reset form on open
- [ ] Export from `index.ts`
- [ ] Verify compile

## Success Criteria

- Modal opens with "Design Service" title
- Title pre-filled with product name
- Message auto-generated with order ID, product ID, meta, personalization
- Item image displayed
- Due date defaults to 7 days from now
- Send Request shows toast, closes modal
- Cancel closes modal
- Form resets when reopened
- Component under 200 lines

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Label component may not exist in shadcn setup | Build fail | Check if `@/components/ui/label` exists; if not, use plain `<label>` |
| DatePicker may not support time selection | UI mismatch with screenshot | Screenshot shows time; current DatePicker is date-only. Accept date-only for MVP |

## Security Considerations

- Message generated from mock data only; when real API, sanitize user inputs
- No file uploads in this modal

## Next Steps

- Phase 04 wires this modal to the "+ Create Task" button in OrderItemCard
