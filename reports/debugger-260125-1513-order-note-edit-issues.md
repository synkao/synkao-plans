# Debug Report: Order List Note Editing Issues

**Date:** 2026-01-25 15:13
**Reporter:** Debugger Agent
**Issues:** Space key navigation & Save button not calling API

---

## Executive Summary

**Impact:** Medium - Users cannot edit order notes properly

**Root Causes Identified:**
1. **Issue 1 (Space Navigation):** Keyboard events bubble from textarea through popover to parent row handler
2. **Issue 2 (No API Call):** Handler exists but contains placeholder TODO - no actual mutation implemented

**Status:** Both issues confirmed, fixes required in different layers

---

## Issue 1: Space Key Navigates to Detail Page

### Symptoms
- Pressing space in note textarea triggers navigation to order detail page
- Expected: Space character should be typed into textarea

### Root Cause Analysis

**Event Flow:**
```
User presses Space in Textarea
  ↓
Textarea (no onKeyDown handler)
  ↓
PopoverContent (no event stopPropagation)
  ↓
OrderRow tr element (onKeyDown handler at line 86-91)
  ↓
Triggers onClick -> navigates to detail page
```

**Evidence from Code:**

1. **OrderRow.tsx:86-91** - Row has keyboard handler:
```tsx
const handleKeyDown = (e: React.KeyboardEvent) => {
  if (e.key === 'Enter' || e.key === ' ') {
    e.preventDefault();
    onClick?.(order);  // Navigates to detail
  }
};
```

2. **OrderRow.tsx:105** - Handler attached to `<tr>`:
```tsx
<tr onKeyDown={onClick ? handleKeyDown : undefined}>
```

3. **OrderRow.tsx:219** - Note cell has click stopPropagation but NOT keyboard:
```tsx
<td className="px-4 py-3" onClick={(e) => e.stopPropagation()}>
```

4. **NoteEditPopover.tsx:86-93** - Textarea has NO keyboard event handling:
```tsx
<Textarea
  value={note}
  onChange={(e) => setNote(e.target.value)}
  // Missing: onKeyDown handler
/>
```

5. **Textarea.tsx:5-16** - Component spreads props without special handling:
```tsx
function Textarea({ className, ...props }: React.ComponentProps<"textarea">) {
  return <textarea {...props} />  // Props spread, no default handlers
}
```

### Event Bubbling Gap

- Mouse clicks: Stopped at cell level (`onClick={(e) => e.stopPropagation()}`)
- Keyboard events: NOT stopped - bubble to row handler
- Result: Space/Enter in textarea triggers row navigation

---

## Issue 2: Save Button Doesn't Call API

### Symptoms
- Click Save shows toast "Note updated for order..."
- No network request sent
- Note not persisted to backend

### Root Cause Analysis

**Call Chain:**
```
NoteEditPopover.handleSave (line 48-54)
  ↓
onSave prop (line 51)
  ↓
OrderRow.onNoteChange prop
  ↓
page.tsx handleNoteChange (line 101-105)
  ↓
TODO placeholder - no mutation
```

**Evidence from Code:**

1. **NoteEditPopover.tsx:48-54** - Save logic exists:
```tsx
const handleSave = () => {
  if (!isOverLimit && hasChanges) {
    const sanitizedNote = sanitizeNote(note);
    onSave(orderId, sanitizedNote);  // Calls parent handler
    setOpen(false);
  }
};
```

2. **page.tsx:101-105** - Handler is placeholder:
```tsx
const handleNoteChange = (orderId: string, newNote: string) => {
  // TODO: Implement mutation when backend API is ready
  toast.success(`Note updated for order ${orderId.slice(0, 8)}...`);
  void newNote; // Placeholder until API integration
};
```

### Missing Implementation

- No mutation hook call (e.g., `useUpdateOrderNote`)
- No API endpoint invocation
- No error handling
- No optimistic update
- No cache invalidation

---

## Technical Analysis

### Issue 1 Event Flow Diagram

