# Scout Report: Orders Feature Structure & Patterns
**Date:** 2026-02-07 | **Time:** 11:18 UTC

## 1. Orders List Page Structure

### Page Location & Entry Point
- **Main page:** `/Users/taquanglinh/Documents/synkao/apps/web/src/app/(main)/orders/page.tsx`
- **Page header:** `/Users/taquanglinh/Documents/synkao/apps/web/src/app/(main)/orders/orders-page-header.tsx`
- **Route structure:** Uses Next.js app router with `(main)` layout group for authenticated pages

### Orders Page Header - Where Import Button Lives
Current location in `orders-page-header.tsx`:
```tsx
<Button variant="outline" onClick={() => setImportDialogOpen(true)}>
  <Upload className="mr-2 h-4 w-4" />
  Import
</Button>
```

**Location in PageHeader component:**
- Import button placed in the `actions` prop of `PageHeader` component
- Alongside "New Order" button
- Both buttons rendered in a flex container with gap-2
- **Path:** `/Users/taquanglinh/Documents/synkao/apps/web/src/components/layout/page-header.tsx`

### Main Page Layout Flow
1. `OrdersPageHeader` - Contains title, breadcrumbs, and action buttons
2. `OrderStatusTabs` - Filter by order status with count badges
3. `OrderListFilters` - Search and additional filter options
4. `OrderListTable` - Main data table with selectable rows and collapsible item expansion
5. `OrdersPagination` - Pagination controls
6. `BulkActionBar` - Floating action bar when orders selected (fixed bottom-6)

---

## 2. Existing Dialog/Modal Patterns

### Pattern 1: ImportOrdersDialog (CSV Upload)
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/components/order-list/import-orders-dialog.tsx`

```tsx
interface ImportOrdersDialogProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  onImport?: (files: File[]) => void;
}
```

**Key characteristics:**
- Uses Dialog component from Radix UI (via `@/components/ui/dialog`)
- Accepts single CSV file (maxFiles={1})
- Max size: 10MB
- File type: `.csv` only
- Local state for files with reset on close
- DialogFooter with Cancel/Import buttons
- FileUploader component with drag-drop support

### Pattern 2: DesignUploadModal
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/components/order-detail/design-upload-modal.tsx`

```tsx
interface DesignUploadModalProps {
  itemId: string;
  itemName: string;
  trigger: React.ReactNode;
  onUploadComplete?: (file: File) => void;
}
```

**Key characteristics:**
- Uses Dialog with DialogTrigger pattern (child component receives trigger button)
- Controlled open state (useState)
- File validation with custom validation function
- Supports: PNG, JPG, PDF, AI, PSD (max 50MB)
- File type validation by MIME type AND extension
- Upload state tracking (isUploading)
- Toast notifications for errors and success
- showPreview={true} to display file preview

### Pattern 3: CancelOrderDialog (AlertDialog)
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/components/order-detail/cancel-order-dialog.tsx`

```tsx
interface CancelOrderDialogProps {
  orderId: string;
  orderNumber: string;
  open: boolean;
  onOpenChange: (open: boolean) => void;
}
```

**Key characteristics:**
- Uses AlertDialog (for destructive actions)
- Optional reason textarea (max 200 chars)
- Integrates with mutation hook (useCancelOrder)
- Handles pending state on submit button
- Toast notifications for success/error
- Redirects to /orders after success
- Red styling for destructive action

### Pattern 4: EditNotesDialog
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/components/order-detail/edit-notes-dialog.tsx`

Usage pattern: Dialog with state management for editable content

### Common Dialog Pattern Summary
```
Dialog Pattern (3 variants):
1. Dialog (regular) - Imports, uploads, forms
2. AlertDialog - Destructive actions, confirmations
3. Popover - Inline edits (used in some cells)

Standard structure:
- DialogHeader (icon + title + description)
- Content (form/input/uploader)
- DialogFooter (Cancel + Action buttons)
- State managed locally via useState
- Callback handlers (onOpenChange, onImport/onUpload)
- Toast notifications (sonner)
```

---

## 3. File Upload Patterns

### FileUploader Component (Reusable)
**Path:** `/Users/taquanglinh/Documents/synkao/apps/web/src/components/ui/file-uploader.tsx`

