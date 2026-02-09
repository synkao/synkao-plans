# Scout Report: Orders Feature Analysis
**Date:** 2026-02-07 | **Status:** Complete | **Coverage:** Orders feature codebase

## Executive Summary

Comprehensive analysis of the Synkao orders management feature reveals a well-structured, modular architecture using Next.js 14, React Query, Zustand, and shadcn/ui. The Import button is already implemented in the page header. The codebase demonstrates strong patterns for dialogs, file uploads, state management, and permission checking.

---

## 1. Orders List Page Structure & Import Button Location

**Key File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/app/(main)/orders/orders-page-header.tsx`

### Current Structure
The orders page is organized into a clean hierarchy:

```
OrdersPage (page.tsx)
├── OrdersPageHeader (orders-page-header.tsx) ← Import button location
│   ├── PageHeader component
│   ├── Import button (Upload icon)
│   ├── New Order button (Plus icon)
│   └── ImportOrdersDialog
├── OrderStatusTabs
├── OrderListFilters
├── OrderListTable
│   ├── OrderRow (collapsible)
│   └── OrderItemRow (lazy-loaded items)
├── OrdersPagination
└── BulkActionBar (floating bottom bar)
```

### Import Button Implementation
**Location:** Already exists in `OrdersPageHeader` component  
**Pattern:** Uses controlled dialog state with callback pattern

```typescript
// In orders-page-header.tsx
const [importDialogOpen, setImportDialogOpen] = useState(false);

const handleImport = (files: File[]) => {
  // TODO: Implement API integration for CSV import
  void files;
};

return (
  <Button 
    variant="outline" 
    onClick={() => setImportDialogOpen(true)}
  >
    <Upload className="mr-2 h-4 w-4" />
    Import
  </Button>
);
```

### Page Header Component Pattern
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/components/layout/page-header.tsx`

```typescript
interface PageHeaderProps {
  title: string;
  description?: string;
  breadcrumbs?: BreadcrumbItem[];
  actions?: React.ReactNode; // For buttons
}
```

---

## 2. Dialog & Modal Patterns

### Pattern 1: Import Orders Dialog (File Upload)
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/components/order-list/import-orders-dialog.tsx`

```typescript
// Props pattern
interface ImportOrdersDialogProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  onImport?: (files: File[]) => void;
}

// Features:
// - Controlled via open/onOpenChange props
// - FileUploader component with drag-drop
// - CSV-only file type restriction
// - Max 10MB file size
// - State reset on close
```

### Pattern 2: Design Upload Modal (Similar but with file preview)
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/components/order-detail/design-upload-modal.tsx`

```typescript
// Uses DialogTrigger pattern instead of controlled state
<Dialog open={open} onOpenChange={handleOpenChange}>
  <DialogTrigger asChild>{trigger}</DialogTrigger>
  <DialogContent>
    {/* Content */}
  </DialogContent>
</Dialog>

// Supports: PNG, JPG, PDF, AI, PSD
// Max 50MB, shows file preview
```

### Pattern 3: Confirmation Dialog (AlertDialog)
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/components/order-detail/cancel-order-dialog.tsx`

```typescript
// Uses AlertDialog for destructive actions
<AlertDialog open={open} onOpenChange={handleOpenChange}>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle className="text-red-600">Cancel Order</AlertDialogTitle>
      <AlertDialogDescription>
        Are you sure you want to cancel order?
      </AlertDialogDescription>
    </AlertDialogHeader>
    {/* Optional textarea for reason */}
    <AlertDialogFooter>
      <AlertDialogCancel>Go Back</AlertDialogCancel>
      <AlertDialogAction className="bg-red-600">Cancel Order</AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```

### Pattern 4: Edit Dialog with Mutation
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/components/order-detail/edit-notes-dialog.tsx`

```typescript
// Integrates with React Query mutations
const [notes, setNotes] = useState(currentNotes ?? '');
const { mutate: updateNotes, isPending } = useUpdateOrderNotes(orderId);

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
```

### Dialog Component Imports
```typescript
// Standard Dialog
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogFooter,
  DialogTrigger,
} from '@/components/ui/dialog';

// Alert Dialog (destructive actions)
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
```

---

## 3. File Upload Patterns

**Component:** `/Users/taquanglinh/Documents/synkao/apps/web/src/components/ui/file-uploader.tsx`