```
┌─────────────────────┐
│  User: Press Space  │
└──────────┬──────────┘
           │
           ▼
┌──────────────────────────┐
│  Textarea Component      │
│  (no onKeyDown)          │
└──────────┬───────────────┘
           │ Event bubbles
           ▼
┌──────────────────────────┐
│  PopoverContent          │
│  (no stopPropagation)    │
└──────────┬───────────────┘
           │ Event bubbles
           ▼
┌──────────────────────────┐
│  TD Cell                 │
│  (onClick stopped only)  │
└──────────┬───────────────┘
           │ Event bubbles
           ▼
┌──────────────────────────┐
│  TR (OrderRow)           │
│  onKeyDown handler       │
│  ✓ Matches Space key     │
│  ✓ Calls onClick(order)  │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│  Navigation to detail    │
└──────────────────────────┘
```

### Issue 2 Missing Pieces

**Current State:**
- UI layer: ✓ Complete (popover, textarea, buttons, validation)
- Event handling: ✓ Complete (onChange, onClick, state management)
- Data layer: ✗ Missing (no mutation, no API call)

**What's Needed:**
- Mutation hook (e.g., TanStack Query `useMutation`)
- API endpoint (`PATCH /api/orders/{id}/note`)
- Error handling & retry logic
- Optimistic UI updates
- Cache invalidation

---

## Recommended Fixes

### Fix 1: Stop Keyboard Event Propagation

**Location:** `apps/web/src/features/orders/components/order-list/note-edit-popover.tsx:86-93`

**Strategy:** Add `onKeyDown` handler to Textarea to prevent Space/Enter bubbling

**Implementation:**
```tsx
<Textarea
  value={note}
  onChange={(e) => setNote(e.target.value)}
  onKeyDown={(e) => {
    // Stop Space/Enter from bubbling to parent row
    if (e.key === ' ' || e.key === 'Enter') {
      e.stopPropagation();
    }
  }}
  placeholder="Add a note..."
  className="min-h-[80px] resize-none"
  maxLength={MAX_LENGTH + 10}
  disabled={isUpdating}
/>
```

**Alternative Strategy:** Stop at PopoverContent level
```tsx
<PopoverContent
  className="w-80 p-3"
  align="start"
  onKeyDown={(e) => e.stopPropagation()}  // Stop all keyboard events
>
```

**Recommendation:** Use Textarea-level fix for precision (allows Escape to close popover)

---

### Fix 2: Implement API Mutation

**Location:** `apps/web/src/app/(main)/orders/page.tsx:101-105`

**Strategy:** Replace TODO with actual mutation hook

**Prerequisites:**
- API endpoint: `PATCH /api/orders/:id/note`
- Mutation hook: Create `useUpdateOrderNote` in orders feature

**Implementation Example:**
```tsx
// In features/orders/hooks/use-update-order-note.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { api } from '@/lib/api';

export function useUpdateOrderNote() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async ({ orderId, note }: { orderId: string; note: string }) => {
      const response = await api.patch(`/orders/${orderId}/note`, { note });
      return response.data;
    },
    onSuccess: (data, variables) => {
      // Invalidate orders list query
      queryClient.invalidateQueries({ queryKey: ['orders'] });
      toast.success(`Note updated successfully`);
    },
    onError: (error) => {
      toast.error(`Failed to update note: ${error.message}`);
    },
  });
}

// In page.tsx
const updateNoteMutation = useUpdateOrderNote();

const handleNoteChange = (orderId: string, newNote: string) => {
  updateNoteMutation.mutate({ orderId, note: newNote });
};

// Pass mutation state to table
<OrderListTable
  onNoteChange={handleNoteChange}
  isNoteUpdating={(orderId) =>
    updateNoteMutation.isPending &&
    updateNoteMutation.variables?.orderId === orderId
  }
/>
```

**Additional Changes Needed:**
- NoteEditPopover: Accept `isUpdating` per order (not global)
- API client: Add PATCH endpoint
- Error boundaries: Handle mutation failures

---

## Risk Assessment

### Issue 1 Fix Risk: **Low**
- Change scope: Single component
- Breaking potential: None (defensive change)
- Test coverage: Manual testing sufficient
- Rollback: Simple revert

### Issue 2 Fix Risk: **Medium**
- Change scope: Multiple layers (API, hook, component)
- Breaking potential: Network errors, rate limits
- Test coverage: Unit + integration tests required
- Rollback: May need database migration rollback