```tsx
interface FileUploaderProps {
  value?: File[]
  onValueChange?: (files: File[]) => void
  accept?: Accept  // From react-dropzone
  maxFiles?: number
  maxSize?: number  // bytes
  showPreview?: boolean
  disabled?: boolean
  className?: string
}
```

**Features:**
- Built on `react-dropzone` library
- Drag & drop + click to upload
- File validation with rejection messages
- Dynamic file size formatting (Bytes, KB, MB, GB)
- Optional image preview with thumbnail
- File list display with remove buttons
- Used in both ImportOrdersDialog and DesignUploadModal

### Upload Pattern Implementation
**File type configuration example:**
```tsx
// From constants.ts - DESIGN_UPLOAD_CONFIG
export const DESIGN_UPLOAD_CONFIG = {
  maxSize: 50 * 1024 * 1024,  // 50MB
  accept: {
    'image/png': ['.png'],
    'image/jpeg': ['.jpg', '.jpeg'],
    'application/pdf': ['.pdf'],
    // Multiple MIME types for AI/PSD (cross-platform)
    'application/postscript': ['.ai'],
    'image/vnd.adobe.photoshop': ['.psd'],
    // ... more formats
  },
};
```

**Validation pattern:**
1. File type check by MIME type
2. File extension fallback check
3. File size validation
4. Toast error for invalid files
5. isUploading state for button disabled state

---

## 4. Feature Module Organization

### Directory Structure
```
/features/orders/
├── types/
│   ├── index.ts (barrel export)
│   └── order.types.ts (extended types + API types)
├── lib/
│   ├── orders-client.ts (HTTP functions - GET/POST/PATCH)
│   ├── constants.ts (status configs, column widths)
│   ├── utils.ts
│   └── index.ts (barrel export)
├── hooks/
│   ├── use-orders.ts (TanStack Query hooks)
│   ├── use-order-counts.ts
│   └── index.ts (barrel export)
├── components/
│   ├── shared/ (badges, status components)
│   │   ├── order-status-badge.tsx
│   │   ├── design-phase-badge.tsx
│   │   ├── item-design-status-badge.tsx
│   │   └── fulfiller-cell.tsx
│   ├── order-list/ (list page components)
│   │   ├── import-orders-dialog.tsx
│   │   ├── order-list-table.tsx
│   │   ├── order-list-filters.tsx
│   │   ├── bulk-action-bar.tsx
│   │   ├── order-status-tabs.tsx
│   │   ├── status-edit-popover.tsx
│   │   ├── note-edit-popover.tsx
│   │   ├── image-lightbox-dialog.tsx
│   │   ├── order-row.tsx
│   │   ├── order-item-row.tsx
│   │   ├── design-status-cell.tsx
│   │   ├── task-status-cell.tsx
│   │   ├── platform-icon.tsx
│   │   └── index.ts (barrel export)
│   ├── order-detail/ (detail page components)
│   │   ├── design-upload-modal.tsx
│   │   ├── cancel-order-dialog.tsx
│   │   ├── edit-notes-dialog.tsx
│   │   ├── order-status-dropdown.tsx
│   │   ├── order-timeline.tsx
│   │   ├── order-status-stepper.tsx
│   │   ├── order-summary-card.tsx
│   │   ├── customer-info-card.tsx
│   │   ├── order-metadata-card.tsx
│   │   ├── order-item-card.tsx
│   │   ├── order-notes-section.tsx
│   │   └── index.ts (barrel export)
└── index.ts (main barrel export)
```

### Barrel Export Pattern
Each module level exports all public APIs:
- `/features/orders/index.ts` exports types, lib, hooks, components
- `/features/orders/components/order-list/index.ts` exports all list components
- Components imported as: `import { OrderListTable, ImportOrdersDialog } from '@/features/orders'`

---

## 5. State Management Patterns