### FileUploader Props
```typescript
interface FileUploaderProps {
  value?: File[];
  onValueChange?: (files: File[]) => void;
  accept?: Accept; // react-dropzone format
  maxFiles?: number;
  maxSize?: number; // bytes
  showPreview?: boolean;
  disabled?: boolean;
  className?: string;
}
```

### Usage Examples
```typescript
// CSV import (ImportOrdersDialog)
<FileUploader
  value={files}
  onValueChange={setFiles}
  accept={{ 'text/csv': ['.csv'] }}
  maxFiles={1}
  maxSize={10 * 1024 * 1024}
  showPreview={false}
/>

// Design files (DesignUploadModal)
<FileUploader
  value={files}
  onValueChange={setFiles}
  accept={DESIGN_UPLOAD_CONFIG.accept}
  maxFiles={1}
  maxSize={DESIGN_UPLOAD_CONFIG.maxSize}
  showPreview={true}
/>
```

### File Type Configuration
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/lib/constants.ts`

```typescript
export const DESIGN_UPLOAD_CONFIG = {
  maxSize: 50 * 1024 * 1024, // 50MB
  accept: {
    'image/png': ['.png'],
    'image/jpeg': ['.jpg', '.jpeg'],
    'application/pdf': ['.pdf'],
    'application/postscript': ['.ai'],
    'application/illustrator': ['.ai'],
    'image/vnd.adobe.photoshop': ['.psd'],
    // Fallback for unrecognized MIME types
    'application/octet-stream': ['.ai', '.psd'],
  },
};
```

---

## 4. Feature Module Structure

### Orders Feature Organization
```
src/features/orders/
├── components/
│   ├── order-list/
│   │   ├── bulk-action-bar.tsx
│   │   ├── design-status-cell.tsx
│   │   ├── image-lightbox-dialog.tsx
│   │   ├── import-orders-dialog.tsx ← CSV import
│   │   ├── note-edit-popover.tsx
│   │   ├── order-item-row.tsx
│   │   ├── order-list-filters.tsx
│   │   ├── order-list-table.tsx
│   │   ├── order-row.tsx
│   │   ├── order-status-tabs.tsx
│   │   ├── platform-icon.tsx
│   │   ├── status-edit-popover.tsx
│   │   ├── task-status-cell.tsx
│   │   └── index.ts
│   ├── order-detail/
│   │   ├── cancel-order-dialog.tsx
│   │   ├── customer-info-card.tsx
│   │   ├── design-upload-modal.tsx
│   │   ├── edit-notes-dialog.tsx
│   │   ├── order-item-card.tsx
│   │   ├── order-metadata-card.tsx
│   │   ├── order-notes-section.tsx
│   │   ├── order-status-dropdown.tsx
│   │   ├── order-status-stepper.tsx
│   │   ├── order-summary-card.tsx
│   │   ├── order-timeline.tsx
│   │   └── index.ts
│   ├── shared/
│   │   ├── design-phase-badge.tsx
│   │   ├── fulfiller-cell.tsx
│   │   ├── item-design-status-badge.tsx
│   │   ├── order-status-badge.tsx
│   │   └── index.ts
│   └── index.ts
├── hooks/
│   ├── use-order-counts.ts
│   ├── use-orders.ts (React Query hooks)
│   └── index.ts
├── lib/
│   ├── constants.ts (Status configs, colors, options)
│   ├── orders-client.ts (API calls)
│   ├── utils.ts (Utility functions)
│   └── index.ts
├── types/
│   ├── order.types.ts
│   └── index.ts
└── index.ts (Main exports)
```

### Module Export Pattern
**File:** `src/features/orders/index.ts`

```typescript
// Types
export * from './types';

// Lib
export * from './lib';

// Hooks
export * from './hooks';

// Components - Shared
export * from './components/shared';

// Components - Order List
export * from './components/order-list';

// Components - Order Detail
export * from './components/order-detail';
```

This allows clean imports:
```typescript
import {
  useOrders,
  OrderListTable,
  ImportOrdersDialog,
  type MockOrder,
} from '@/features/orders';
```

---

## 5. State Management Patterns (Zustand)

### Auth Store
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/stores/auth.store.ts`

