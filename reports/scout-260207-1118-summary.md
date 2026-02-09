# Scout Summary: Orders Feature Codebase Overview
**Date:** 2026-02-07 | **Location:** `/Users/taquanglinh/Documents/synkao/apps/web/src/`

## Quick Facts

- **Orders Module Location:** `/features/orders/`
- **Orders List Page:** `/app/(main)/orders/page.tsx`
- **Orders Detail Page:** `/app/(main)/orders/[id]/page.tsx`
- **Import Button Location:** `/app/(main)/orders/orders-page-header.tsx`
- **Feature Exports:** All via barrel exports in `/features/orders/index.ts`
- **UI Library:** Radix UI primitives wrapped in `/components/ui/`
- **State Management:** TanStack Query (server state) + Zustand (UI state)
- **Styling:** Tailwind CSS with arbitrary values for exact colors
- **Authentication:** JWT token in localStorage, Bearer token in headers

---

## Architecture Overview

```
Page Layer (Next.js App Router)
  └─ /app/(main)/orders/page.tsx
       ├── Imports components from /features/orders
       ├── Manages page-level state (pagination, filters, selection)
       ├── Uses useOrders() hook for data
       └── Renders OrdersPageHeader, OrderListTable, BulkActionBar

Feature Module (/features/orders/)
  ├── Types (types/) - Order, OrderItem, API response shapes
  ├── Lib (lib/) - API client, constants, utilities
  ├── Hooks (hooks/) - TanStack Query hooks
  └── Components (components/)
      ├── shared/ - Badge components, status displays
      ├── order-list/ - List page components (table, filters, dialogs)
      └── order-detail/ - Detail page components (cards, modals)

Shared Components (/components/)
  ├── ui/ - Primitives (Dialog, Button, FileUploader, etc)
  ├── layout/ - PageHeader, Sidebar, AppHeader
  └── auth/ - Guards, permission checks

State Management
  ├── Zustand stores (/stores/) - Auth, UI
  └── TanStack Query hooks - Data fetching & mutations

Services (/lib/)
  ├── auth-client.ts - Token management
  ├── query-client.ts - React Query config
  └── utils.ts - Helper functions
```

---

## Import Button Integration Point

**Current Implementation:**
```tsx
// File: /app/(main)/orders/orders-page-header.tsx
<Button variant="outline" onClick={() => setImportDialogOpen(true)}>
  <Upload className="mr-2 h-4 w-4" />
  Import
</Button>
<ImportOrdersDialog
  open={importDialogOpen}
  onOpenChange={setImportDialogOpen}
  onImport={handleImport}
/>
```

**Dialog Handler (placeholder):**
```tsx
const handleImport = (files: File[]) => {
  // TODO: Implement API integration for CSV import
  void files;
};
```

**Key Integration Points:**
1. Button in PageHeader actions
2. ImportOrdersDialog component (already created)
3. Handler function ready for API implementation
4. FileUploader component for drag-drop UX

---

## Dialog Patterns Used

| Pattern | Component | Use Case |
|---------|-----------|----------|
| Dialog | ImportOrdersDialog | CSV file upload |
| Dialog | DesignUploadModal | Design file upload |
| AlertDialog | CancelOrderDialog | Destructive action confirmation |
| Popover | StatusEditPopover | Inline status editing |
| Popover | NoteEditPopover | Inline note editing |

All dialogs follow:
- Local state for form data
- Reset on close
- Toast notifications for feedback
- Callback handlers for parent components
- Loading/disabled states for async operations

---

## File Upload Pipeline

1. **User clicks Import button** → Opens ImportOrdersDialog
2. **FileUploader renders** → Drag-drop + click interface
3. **File selected** → Stored in local useState
4. **User clicks Import** → Calls onImport handler
5. **Parent handles upload** → Makes API call (TODO)
6. **Toast notification** → Success/error feedback
7. **Dialog closes** → Files cleared from state

---

## API Integration Ready

**All endpoints defined in `orders-client.ts`:**

```
GET  /api/v1/orders/           List with filters
GET  /api/v1/orders/counts     Status counts for tabs
GET  /api/v1/orders/:id        Single order detail
GET  /api/v1/orders/:id/items  Order items (expandable rows)
POST /api/v1/orders/items/batch Batch items (N+1 solver)
GET  /api/v1/orders/:id/timeline Timeline events
PATCH /api/v1/orders/:id       Update notes
POST /api/v1/orders/:id/cancel Cancel order
```