---

## Success Criteria

### Issue 1 Resolution
- [ ] Space key types space character in textarea
- [ ] Enter key adds newline in textarea
- [ ] Escape key still closes popover
- [ ] Click outside still closes popover
- [ ] Row keyboard navigation still works when NOT editing

### Issue 2 Resolution
- [ ] Save button sends PATCH request to backend
- [ ] Success toast shows after API confirms
- [ ] Order list refreshes with new note
- [ ] Error toast shows on failure
- [ ] Optimistic update shows immediate feedback
- [ ] Loading state disables UI during mutation

---

## Testing Plan

### Manual Testing

**Issue 1:**
1. Open note edit popover
2. Click in textarea
3. Press Space → verify space character appears (not navigation)
4. Press Enter → verify newline appears (not navigation)
5. Press Escape → verify popover closes
6. Focus row (not editing) → press Space → verify navigation works

**Issue 2:**
1. Edit note text
2. Click Save
3. Open DevTools Network tab → verify PATCH request sent
4. Verify response 200 OK
5. Close popover → verify note updated in list
6. Refresh page → verify note persisted
7. Test error case: Disconnect network → verify error toast

### Automated Testing

```tsx
// note-edit-popover.test.tsx
describe('NoteEditPopover keyboard events', () => {
  it('should type space in textarea without triggering row click', () => {
    const onRowClick = jest.fn();
    render(<OrderRow onClick={onRowClick} />);

    const textarea = screen.getByRole('textbox');
    fireEvent.keyDown(textarea, { key: ' ' });

    expect(onRowClick).not.toHaveBeenCalled();
  });
});

// page.test.tsx
describe('handleNoteChange', () => {
  it('should call API mutation on save', async () => {
    const { result } = renderHook(() => useUpdateOrderNote());

    act(() => {
      result.current.mutate({ orderId: '123', note: 'test' });
    });

    await waitFor(() => {
      expect(mockApi.patch).toHaveBeenCalledWith('/orders/123/note', { note: 'test' });
    });
  });
});
```

---

## Supporting Evidence

### File Locations
- Popover: `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/components/order-list/note-edit-popover.tsx`
- Row: `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/components/order-list/order-row.tsx`
- Page: `/Users/taquanglinh/Documents/synkao/apps/web/src/app/(main)/orders/page.tsx`

### Code Excerpts

**OrderRow keyboard handler (line 86-91):**
```tsx
const handleKeyDown = (e: React.KeyboardEvent) => {
  if (e.key === 'Enter' || e.key === ' ') {
    e.preventDefault();
    onClick?.(order);
  }
};
```

**Page handler placeholder (line 101-105):**
```tsx
const handleNoteChange = (orderId: string, newNote: string) => {
  // TODO: Implement mutation when backend API is ready
  toast.success(`Note updated for order ${orderId.slice(0, 8)}...`);
  void newNote; // Placeholder until API integration
};
```

---

## Next Steps

### Immediate Actions
1. Implement Fix 1 (keyboard event handling) - **15 min effort**
2. Test Fix 1 manually
3. Coordinate with backend team on API endpoint spec

### Follow-up Actions
4. Backend: Implement `PATCH /api/orders/:id/note` endpoint
5. Frontend: Create `useUpdateOrderNote` mutation hook
6. Frontend: Update `handleNoteChange` in page.tsx
7. Test Fix 2 end-to-end
8. Add automated tests for both fixes

### Timeline
- Fix 1: Ready to implement now
- Fix 2: Blocked on backend API (estimate: 1-2 days backend + 2-3 hours frontend)

---

## Unresolved Questions

1. **API Spec:** What's the exact request/response format for `PATCH /api/orders/:id/note`?
2. **Permissions:** Do all users have permission to edit notes, or is role-based auth required?
3. **Validation:** Is 255 char limit enforced on backend, or different limit?
4. **Rate Limiting:** Should we debounce/throttle note updates to prevent abuse?
5. **Optimistic Updates:** Should we show optimistic UI before API confirms, or wait for response?
6. **Error Recovery:** If mutation fails, should we retry automatically or require user action?