```typescript
interface AuthState {
  user: User | null;
  permissions: string[];
  isAuthenticated: boolean;
  setUser: (user: User | null) => void;
  setPermissions: (permissions: string[]) => void;
  clearAuth: () => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  permissions: [],
  isAuthenticated: false,

  setUser: (user) =>
    set({
      user,
      isAuthenticated: user !== null,
    }),

  setPermissions: (permissions) => set({ permissions }),

  clearAuth: () =>
    set({
      user: null,
      permissions: [],
      isAuthenticated: false,
    }),
}));

// Selectors for optimized re-renders
export const useUser = () => useAuthStore((state) => state.user);
export const useIsAuthenticated = () => useAuthStore((state) => state.isAuthenticated);
export const usePermissions = () => useAuthStore((state) => state.permissions);
```

### Other Stores Available
- `ui.store.ts` - UI state (theme, sidebar, etc.)
- `hub.store.ts` - Hub/workspace selection

### Pattern: Selector Usage
```typescript
// ❌ Bad: Causes re-render on any store change
const user = useAuthStore();

// ✅ Good: Only re-renders when user changes
const user = useUser();
```

---

## 6. API Integration Patterns

### API Client Pattern
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/lib/orders-client.ts`

```typescript
const API_BASE = '/api/v1';

// Query string builder
function buildQueryString(params: OrderListParams): string {
  const searchParams = new URLSearchParams();
  if (params.page) searchParams.set('page', String(params.page));
  if (params.status) searchParams.set('status', params.status);
  // ... more params
  return searchParams.toString() ? `?${searchParams.toString()}` : '';
}

// Fetch with error handling
export async function getOrders(params: OrderListParams = {}): Promise<OrderListResponse> {
  const response = await fetch(`${API_BASE}/orders/${buildQueryString(params)}`);

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Failed to fetch orders');
  }

  return response.json();
}
```

### React Query Hook Pattern
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/hooks/use-orders.ts`

```typescript
// Query keys (structured for cache invalidation)
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

// Query hook
export function useOrders(params: OrderListParams = {}) {
  return useQuery({
    queryKey: orderKeys.list(params),
    queryFn: () => getOrders(params),
    staleTime: 1000 * 60 * 2, // 2 minutes
  });
}

// Mutation hook with cache invalidation
export function useUpdateOrderNotes(orderId: string) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (notes: string) => updateOrderNotes(orderId, notes),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: orderKeys.detail(orderId) });
      queryClient.invalidateQueries({ queryKey: orderKeys.timeline(orderId) });
    },
  });
}
```

### N+1 Query Problem Solution
```typescript
// Batch endpoint to fetch items for multiple orders
export function useOrdersWithItems(params: OrderListParams = {}) {
  const ordersQuery = useOrders(params);
  const orders = ordersQuery.data?.data ?? [];
  const orderIds = orders.map((o) => o.id);

  // Single batch request instead of N separate requests
  const itemsBatchQuery = useQuery({
    queryKey: orderKeys.itemsBatch(orderIds),
    queryFn: () => getOrderItemsBatch(orderIds),
    enabled: ordersQuery.isSuccess && orderIds.length > 0,
  });

  // Combine results
  const itemsMap = itemsBatchQuery.data ?? {};
  const ordersWithItems = orders.map((order) => ({
    ...order,
    items: itemsMap[order.id]?.items ?? [],
  }));

  return { data: ordersWithItems, ... };
}
```

---

## 7. Permission & Role Checking Patterns

### Auth Guard Component
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/components/auth/auth-guard.tsx`

```typescript
// Wraps pages/sections that require authentication
export function AuthGuard({ children }: AuthGuardProps) {
  const router = useRouter();
  const pathname = usePathname();
  const { isLoading, isAuthenticated } = useAuth();

  useEffect(() => {
    if (isLoading) return;

    // Redirect to login if not authenticated
    if (!hasToken() && !isAuthenticated) {
      const callbackUrl = encodeURIComponent(pathname);
      router.replace(`/login?callbackUrl=${callbackUrl}`);
    }
  }, [isLoading, isAuthenticated, router, pathname]);

  if (isLoading || (!isAuthenticated && hasToken())) {
    return <Loader2 className="h-8 w-8 animate-spin" />;
  }

  if (!isAuthenticated && !hasToken()) {
    return null; // Will redirect
  }

  return <>{children}</>;
}
```

### Admin Guard Component
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/components/auth/admin-guard.tsx`