### TanStack Query (React Query) - Data Fetching
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/hooks/use-orders.ts`

**Query Key Factory Pattern:**
```tsx
export const orderKeys = {
  all: ['orders'] as const,
  lists: () => [...orderKeys.all, 'list'] as const,
  list: (filters: OrderListParams) => [...orderKeys.lists(), filters] as const,
  details: () => [...orderKeys.all, 'detail'] as const,
  detail: (id: string) => [...orderKeys.details(), id] as const,
  items: (orderId: string) => [...orderKeys.all, 'items', orderId] as const,
  itemsBatch: (orderIds: string[]) => [...orderKeys.all, 'items-batch', orderIds] as const,
  timeline: (orderId: string) => [...orderKeys.all, 'timeline', orderId] as const,
};
```

**Key hooks provided:**
1. `useOrders(params)` - Fetch paginated list with filters
   - staleTime: 2 minutes
   - Supports page, limit, status, design_status, search, date ranges

2. `useOrder(id)` - Fetch single order
   - enabled based on ID presence
   - staleTime: 2 minutes

3. `useOrderItems(orderId, options)` - Lazy-load items for expanded rows

4. `useOrdersWithItems(params)` - N+1 problem solver
   - Combines useOrders + batch fetch
   - Single request gets all items instead of N separate requests

5. `useUpdateOrderNotes(orderId)` - Mutation for notes updates
   - Invalidates detail and timeline queries on success

6. `useCancelOrder(orderId)` - Mutation for cancellation
   - Invalidates detail, timeline, and lists on success

7. `useOrderTimeline(orderId)` - Fetch timeline events

### Zustand Stores - UI & Auth State
**Auth Store:** `/Users/taquanglinh/Documents/synkao/apps/web/src/stores/auth.store.ts`
```tsx
interface AuthState {
  user: User | null;
  permissions: string[];
  isAuthenticated: boolean;
  setUser: (user: User | null) => void;
  setPermissions: (permissions: string[]) => void;
  clearAuth: () => void;
}
```

**UI Store:** `/Users/taquanglinh/Documents/synkao/apps/web/src/stores/ui.store.ts`
```tsx
interface UIState {
  sidebarOpen: boolean;
  sidebarCollapsed: boolean;
  toggleSidebar: () => void;
  toggleSidebarCollapsed: () => void;
  setSidebarOpen: (open: boolean) => void;
}
```

**Pattern:** 
- Zustand for lightweight UI state (sidebar, modals if needed)
- TanStack Query for server state (orders data)
- Selector functions for optimized re-renders

### Local Component State
- Dialog open/close: `useState(false)`
- Form values: `useState()`
- Selection: `useState(new Set())`
- Expansion state: `useState(new Set())`

---

## 6. API Integration Patterns

### API Client Pattern
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/lib/orders-client.ts`

**Core pattern:**
```tsx
const API_BASE = '/api/v1';

export async function getOrders(params: OrderListParams = {}): Promise<OrderListResponse> {
  const response = await fetch(`${API_BASE}/orders/${buildQueryString(params)}`);
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Failed to fetch orders');
  }
  
  return response.json();
}
```

**Available endpoints:**
- GET `/api/v1/orders/` - List with filters
- GET `/api/v1/orders/:id` - Single order detail
- GET `/api/v1/orders/:id/items` - Order items
- POST `/api/v1/orders/items/batch` - Batch items fetch
- GET `/api/v1/orders/:id/timeline` - Timeline events
- PATCH `/api/v1/orders/:id` - Update notes
- POST `/api/v1/orders/:id/cancel` - Cancel order
- GET `/api/v1/orders/counts` - Status counts

### Response Format
```tsx
// List response
{
  data: MockOrder[],
  pagination: {
    page: number,
    limit: number,
    total: number,
    totalPages: number
  }
}

// Single resource response
{
  data: MockOrder  // or specific type
}
```

### Error Handling
- Try/catch in mutations with toast.error()
- Query errors logged and exposed via hook
- Graceful degradation with placeholderData for counts
- Retry logic with 2 retries and 1s delay for counts

### Loading states in components:
```tsx
if (isLoading) {
  return <LoadingUI />;
}
if (error) {
  return <ErrorUI error={error.message} />;
}
return <SuccessUI />;
```

---

## 7. Permission & Role Checking Patterns

### Admin Guard Component
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/components/auth/admin-guard.tsx`

```tsx
export function AdminGuard({ children }: AdminGuardProps) {
  const { user, isLoading, isAuthenticated } = useAuth();

  useEffect(() => {
    if (isLoading) return;

    // Not authenticated - redirect to login
    if (!hasToken() && !isAuthenticated) {
      router.replace('/login?callbackUrl=/admin');
      return;
    }

    // Authenticated but not admin - redirect to main app
    if (user && user.role !== 'ADMIN') {
      router.replace('/');
    }
  }, [isLoading, isAuthenticated, user, router]);

  // Loading state - show spinner
  if (isLoading || (!isAuthenticated && hasToken())) {
    return <LoadingSpinner />;
  }

  // Not authenticated - will redirect
  if (!isAuthenticated && !hasToken()) {
    return null;
  }

  // Not admin - will redirect
  if (user && user.role !== 'ADMIN') {
    return <LoadingSpinner />;
  }

  return <>{children}</>;
}
```

### useAuth Hook Pattern
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/hooks/use-auth.ts`

