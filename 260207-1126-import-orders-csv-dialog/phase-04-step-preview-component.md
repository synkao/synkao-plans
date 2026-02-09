# Phase 04 — Step Preview Component

## Context Links
- Parent: [plan.md](./plan.md)
- Depends on: [Phase 01](./phase-01-setup-and-types.md), [Phase 02](./phase-02-csv-parser-and-validator.md)
- shadcn Table: `apps/web/src/components/ui/table.tsx`
- shadcn Badge: `apps/web/src/components/ui/badge.tsx`
- shadcn Tabs: `apps/web/src/components/ui/tabs.tsx`

## Overview
- **Priority:** P1
- **Status:** pending
- **Description:** Step 2 of wizard — validation summary chips, filterable data table preview with row-level error/warning highlighting.

## Key Insights
- Summary chips show at-a-glance counts: total, valid, errors, warnings
- Table shows all CSV columns as headers + a "Status" column
- Filter tabs: All | Valid | Errors | Warnings — client-side filter on `ImportValidatedRow.status`
- Error rows: light red background; warning rows: light yellow; valid: default
- Row issues shown as tooltip or inline beneath the row
- Table should scroll horizontally (CSV can have many columns) and vertically (max ~300px height)

## Requirements

### Functional
- Summary chips row: 4 colored badges showing total/valid/error/warning counts
- Filter tabs: Tat ca | Hop le | Loi | Canh bao — filter displayed rows
- Data table with CSV headers as columns + status indicator column
- Row highlighting: error=red bg, warning=yellow bg, valid=default
- Hovering an error/warning cell shows issue tooltip
- Table virtualization NOT needed for 500 rows max — simple scroll

### Non-functional
- Component < 200 LOC (if table gets big, extract table sub-component)
- Responsive horizontal scroll for wide tables

## Architecture

```
Props:
  parseResult: ImportParseResult
  activeFilter: ImportPreviewFilter
  onFilterChange: (filter: ImportPreviewFilter) => void

Renders:
  ┌──────────────────────────────────┐
  │ [Total] [Valid] [Errors] [Warns] │  ← summary chips
  │                                  │
  │ [All] [Valid] [Errors] [Warns]   │  ← filter tabs
  │ ┌──────────────────────────────┐ │
  │ │ # │ order_no │ name │ ... │ S│ │  ← scrollable table
  │ │ 1 │ ORD-001  │ John │ ... │ ✓│ │
  │ │ 2 │          │ Jane │ ... │ ✗│ │  ← red bg (missing order_no)
  │ └──────────────────────────────┘ │
  └──────────────────────────────────┘
```

## Related Code Files

### Create
- `apps/web/src/features/orders/components/order-list/import-step-preview.tsx`

### Reference (read-only)
- `apps/web/src/components/ui/table.tsx`
- `apps/web/src/components/ui/badge.tsx`

## Implementation Steps

1. **Create `import-step-preview.tsx`**

2. **Define props:**
   ```typescript
   interface ImportStepPreviewProps {
     parseResult: ImportParseResult;
     activeFilter: ImportPreviewFilter;
     onFilterChange: (filter: ImportPreviewFilter) => void;
   }
   ```

3. **Summary chips section:**
   - 4 inline badges in a flex row with gap-2
   - Total: gray badge — `{summary.total} dong` (rows)
   - Valid: green badge — `{summary.valid} hop le`
   - Errors: red badge — `{summary.errors} loi`
   - Warnings: yellow badge — `{summary.warnings} canh bao`
   - Each badge: `px-3 py-1 rounded-full text-xs font-medium`

4. **Filter tabs:**
   - 4 tab buttons in a row (not using shadcn Tabs — simpler inline buttons)
   - Each: `Tat ca (N)`, `Hop le (N)`, `Loi (N)`, `Canh bao (N)` where N is count
   - Active tab: border-b-2 border-primary text-primary font-medium
   - Inactive: text-muted-foreground hover:text-foreground
   - onClick → `onFilterChange(filter)`

5. **Filter logic:**
   ```typescript
   const filteredRows = useMemo(() => {
     if (activeFilter === 'all') return parseResult.rows;
     return parseResult.rows.filter(r => r.status === activeFilter);
   }, [parseResult.rows, activeFilter]);
   ```

6. **Data table:**
   - Scrollable container: `max-h-[300px] overflow-auto border rounded-lg`
   - Use shadcn `<Table>`, `<TableHeader>`, `<TableBody>`, `<TableRow>`, `<TableCell>`
   - First column: row index `#` (sticky left, narrow)
   - Middle columns: one per CSV header from `parseResult.headers`
   - Last column: status icon (checkmark green / X red / ! yellow)
   - Header row: sticky top, bg-muted

7. **Row styling:**
   ```typescript
   const rowBgClass = {
     valid: '',
     error: 'bg-red-50',
     warning: 'bg-amber-50',
   }[row.status];
   ```

8. **Cell-level error indicators:**
   - If a cell's column has an issue in `row.issues`, show:
     - Red underline for errors, yellow underline for warnings
     - Tooltip on hover with issue message (use `title` attribute for simplicity)
   - Implementation: check `row.issues.find(i => i.column === header)`

9. **Empty state:**
   - If filteredRows.length === 0: show "Khong co dong nao" centered text

10. **Export** as named export

## Todo List
- [ ] Create import-step-preview.tsx
- [ ] Implement summary chips with counts
- [ ] Implement filter tabs with active state
- [ ] Implement scrollable data table with CSV columns
- [ ] Implement row highlighting (red/yellow/default)
- [ ] Implement cell-level error indicators with tooltips
- [ ] Handle empty filtered state
- [ ] Compile check passes

## Success Criteria
- Summary chips show correct counts matching validation
- Filter tabs correctly filter displayed rows
- Error rows have red background, warning rows yellow
- Cells with issues show tooltip on hover
- Table scrolls horizontally and vertically
- Component < 200 LOC

## Risk Assessment
| Risk | Impact | Mitigation |
|------|--------|------------|
| Too many columns overflow | Low | Horizontal scroll container |
| 500 rows renders slowly | Low | Simple HTML table is fine for 500 rows; no virtualization needed |
| Long cell values break layout | Low | Truncate with text-ellipsis, max-w on cells |

## Security Considerations
- Sanitize cell values before rendering — use React's built-in JSX escaping (default behavior)
- No `dangerouslySetInnerHTML` anywhere

## Next Steps
- Phase 06 passes `parseResult` from step 1 to this component