```typescript
// More restrictive: requires ADMIN role
export function AdminGuard({ children }: AdminGuardProps) {
  const router = useRouter();
  const { user, isLoading, isAuthenticated } = useAuth();

  useEffect(() => {
    if (isLoading) return;

    // Not authenticated -> redirect to login
    if (!hasToken() && !isAuthenticated) {
      router.replace('/login?callbackUrl=/admin');
      return;
    }

    // Authenticated but not admin -> redirect to main app
    if (user && user.role !== 'ADMIN') {
      router.replace('/');
    }
  }, [isLoading, isAuthenticated, user, router]);

  // Loading/redirect states...

  if (user && user.role !== 'ADMIN') {
    return <div className="min-h-screen flex items-center justify-center">
      <Loader2 className="h-8 w-8 animate-spin" />
    </div>;
  }

  return <>{children}</>;
}
```

### Auth Hook
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/hooks/use-auth.ts`

```typescript
export function useAuth() {
  const { setUser, clearAuth, user, isAuthenticated } = useAuthStore();

  const query = useQuery({
    queryKey: AUTH_QUERY_KEY,
    queryFn: getMe,
    enabled: hasToken(),
    retry: false,
    staleTime: 1000 * 60 * 5, // 5 minutes
  });

  // Sync query data with store
  useEffect(() => {
    if (query.data && query.data.id !== user?.id) {
      setUser(query.data);
    }
  }, [query.data, user?.id, setUser]);

  // Clear on error (unauthorized)
  useEffect(() => {
    if (query.error && isAuthenticated) {
      clearAuth();
    }
  }, [query.error, isAuthenticated, clearAuth]);

  return {
    user: query.data ?? user,
    isLoading: query.isLoading,
    isAuthenticated: !!query.data || isAuthenticated,
    error: query.error,
  };
}
```

### Usage in Components
```typescript
// Check if user has specific permission
const { user, isAuthenticated } = useAuth();

if (isAuthenticated && user?.role === 'ADMIN') {
  // Show admin-only features
}

// Check permissions array
const { permissions } = usePermissions();
if (permissions.includes('orders.export')) {
  // Show export button
}
```

---

## 8. Status & Configuration Constants

**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/lib/constants.ts`

### Status Configurations Pattern
```typescript
interface StatusConfig {
  label: string;
  color: string;
  bgClass: string;
  textClass: string;
}

export const ORDER_STATUS_CONFIG: Record<OrderStatusType, StatusConfig> = {
  PENDING: {
    label: 'Pending',
    color: 'gray',
    bgClass: 'bg-gray-50',
    textClass: 'text-gray-600',
  },
  IN_DESIGN: {
    label: 'In Design',
    color: 'amber',
    bgClass: 'bg-amber-50',
    textClass: 'text-amber-600',
  },
  // ... more statuses
};

// Design phase status (NONE, PENDING, IN_PROGRESS, COMPLETED)
export const DESIGN_PHASE_CONFIG: Record<DesignPhaseStatusType, StatusConfig> = { ... };

// Item design status (NOT_REQUIRED, PENDING, IN_PROGRESS, APPROVED)
export const ITEM_DESIGN_STATUS_CONFIG: Record<ItemDesignStatusType, StatusConfig> = { ... };

// Payment status (UNPAID, PAID, PARTIALLY_PAID, REFUNDED)
export const PAYMENT_STATUS_CONFIG: Record<PaymentStatusType, StatusConfig> = { ... };

// Task status (PENDING, TODO, IN_PROGRESS, REVIEW, REVISION, APPROVED, BLOCKED)
export const TASK_STATUS_CONFIG: Record<DesignTaskStatusType, StatusConfig> = { ... };
```

### Platform Configuration
```typescript
export const PLATFORM_CONFIG: Record<PlatformType | 'ETSY', PlatformConfig> = {
  WOOCOMMERCE: { letter: 'W', bgClass: 'bg-[#7B3DB0]' },
  SHOPIFY: { letter: 'S', bgClass: 'bg-[#96BF48]' },
  ETSY: { letter: 'E', bgClass: 'bg-[#F56400]' },
  MANUAL: { letter: 'M', bgClass: 'bg-[#6B7280]' },
};
```

### Design Position Options
```typescript
export const DESIGN_POSITION_OPTIONS = [
  { value: 'FRONT', label: 'Front' },
  { value: 'BACK', label: 'Back' },
  { value: 'LEFT_SLEEVE', label: 'Left Sleeve' },
  // ... 8 more positions
] as const;
```

---

## 9. Type System & Architecture