Returns:
- `user: User | null`
- `isLoading: boolean`
- `isAuthenticated: boolean`

### Permission Check Pattern
```tsx
// In store
permissions: string[]  // List of permission strings

// Usage would be:
if (permissions.includes('orders:import')) {
  // Show import button
}
```

### Token Management
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/lib/auth-client.ts`

```tsx
export function getToken(): string | null {
  if (typeof window === 'undefined') return null;
  return localStorage.getItem(AUTH_TOKEN_KEY);
}

export function hasToken(): boolean {
  return !!getToken();
}

// Auth headers include token
function getAuthHeaders(): HeadersInit {
  const token = getToken();
  return {
    'Content-Type': 'application/json',
    ...(token && { Authorization: `Bearer ${token}` }),
  };
}
```

---

## 8. Component Integration Examples

### Using Multiple Patterns Together

**Orders Page Example:**
```tsx
'use client';

// 1. Page-level state
const [selectedOrderIds, setSelectedOrderIds] = useState<Set<string>>();

// 2. TanStack Query hook for data
const { data, isLoading } = useOrders({ page, limit, search });

// 3. Dialog state
const [importDialogOpen, setImportDialogOpen] = useState(false);

// 4. Handler functions
const handleImport = (files: File[]) => {
  // API call would go here
  toast.success('Orders imported');
};

// 5. JSX with components
<OrdersPageHeader />
<OrderListTable 
  orders={data?.data}
  onSelectionChange={setSelectedOrderIds}
/>
<BulkActionBar 
  selectedCount={selectedOrderIds.size}
/>
<ImportOrdersDialog 
  open={importDialogOpen}
  onOpenChange={setImportDialogOpen}
  onImport={handleImport}
/>
```

---

## 9. Key File Paths Summary

### Core Files
| Path | Purpose |
|------|---------|
| `/app/(main)/orders/page.tsx` | Orders list page component |
| `/app/(main)/orders/orders-page-header.tsx` | Page header with Import button |
| `/app/(main)/orders/[id]/page.tsx` | Order detail page |
| `/features/orders/components/order-list/import-orders-dialog.tsx` | CSV import dialog |
| `/features/orders/lib/orders-client.ts` | API client functions |
| `/features/orders/hooks/use-orders.ts` | Query/mutation hooks |

### Shared UI Components
| Path | Purpose |
|------|---------|
| `/components/ui/file-uploader.tsx` | Drag-drop file upload component |
| `/components/ui/dialog.tsx` | Dialog primitive from Radix UI |
| `/components/ui/alert-dialog.tsx` | AlertDialog for destructive actions |
| `/components/layout/page-header.tsx` | Reusable page header with breadcrumbs |
| `/components/auth/admin-guard.tsx` | Role-based access control |

### Stores
| Path | Purpose |
|------|---------|
| `/stores/auth.store.ts` | Authentication state & permissions |
| `/stores/ui.store.ts` | UI state (sidebar, etc) |

---

## 10. Configuration & Constants

**Status display configurations** in `/features/orders/lib/constants.ts`:
- ORDER_STATUS_CONFIG - Colors and labels
- DESIGN_PHASE_CONFIG - Design phase statuses
- ITEM_DESIGN_STATUS_CONFIG - Item-level design statuses
- PAYMENT_STATUS_CONFIG - Payment statuses
- ORDER_TIMELINE_EVENT_CONFIG - Timeline event styling
- PLATFORM_CONFIG - Platform badges (Shopify, WooCommerce, Etsy, Manual)
- TABLE_COLUMN_WIDTHS - Tailwind classes for column sizing
- DESIGN_UPLOAD_CONFIG - File upload settings
- DESIGN_POSITION_OPTIONS - 11 placement options (Front, Back, Sleeve, etc)

---

## Unresolved Questions
None identified at this time. Codebase structure is well-organized and patterns are clear.

