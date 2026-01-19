# Phase 4: Pagination UI

## Overview
Add pagination controls below the table.

## Tasks

- [ ] Create BacklogPagination component
- [ ] Integrate into page
- [ ] Export from feature index

---

## Decision: Custom vs Existing

**Existing `DataTablePagination`:**
- Requires TanStack Table instance
- BacklogTable uses custom table, not TanStack

**Decision:** Create simple standalone `BacklogPagination` component.

---

## Implementation

**File:** `apps/web/src/features/design/components/backlog/backlog-pagination.tsx` (NEW)

```tsx
'use client';

import {
  ChevronLeft,
  ChevronRight,
  ChevronsLeft,
  ChevronsRight,
} from 'lucide-react';
import { Button } from '@/components/ui/button';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';

interface BacklogPaginationProps {
  page: number;
  pageSize: number;
  total: number;
  totalPages: number;
  onPageChange: (page: number) => void;
  onPageSizeChange: (size: number) => void;
  pageSizeOptions?: number[];
}

export function BacklogPagination({
  page,
  pageSize,
  total,
  totalPages,
  onPageChange,
  onPageSizeChange,
  pageSizeOptions = [10, 20, 30, 50],
}: BacklogPaginationProps) {
  const canPrevious = page > 1;
  const canNext = page < totalPages;

  return (
    <div className="flex items-center justify-between px-2 py-4">
      <div className="text-sm text-muted-foreground">
        {total} task{total !== 1 ? 's' : ''} total
      </div>

      <div className="flex items-center gap-6">
        {/* Page size selector */}
        <div className="flex items-center gap-2">
          <span className="text-sm">Rows</span>
          <Select
            value={String(pageSize)}
            onValueChange={(v) => onPageSizeChange(Number(v))}
          >
            <SelectTrigger className="h-8 w-[70px]">
              <SelectValue />
            </SelectTrigger>
            <SelectContent>
              {pageSizeOptions.map((size) => (
                <SelectItem key={size} value={String(size)}>
                  {size}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>

        {/* Page indicator */}
        <span className="text-sm">
          Page {page} of {totalPages || 1}
        </span>

        {/* Navigation */}
        <div className="flex items-center gap-1">
          <Button
            variant="outline"
            size="icon"
            className="h-8 w-8"
            disabled={!canPrevious}
            onClick={() => onPageChange(1)}
          >
            <ChevronsLeft className="h-4 w-4" />
          </Button>
          <Button
            variant="outline"
            size="icon"
            className="h-8 w-8"
            disabled={!canPrevious}
            onClick={() => onPageChange(page - 1)}
          >
            <ChevronLeft className="h-4 w-4" />
          </Button>
          <Button
            variant="outline"
            size="icon"
            className="h-8 w-8"
            disabled={!canNext}
            onClick={() => onPageChange(page + 1)}
          >
            <ChevronRight className="h-4 w-4" />
          </Button>
          <Button
            variant="outline"
            size="icon"
            className="h-8 w-8"
            disabled={!canNext}
            onClick={() => onPageChange(totalPages)}
          >
            <ChevronsRight className="h-4 w-4" />
          </Button>
        </div>
      </div>
    </div>
  );
}
```

---

## Export

**File:** `apps/web/src/features/design/components/backlog/index.ts`

Add export:
```ts
export { BacklogPagination } from './backlog-pagination';
```

**File:** `apps/web/src/features/design/index.ts`

Ensure re-export if needed.

---

## Integration in Page

**File:** `apps/web/src/app/(main)/design/backlog/page.tsx`

```tsx
import { BacklogPagination } from '@/features/design';

// In render, after BacklogTable:
{pagination && (
  <BacklogPagination
    page={page}
    pageSize={pageSize}
    total={pagination.total}
    totalPages={pagination.total_pages}
    onPageChange={setPage}
    onPageSizeChange={(size) => {
      setPageSize(size);
      setPage(1); // Reset to page 1
    }}
  />
)}
```

---

## Validation

- [ ] Pagination shows correct page/total
- [ ] Navigation buttons work
- [ ] Page size change resets to page 1
- [ ] Disabled states correct at boundaries