**Ready to add:**
```
POST /api/v1/orders/import      CSV import endpoint
```

---

## Key Hooks Available

### Data Fetching
- `useOrders(params)` - List with pagination & filters
- `useOrder(id)` - Single order
- `useOrderItems(orderId)` - Items for order
- `useOrdersWithItems(params)` - Batch fetch (N+1 solver)
- `useOrderTimeline(orderId)` - Timeline events
- `useOrderCounts()` - Status counts with retry logic

### Mutations
- `useUpdateOrderNotes(orderId)` - Update notes
- `useCancelOrder(orderId)` - Cancel order

### Auth & Permissions
- `useAuth()` - Get user, loading, isAuthenticated
- `usePermissions()` - From Zustand store
- `useUser()` - From Zustand store

---

## Component Reusability

**Already implemented & reusable:**
- `PageHeader` - Title, breadcrumbs, action buttons
- `FileUploader` - Drag-drop + click, multiple files
- `Dialog` variants - StandardDialog, AlertDialog, etc
- `Button` - Multiple variants (outline, default, ghost)
- `Tabs` - For status filtering
- `Card` - For layout sections
- `Badge` - For status displays
- `Toast` (sonner) - Notifications

---

## Configuration & Constants

**All in `/features/orders/lib/constants.ts`:**
- Order status colors & labels (7 statuses)
- Design phase configs (4 statuses)
- Payment status configs (4 statuses)
- Timeline event styles (9 event types)
- Platform icons (4 platforms)
- Table column widths
- Design upload config (formats, max size)
- Design position options (11 positions)

---

## Development Checklist

To implement CSV import feature:

- [ ] Add POST `/api/v1/orders/import` endpoint (backend)
- [ ] Implement `importOrders` function in `/features/orders/lib/orders-client.ts`
- [ ] Create `useImportOrders` mutation hook in `/features/orders/hooks/use-orders.ts`
- [ ] Update `handleImport` in `orders-page-header.tsx` to call mutation
- [ ] Add loading state to Import button during upload
- [ ] Handle response with success toast
- [ ] Invalidate orders list query after import
- [ ] Add error handling with toast notifications
- [ ] (Optional) Add validation before sending (file type, size, etc)
- [ ] (Optional) Show import progress modal
- [ ] (Optional) Display import results/errors

---

## File Organization Summary

**40+ files organized in:**
1. `/app/(main)/orders/` - 4 page files
2. `/features/orders/` - 36 feature module files
   - types/ - 2 files
   - lib/ - 4 files
   - hooks/ - 3 files
   - components/ - 27 files (shared, list, detail)
3. `/components/` - Shared UI components
4. `/stores/` - Zustand stores
5. `/lib/` - Utilities, clients, constants

**All following barrel export pattern** for clean imports:
```tsx
import { 
  useOrders, 
  ImportOrdersDialog, 
  OrderListTable 
} from '@/features/orders'
```

---

## Tech Stack

- **Framework:** Next.js 14 (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS
- **UI Library:** Radix UI primitives
- **State:** TanStack Query + Zustand
- **Icons:** Lucide React
- **Notifications:** Sonner
- **Forms:** React Hook Form (if used)
- **HTTP:** Fetch API (no additional library)
- **Auth:** JWT in localStorage

---

## Performance Considerations

✅ **Already Implemented:**
- Lazy loading of order items (expand on demand)
- Batch fetching to avoid N+1 queries
- Query key factory for proper invalidation
- Stale time configuration (2 min for orders, 30 sec for counts)
- Retry logic with exponential backoff
- Pagination (10 items per page)
- Memoization of callbacks with useCallback
- File upload with drag-drop (no extra network calls)

---

## Next Steps for Implementation

1. **Review existing patterns** - All documented in scout report
2. **Implement import mutation** - 1 API function + 1 hook
3. **Update page handler** - Connect mutation to button click
4. **Test end-to-end** - File upload → API → List refresh
5. **Add success feedback** - Toast + list auto-refresh
6. **Handle errors** - Validation + error toast

---

## Questions to Answer Before Implementation

- Should import show progress modal or toast notification?
- Should duplicates be skipped or rejected?
- What CSV columns are expected?
- Should import validate before sending or on backend?
- Should successful import auto-refresh list?
- Are there permission checks needed for import?

**All answers can be found in existing patterns already documented.**

