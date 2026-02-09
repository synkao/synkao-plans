# Phase 01 — Setup & Types

## Context Links
- Parent: [plan.md](./plan.md)
- Issue: [#78](https://github.com/synkao/synkao/issues/78)
- Existing types: `apps/web/src/features/orders/types/order.types.ts`
- Mock types: `apps/web/src/mocks/types.ts`

## Overview
- **Priority:** P1
- **Status:** pending
- **Description:** Install papaparse dependency and create import-specific TypeScript types.

## Key Insights
- CSV template has 11 columns; maps loosely to MockOrder/MockOrderItem fields
- Validation produces row-level errors/warnings — need typed error objects
- Wizard state needs explicit step enum + per-step data containers
- Keep types in a dedicated file to avoid bloating `order.types.ts`

## Requirements

### Functional
- Install `papaparse` and `@types/papaparse`
- Define CSV column schema (required vs optional columns)
- Define parsed row type, validation result type, import result type
- Define wizard step enum and wizard state type

### Non-functional
- Types file < 100 LOC
- Zero runtime code in types file (types only)

## Architecture

```
ImportCsvRow (raw parsed row from CSV)
  ↓ validate
ImportValidatedRow (row + errors[] + warnings[])
  ↓ group
ImportValidationSummary { total, valid, errors, warnings }
  ↓ mock import
ImportResult { success, failed, errors[] }
```

## Related Code Files

### Modify
- `apps/web/package.json` — add `papaparse`, `@types/papaparse`
- `apps/web/src/features/orders/types/index.ts` — add export for new types file

### Create
- `apps/web/src/features/orders/types/import-orders.types.ts` — all import-specific types

## Implementation Steps

1. **Install papaparse**
   ```bash
   cd apps/web && pnpm add papaparse && pnpm add -D @types/papaparse
   ```

2. **Create `import-orders.types.ts`** with these types:

   ```typescript
   // Wizard step enum
   export type ImportWizardStep = 'upload' | 'preview' | 'result';

   // CSV column definition
   export interface CsvColumnDef {
     key: string;
     label: string;
     required: boolean;
   }

   // CSV_TEMPLATE_COLUMNS constant array
   export const CSV_TEMPLATE_COLUMNS: CsvColumnDef[] = [
     { key: 'order_number', label: 'Order Number', required: true },
     { key: 'customer_name', label: 'Customer Name', required: true },
     { key: 'customer_email', label: 'Email', required: false },
     { key: 'customer_phone', label: 'Phone', required: false },
     { key: 'product_name', label: 'Product Name', required: true },
     { key: 'variant', label: 'Variant', required: false },
     { key: 'quantity', label: 'Quantity', required: true },
     { key: 'unit_price', label: 'Unit Price', required: true },
     { key: 'shipping_address', label: 'Shipping Address', required: false },
     { key: 'note', label: 'Note', required: false },
     { key: 'platform', label: 'Platform', required: false },
   ];

   // Raw parsed CSV row (all strings)
   export interface ImportCsvRow {
     [key: string]: string;
   }

   // Validation issue on a single row
   export interface ImportRowIssue {
     column: string;
     message: string;
     type: 'error' | 'warning';
   }

   // Validated row with status
   export interface ImportValidatedRow {
     rowIndex: number;       // 1-based (matches CSV line number)
     data: ImportCsvRow;
     issues: ImportRowIssue[];
     status: 'valid' | 'error' | 'warning';
   }

   // Summary of validation pass
   export interface ImportValidationSummary {
     total: number;
     valid: number;
     errors: number;
     warnings: number;
   }

   // Parse result combining rows + summary
   export interface ImportParseResult {
     rows: ImportValidatedRow[];
     summary: ImportValidationSummary;
     headers: string[];
   }

   // Filter tab for preview
   export type ImportPreviewFilter = 'all' | 'valid' | 'error' | 'warning';

   // Mock import result
   export interface ImportResult {
     success: number;
     failed: number;
     errors: { rowIndex: number; message: string }[];
   }
   ```

3. **Update barrel export** in `features/orders/types/index.ts`:
   ```typescript
   export * from './import-orders.types';
   ```

4. **Run compile check** — `pnpm --filter web build` or `tsc --noEmit`

## Todo List
- [ ] Install papaparse + @types/papaparse
- [ ] Create import-orders.types.ts
- [ ] Update types barrel export
- [ ] Compile check passes

## Success Criteria
- `pnpm --filter web tsc --noEmit` passes
- All import-related types importable from `@/features/orders/types`
- papaparse available in node_modules

## Risk Assessment
| Risk | Impact | Mitigation |
|------|--------|------------|
| papaparse version conflict | Low | Pin to latest stable (~5.x) |
| Type naming clash with existing | Low | Prefix all types with `Import` |

## Security Considerations
- N/A for types-only phase

## Next Steps
- Phase 02 depends on these types for parser/validator implementation
