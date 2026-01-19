# Phase 3: Page Component Refactor

## Overview
Refactor BacklogPage to use server-side filtering and pagination.

## Tasks

- [ ] Add pagination state (page, pageSize)
- [ ] Integrate debounced search
- [ ] Replace client-side filtering with server params
- [ ] Handle pagination metadata from API
- [ ] Clear selection on page/filter change

---

## Implementation

**File:** `apps/web/src/app/(main)/design/backlog/page.tsx`

### 1. Add Imports

```ts
import { useDebouncedValue } from '@/hooks/use-debounced-value';
```

### 2. State Changes

```tsx
// Pagination state
const [page, setPage] = useState(1);
const [pageSize, setPageSize] = useState(20);

// Filter state (keep existing)
const [search, setSearch] = useState('');
const [urgentOnly, setUrgentOnly] = useState(false);
const [source, setSource] = useState<TaskSourceType | 'ALL'>('ALL');

// Debounced search
const debouncedSearch = useDebouncedValue(search, 300);
```

### 3. API Call with Server Params

```tsx
// React Query with server-side params
const { data: taskResponse, isLoading, error } = useBacklogTasks({
  page,
  limit: pageSize,
  search: debouncedSearch || undefined,
  is_urgent: urgentOnly || undefined,
  source: source !== 'ALL' ? source : undefined,
});

// Extract data
const tasks = taskResponse?.data ?? [];
const pagination = taskResponse?.pagination;
```

### 4. Remove Client-side Filtering

DELETE the entire `useMemo` block that does client-side filtering:

```tsx
// DELETE THIS BLOCK
const { backlogTasks, filteredTasks } = useMemo(() => {
  // ... all client-side filter logic
}, [taskResponse?.data, search, urgentOnly, source]);
```

### 5. Reset Page on Filter Change

```tsx
// Reset to page 1 when filters change
useEffect(() => {
  setPage(1);
}, [debouncedSearch, urgentOnly, source]);
```

### 6. Clear Selection on Page Change

```tsx
// Clear selection when page changes
useEffect(() => {
  setSelectedIds(new Set());
}, [page]);
```

### 7. Update Stats Bar

```tsx
// Use pagination.total for accurate count
<BacklogStatsBar
  tasks={tasks}
  totalCount={pagination?.total}
/>
```

### 8. Update Table Usage

```tsx
<BacklogTable
  tasks={tasks}  // Use tasks directly, not filteredTasks
  onTaskClick={handleTaskClick}
  onSelectionChange={handleSelectionChange}
/>
```

---

## Validation

- [ ] Search triggers API call after 300ms delay
- [ ] Filter changes reset to page 1
- [ ] Selection clears on page change
- [ ] Loading state shows during fetches
- [ ] Stats show correct total count
