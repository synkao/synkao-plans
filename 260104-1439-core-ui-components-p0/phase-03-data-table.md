# Phase 3: Data Table Component

## Context
- **Issue:** #10
- **Parent Plan:** [plan.md](./plan.md)
- **Reference:** `apps/web/src/components/ui/table.tsx`

## Overview
| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P0 |
| Effort | 4h |
| Status | completed |
| Dependencies | @tanstack/react-table |

## Key Insights

Từ codebase analysis:
- `table.tsx` có primitives: Table, TableHeader, TableBody, TableRow, TableCell
- Pattern: compound components, data-slot attributes
- Use case: Orders table, Tasks backlog, Workspaces list
- Need: sorting, filtering, pagination, row selection

## Decisions (from user)
- **Server-side pagination:** Cần ngay từ đầu với mock API data
- Approach: TanStack Table manual pagination mode + useQuery for API calls

## Requirements

1. Generic DataTable component
2. Column header với sort indicators
3. Pagination controls
4. Toolbar với search/filter
5. Row actions dropdown
6. Empty state display
7. Loading skeleton

## Dependencies Installation

```bash
pnpm add -F @synkao/web @tanstack/react-table
```

## Architecture

### Main Component API

```tsx
interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[];
  data: TData[];
  // Server-side pagination
  pageCount?: number;
  pageIndex?: number;
  pageSize?: number;
  onPaginationChange?: (pagination: { pageIndex: number; pageSize: number }) => void;
  // Other
  searchKey?: keyof TData;
  onRowClick?: (row: TData) => void;
  emptyMessage?: string;
  loading?: boolean;
}
```

### Server-side Pagination Flow

```
1. Component receives: data (current page), pageCount, pageIndex, pageSize
2. User clicks next page → onPaginationChange({ pageIndex: n+1, pageSize })
3. Parent component (via useQuery) fetches new page from API
4. Pass new data + updated pageIndex back to DataTable
```

### File Structure

```
apps/web/src/components/data-table/
├── data-table.tsx              # Main component
├── data-table-column-header.tsx # Sortable header
├── data-table-pagination.tsx    # Page controls
├── data-table-toolbar.tsx       # Search/filter bar
├── data-table-row-actions.tsx   # Row action dropdown
├── data-table-empty-state.tsx   # Empty message
└── index.ts                     # Barrel export
```

## Related Code Files

- `apps/web/src/components/ui/table.tsx` - Base table primitives
- `apps/web/src/components/ui/button.tsx` - Button for actions
- `apps/web/src/components/ui/dropdown-menu.tsx` - Row actions menu
- `apps/web/src/components/ui/input.tsx` - Search input
- `apps/web/src/features/design/components/backlog/backlog-table.tsx` - Existing table

## Implementation Steps

### Step 1: Install dependency (5m)

```bash
cd apps/web && pnpm add @tanstack/react-table
```

### Step 2: Create data-table-column-header.tsx (30m)

```tsx
interface DataTableColumnHeaderProps<TData, TValue>
  extends React.HTMLAttributes<HTMLDivElement> {
  column: Column<TData, TValue>;
  title: string;
}

function DataTableColumnHeader<TData, TValue>({
  column,
  title,
  className,
}: DataTableColumnHeaderProps<TData, TValue>) {
  if (!column.getCanSort()) {
    return <div className={cn(className)}>{title}</div>;
  }

  return (
    <Button
      variant="ghost"
      size="sm"
      onClick={() => column.toggleSorting(column.getIsSorted() === "asc")}
    >
      {title}
      {column.getIsSorted() === "asc" ? (
        <ArrowUpIcon className="ml-2 size-4" />
      ) : column.getIsSorted() === "desc" ? (
        <ArrowDownIcon className="ml-2 size-4" />
      ) : (
        <ArrowUpDownIcon className="ml-2 size-4" />
      )}
    </Button>
  );
}
```

### Step 3: Create data-table-pagination.tsx (45m)

```tsx
interface DataTablePaginationProps<TData> {
  table: Table<TData>;
}

function DataTablePagination<TData>({ table }: DataTablePaginationProps<TData>) {
  return (
    <div className="flex items-center justify-between px-2">
      <div className="text-muted-foreground text-sm">
        {table.getFilteredSelectedRowModel().rows.length} of{" "}
        {table.getFilteredRowModel().rows.length} row(s) selected.
      </div>
      <div className="flex items-center space-x-6 lg:space-x-8">
        {/* Rows per page selector */}
        {/* Page navigation buttons */}
      </div>
    </div>
  );
}
```

### Step 4: Create data-table-toolbar.tsx (30m)