### Order Types
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/types/order.types.ts`

```typescript
// Re-exported from mocks
export type MockOrder, MockOrderItem, OrderStatusType, DesignPhaseStatusType, ...

// Extended types
export interface OrderWithItems extends MockOrder {
  items: MockOrderItem[];
}

export interface OrderListParams {
  page?: number;
  limit?: number;
  status?: OrderStatusType;
  design_status?: DesignPhaseStatusType;
  fulfiller_name?: string;
  search?: string;
  from?: string; // ISO date
  to?: string;   // ISO date
}

export interface OrderListResponse {
  data: MockOrder[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

export interface OrderItemsResponse {
  items: MockOrderItem[];
  total: number;
}

export interface OrderTimelineEvent {
  id: string;
  orderId: string;
  type: string; // ORDER_CREATED, DESIGN_STARTED, etc.
  userId: string;
  userName: string;
  description: string;
  metadata?: Record<string, unknown>;
  createdAt: string;
}

export interface OrderTimelineResponse {
  events: OrderTimelineEvent[];
  total: number;
}
```

---

## 10. Page Component Architecture

### Orders List Page
**File:** `/Users/taquanglinh/Documents/synkao/apps/web/src/app/(main)/orders/page.tsx`

```typescript
export default function OrdersPage() {
  // State Management
  const [page, setPage] = useState(1);
  const [search, setSearch] = useState('');
  const [status, setStatus] = useState<OrderStatusType | 'ALL'>('ALL');
  const [designStatus, setDesignStatus] = useState<DesignPhaseStatusType | 'ALL'>('ALL');
  const [selectedOrderIds, setSelectedOrderIds] = useState<Set<string>>(new Set());

  // Data Fetching
  const debouncedSearch = useDebouncedValue(search, 300);
  const { data: countsData } = useOrderCounts();
  const { data, isLoading, error } = useOrders({
    page, limit: 10, search: debouncedSearch || undefined, status, design_status
  });

  // Event Handlers
  const handleSearchChange = (value: string) => { /* reset page & selection */ };
  const handleStatusChange = (value: OrderStatusType | 'ALL') => { /* reset */ };
  const handlePageChange = (newPage: number) => { /* update page */ };
  const handleOrderClick = (order: MockOrder) => { router.push(`/orders/${order.id}`); };
  const handleSelectionChange = (ids: Set<string>) => { /* track selected */ };

  // Bulk Actions
  const handleExport = () => { /* export selected */ };
  const handleCreateFulfillment = () => { /* create fulfillment */ };
  const handleAssignDesigner = () => { /* assign designer */ };
  const handleClearSelection = () => { /* clear */ };

  return (
    <>
      <OrdersPageHeader />
      <OrderStatusTabs />
      <OrderListFilters />
      <OrderListTable />
      <OrdersPagination />
      <BulkActionBar />
    </>
  );
}
```

---

## 11. Key Files Reference Map

### Core Feature Files
| File | Purpose | Key Exports |
|------|---------|-------------|
| `features/orders/index.ts` | Feature barrel export | All types, hooks, components |
| `features/orders/types/order.types.ts` | Type definitions | OrderListParams, OrderListResponse |
| `features/orders/lib/orders-client.ts` | API calls | getOrders, getOrderItems, updateOrderNotes |
| `features/orders/hooks/use-orders.ts` | React Query hooks | useOrders, useOrdersWithItems, orderKeys |
| `features/orders/lib/constants.ts` | UI configs | STATUS_CONFIG, DESIGN_UPLOAD_CONFIG |

### Dialog Components
| File | Type | Use Case |
|------|------|----------|
| `order-list/import-orders-dialog.tsx` | Controlled Dialog | CSV file import |
| `order-detail/design-upload-modal.tsx` | DialogTrigger Modal | Design file upload |
| `order-detail/cancel-order-dialog.tsx` | AlertDialog | Destructive action |
| `order-detail/edit-notes-dialog.tsx` | Dialog + Mutation | Form submission with API call |

### Layout Components
| File | Purpose |
|------|---------|
| `components/layout/page-header.tsx` | Page title, breadcrumbs, action buttons |
| `components/layout/sidebar-nav.tsx` | Main navigation |
| `components/layout/admin-header.tsx` | Admin section header |

### Auth & Stores
| File | Type |
|------|------|
| `components/auth/auth-guard.tsx` | Auth check wrapper |
| `components/auth/admin-guard.tsx` | Admin role wrapper |
| `hooks/use-auth.ts` | Auth query + store sync |
| `stores/auth.store.ts` | Zustand auth store |

---

## 12. Data Flow Patterns

### List Page Data Flow
```
User Filter Input
    ↓
useDebouncedValue(300ms)
    ↓
useOrders() React Query
    ↓
orders-client.getOrders(API_BASE/orders)
    ↓
OrderListTable (renders with orders)
    ↓
OrderRow (with expand)
    ↓
User expands → useOrderItems(lazy-load items)
    ↓
OrderItemRow (shows item details)
```

### Dialog Form Submission Flow
```
User fills form
    ↓
State updated (setNotes, setFiles)
    ↓
User clicks Save/Submit
    ↓
useMutation triggered (updateNotes, cancelOrder)
    ↓
orders-client.updateOrderNotes() / cancelOrder()
    ↓
onSuccess callback:
  - queryClient.invalidateQueries() (refresh lists)
  - toast.success()
  - dialog.onOpenChange(false) (close)
  - router.push() (navigate if needed)
    ↓
UI reflects updates (cached data invalidated)
```

---

## 13. Best Practices Identified

### ✅ Strong Points
1. **Modular feature structure** - Orders feature is self-contained with clear boundaries
2. **Query key organization** - Structured query keys for precise cache invalidation
3. **Selector pattern** - useUser, usePermissions selectors prevent unnecessary re-renders
4. **Error handling** - Consistent error messages from API responses
5. **File upload validation** - Both MIME type and extension validation
6. **Dialog patterns** - Controlled state management, proper cleanup on close
7. **Batch queries** - Solves N+1 problem with single batch endpoint
8. **Type safety** - Full TypeScript coverage with no `any` types
9. **Loading states** - Proper loading spinners in auth guards
10. **Debouncing** - Search input debounced to reduce API calls

### Integration Points
- **Page header**: `OrdersPageHeader` imports `ImportOrdersDialog` directly
- **Dialog callback**: `onImport` callback receives files, page handles integration
- **Auth integration**: Pages wrapped in `AuthGuard` via layout structure
- **Toast notifications**: `sonner` for user feedback in dialogs

---

## File Paths Summary

### Critical Files for Import Feature
```
✓ Already implemented:
/Users/taquanglinh/Documents/synkao/apps/web/src/app/(main)/orders/orders-page-header.tsx
/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/components/order-list/import-orders-dialog.tsx

⚠ Needs integration:
/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/lib/orders-client.ts (add POST /orders/import endpoint)
/Users/taquanglinh/Documents/synkao/apps/web/src/features/orders/hooks/use-orders.ts (add useImportOrders mutation)

Layout structure:
/Users/taquanglinh/Documents/synkao/apps/web/src/app/(main)/orders/page.tsx
/Users/taquanglinh/Documents/synkao/apps/web/src/app/(main)/layout.tsx (AuthGuard wrapper)
```

### Permission Checking Files
```
/Users/taquanglinh/Documents/synkao/apps/web/src/components/auth/auth-guard.tsx
/Users/taquanglinh/Documents/synkao/apps/web/src/components/auth/admin-guard.tsx
/Users/taquanglinh/Documents/synkao/apps/web/src/stores/auth.store.ts
/Users/taquanglinh/Documents/synkao/apps/web/src/hooks/use-auth.ts
/Users/taquanglinh/Documents/synkao/apps/web/src/lib/auth-client.ts
```

---

## Unresolved Questions

1. **CSV Import Backend API**: What's the endpoint path for importing CSV orders? (Expected: POST /api/v1/orders/import or similar)
2. **Permission Scope**: Should "Import Orders" be restricted to specific roles (ADMIN only, or MANAGER too)?
3. **File Storage**: Where are imported CSV files temporarily stored during processing? (Expected: /tmp or S3)
4. **Bulk Operations**: Implement Export, Create Fulfillment, Assign Designer in BulkActionBar?

---

## Recommendations

1. **Create `useImportOrders` hook** in `hooks/use-orders.ts` following the mutation pattern
2. **Add API client function** `importOrdersCSV()` in `lib/orders-client.ts`
3. **Implement permission check** for import button (admin-only or via permissions array)
4. **Add error handling** for CSV validation errors (line-by-line feedback)
5. **Consider progress tracking** for large CSV imports (multiple files or large datasets)

