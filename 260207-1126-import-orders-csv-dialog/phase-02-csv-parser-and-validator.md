# Phase 02 — CSV Parser & Validator

## Context Links
- Parent: [plan.md](./plan.md)
- Depends on: [Phase 01](./phase-01-setup-and-types.md) (types + papaparse)
- Types: `apps/web/src/features/orders/types/import-orders.types.ts`
- Existing lib: `apps/web/src/features/orders/lib/`

## Overview
- **Priority:** P1
- **Status:** pending
- **Description:** Client-side CSV parsing with papaparse + row-level validation logic. Pure functions, no React.

## Key Insights
- papaparse `Papa.parse(file, { header: true })` returns array of objects keyed by header
- Validation is row-by-row: check required fields, data types, value ranges
- Warnings vs errors: missing optional fields = warning; missing required = error; invalid number = error
- Platform column: validate against known values (WOOCOMMERCE, SHOPIFY, MANUAL) — warn if unknown
- Keep parsing and validation as separate pure functions for testability

## Requirements

### Functional
- Parse CSV file → array of `ImportCsvRow` objects
- Validate headers against expected columns (warn on unknown, error on missing required)
- Validate each row: required fields present, quantity/price are positive numbers
- Return `ImportParseResult` with rows, summary, and detected headers
- Generate downloadable CSV template file
- Enforce limits: max 5MB file, max 500 rows

### Non-functional
- File < 200 LOC
- Pure functions (no side effects, no React hooks)
- Synchronous validation after async parse

## Architecture

```
File (CSV)
  ↓ parseImportCsv(file)         — async, uses papaparse
ImportCsvRow[]
  ↓ validateImportRows(rows)     — sync, pure
ImportValidatedRow[]
  ↓ computeValidationSummary(rows) — sync, pure
ImportParseResult
```

### Validation Rules

| Column | Rule | Type |
|--------|------|------|
| order_number | Non-empty string | error |
| customer_name | Non-empty string | error |
| customer_email | Optional; if present, basic email format check | warning |
| customer_phone | Optional | — |
| product_name | Non-empty string | error |
| variant | Optional | — |
| quantity | Positive integer (>0) | error |
| unit_price | Non-negative number (>=0) | error |
| shipping_address | Optional | — |
| note | Optional | — |
| platform | Optional; warn if not in known list | warning |

## Related Code Files

### Create
- `apps/web/src/features/orders/lib/import-csv-parser.ts`

### Modify
- `apps/web/src/features/orders/lib/index.ts` — add export

## Implementation Steps

1. **Create `import-csv-parser.ts`** with these functions:

2. **`parseImportCsv(file: File): Promise<ImportParseResult>`**
   - Use `Papa.parse(file, { header: true, skipEmptyLines: true })` wrapped in a Promise
   - After parse, extract headers from `results.meta.fields`
   - Validate headers: check all required columns from `CSV_TEMPLATE_COLUMNS` are present
   - If missing required headers, return early with all rows marked as error
   - Call `validateImportRows()` on parsed data
   - Call `computeValidationSummary()` on validated rows
   - Return `{ rows, summary, headers }`

3. **`validateImportRows(rawRows: ImportCsvRow[]): ImportValidatedRow[]`**
   - Map over raw rows with 1-based index
   - For each row, collect issues array:
     - Required field checks: `if (!row[key]?.trim())` → push error
     - `quantity`: `parseInt` → check `isNaN` or `<= 0` → push error
     - `unit_price`: `parseFloat` → check `isNaN` or `< 0` → push error
     - `customer_email`: if present, check `/^[^\s@]+@[^\s@]+\.[^\s@]+$/` → push warning
     - `platform`: if present, check against `['WOOCOMMERCE', 'SHOPIFY', 'MANUAL']` → push warning
   - Set row `status`: has error → 'error', has warning only → 'warning', else → 'valid'

4. **`computeValidationSummary(rows: ImportValidatedRow[]): ImportValidationSummary`**
   - Count rows by status using reduce
   - Return `{ total, valid, errors, warnings }`

5. **`generateCsvTemplate(): string`**
   - Build CSV header line from `CSV_TEMPLATE_COLUMNS` keys
   - Add one example row with sample data
   - Return as string (caller can create Blob for download)

6. **`downloadCsvTemplate(): void`**
   - Call `generateCsvTemplate()`
   - Create Blob → URL.createObjectURL → trigger download via hidden `<a>` click
   - Revoke URL after

7. **Constants** at top of file:
   ```typescript
   export const IMPORT_CSV_MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
   export const IMPORT_CSV_MAX_ROWS = 500;
   ```

8. **Update barrel export** in `lib/index.ts`:
   ```typescript
   export * from './import-csv-parser';
   ```

9. **Compile check**

## Todo List
- [ ] Create import-csv-parser.ts with all functions
- [ ] Export parseImportCsv, validateImportRows, computeValidationSummary
- [ ] Export generateCsvTemplate, downloadCsvTemplate
- [ ] Export IMPORT_CSV_MAX_FILE_SIZE, IMPORT_CSV_MAX_ROWS
- [ ] Update lib barrel export
- [ ] Compile check passes

## Success Criteria
- `parseImportCsv(file)` returns correct `ImportParseResult` for valid/invalid CSVs
- Validation catches: missing required fields, invalid numbers, bad emails, unknown platforms
- Template download generates valid CSV with correct headers
- All functions are pure (except downloadCsvTemplate which triggers DOM download)
- File stays < 200 LOC

## Risk Assessment
| Risk | Impact | Mitigation |
|------|--------|------------|
| Large CSV causes UI freeze | Medium | papaparse has streaming mode; not needed for 500 rows but can upgrade later |
| Encoding issues (UTF-8 BOM) | Low | papaparse handles BOM automatically |
| Header case sensitivity | Medium | Normalize headers to lowercase during parse |

## Security Considerations
- Sanitize parsed values before display (handled in preview component)
- File size check before parsing to prevent memory issues

## Next Steps
- Phase 03 uses `parseImportCsv` and `downloadCsvTemplate` in upload step
- Phase 04 uses `ImportParseResult` to render preview table