```tsx
interface DataTableToolbarProps<TData> {
  table: Table<TData>;
  searchKey?: string;
  searchPlaceholder?: string;
}

function DataTableToolbar<TData>({
  table,
  searchKey,
  searchPlaceholder = "Search...",
}: DataTableToolbarProps<TData>) {
  return (
    <div className="flex items-center justify-between">
      {searchKey && (
        <Input
          placeholder={searchPlaceholder}
          value={(table.getColumn(searchKey)?.getFilterValue() as string) ?? ""}
          onChange={(e) =>
            table.getColumn(searchKey)?.setFilterValue(e.target.value)
          }
          className="max-w-sm"
        />
      )}
      {/* Additional filters, view options */}
    </div>
  );
}
```

### Step 5: Create data-table-row-actions.tsx (30m)

```tsx
interface DataTableRowActionsProps<TData> {
  row: Row<TData>;
  actions: {
    label: string;
    onClick: (row: TData) => void;
    icon?: React.ReactNode;
    variant?: "default" | "destructive";
  }[];
}

function DataTableRowActions<TData>({ row, actions }) {
  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="ghost" size="icon">
          <MoreHorizontalIcon className="size-4" />
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end">
        {actions.map((action) => (
          <DropdownMenuItem key={action.label} onClick={() => action.onClick(row.original)}>
            {action.icon}
            {action.label}
          </DropdownMenuItem>
        ))}
      </DropdownMenuContent>
    </DropdownMenu>
  );
}
```

### Step 6: Create data-table-empty-state.tsx (15m)

```tsx
interface DataTableEmptyStateProps {
  message?: string;
  icon?: React.ReactNode;
}

function DataTableEmptyState({
  message = "No results found.",
  icon,
}: DataTableEmptyStateProps) {
  return (
    <TableRow>
      <TableCell colSpan={100} className="h-24 text-center">
        {icon}
        <p className="text-muted-foreground">{message}</p>
      </TableCell>
    </TableRow>
  );
}
```

### Step 7: Create data-table.tsx (1h)

Main component tying everything together:

```tsx
function DataTable<TData, TValue>({
  columns,
  data,
  pagination = true,
  searchKey,
  onRowClick,
  emptyMessage,
  loading,
}: DataTableProps<TData, TValue>) {
  const [sorting, setSorting] = useState<SortingState>([]);
  const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([]);
  const [rowSelection, setRowSelection] = useState({});

  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    onSortingChange: setSorting,
    onColumnFiltersChange: setColumnFilters,
    onRowSelectionChange: setRowSelection,
    state: { sorting, columnFilters, rowSelection },
  });

  return (
    <div className="space-y-4">
      <DataTableToolbar table={table} searchKey={searchKey} />
      <Table>
        <TableHeader>
          {table.getHeaderGroups().map((headerGroup) => (
            <TableRow key={headerGroup.id}>
              {headerGroup.headers.map((header) => (
                <TableHead key={header.id}>
                  {flexRender(header.column.columnDef.header, header.getContext())}
                </TableHead>
              ))}
            </TableRow>
          ))}
        </TableHeader>
        <TableBody>
          {table.getRowModel().rows?.length ? (
            table.getRowModel().rows.map((row) => (
              <TableRow
                key={row.id}
                onClick={() => onRowClick?.(row.original)}
                className={onRowClick ? "cursor-pointer" : ""}
              >
                {row.getVisibleCells().map((cell) => (
                  <TableCell key={cell.id}>
                    {flexRender(cell.column.columnDef.cell, cell.getContext())}
                  </TableCell>
                ))}
              </TableRow>
            ))
          ) : (
            <DataTableEmptyState message={emptyMessage} />
          )}
        </TableBody>
      </Table>
      {pagination && <DataTablePagination table={table} />}
    </div>
  );
}
```

### Step 8: Create index.ts & test (15m)

```tsx
export { DataTable } from "./data-table";
export { DataTableColumnHeader } from "./data-table-column-header";
// ... other exports
```

## Todo List

- [x] Install @tanstack/react-table
- [x] Create data-table-column-header.tsx
- [x] Create data-table-pagination.tsx
- [x] Create data-table-toolbar.tsx
- [x] Create data-table-row-actions.tsx
- [x] Create data-table-empty-state.tsx
- [x] Create data-table.tsx (main)
- [x] Create index.ts barrel export
- [x] Add to design-system showcase (dev-tools page)
- [x] Test với mock orders data

## Success Criteria

- [x] Table renders với data
- [x] Click header sorts asc/desc
- [x] Search filters rows
- [x] Pagination changes pages
- [x] Row selection checkboxes work (built-in feature)
- [x] Empty state shows when no data
- [x] Row click callback fires (optional prop)

## Risk Assessment

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| TanStack learning curve | Low | Low | Follow shadcn examples |
| Performance với large data | Medium | Medium | Virtual scrolling (later) |
| Column definition complexity | Low | Low | Provide example columns |

## Security Considerations

- No sensitive data handling - pure UI
- Row actions should validate permissions in handlers

## Next Steps

Sau khi complete Phase 3:
1. Create example column definitions for Orders, Tasks
2. Replace backlog-table với DataTable
3. Document usage patterns
