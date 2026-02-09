# Phase 03 — Step Upload Component

## Context Links
- Parent: [plan.md](./plan.md)
- Depends on: [Phase 01](./phase-01-setup-and-types.md), [Phase 02](./phase-02-csv-parser-and-validator.md)
- FileUploader ref: `apps/web/src/components/ui/file-uploader.tsx`
- Existing dialog: `apps/web/src/features/orders/components/order-list/import-orders-dialog.tsx`

## Overview
- **Priority:** P1
- **Status:** pending
- **Description:** Step 1 of the wizard — file dropzone, file info pill, helper chips, and CSV template download.

## Key Insights
- Reuse existing `FileUploader` component for drag-drop — already has `react-dropzone`
- After file selection, immediately parse CSV to show row count in file pill
- "Template mau" chip triggers `downloadCsvTemplate()` from Phase 02
- Other helper chips (Huong dan, WooCommerce, Shopify) render disabled with `// TODO`
- Parent dialog passes `onFileReady(result: ImportParseResult)` callback

## Requirements

### Functional
- Dropzone area accepting .csv only, max 5MB, max 1 file
- After file drop/select: parse CSV, show file pill with name + size + row count
- File pill has remove button (resets to empty state)
- 4 helper chips below dropzone:
  - "Template mau" — active, calls downloadCsvTemplate()
  - "Huong dan" — disabled, tooltip "Coming soon"
  - "WooCommerce" — disabled, tooltip "Coming soon"
  - "Shopify" — disabled, tooltip "Coming soon"
- Loading spinner while parsing CSV
- Error state if file too large or parse fails

### Non-functional
- Component < 200 LOC
- No internal state for parsed result — lift up to parent via callback

## Architecture

```
Props:
  file: File | null
  parseResult: ImportParseResult | null
  isParsing: boolean
  onFileSelect: (file: File) => void
  onFileRemove: () => void

Renders:
  ┌─────────────────────────────┐
  │  [Dropzone / File Pill]     │
  │  [Helper Chips Row]         │
  └─────────────────────────────┘
```

- File selection triggers `onFileSelect(file)` → parent calls `parseImportCsv` → passes back `parseResult`
- This keeps the component stateless w.r.t. parse result

## Related Code Files

### Create
- `apps/web/src/features/orders/components/order-list/import-step-upload.tsx`

### Reference (read-only)
- `apps/web/src/components/ui/file-uploader.tsx` — reuse dropzone pattern
- `apps/web/src/features/orders/lib/import-csv-parser.ts` — for downloadCsvTemplate

## Implementation Steps

1. **Create `import-step-upload.tsx`**

2. **Define props interface:**
   ```typescript
   interface ImportStepUploadProps {
     file: File | null;
     parseResult: ImportParseResult | null;
     isParsing: boolean;
     onFileSelect: (file: File) => void;
     onFileRemove: () => void;
   }
   ```

3. **Dropzone section** (when no file selected):
   - Use `useDropzone` from `react-dropzone` directly (not FileUploader, for more control)
   - Accept: `{ 'text/csv': ['.csv'] }`
   - maxSize: `IMPORT_CSV_MAX_FILE_SIZE` (5MB)
   - maxFiles: 1
   - `onDrop`: call `onFileSelect(acceptedFiles[0])` if valid
   - Render: upload cloud icon, "Keo tha file CSV hoac click de chon", "Chi chap nhan .csv, toi da 5MB"
   - Style: dashed border, hover bg-accent/50, drag-active blue highlight

4. **File pill section** (when file selected):
   - Show file icon + filename + file size (formatted) + row count from parseResult
   - Row count: `parseResult?.summary.total ?? '...'` (show '...' while parsing)
   - Remove button (X icon) → calls `onFileRemove()`
   - If isParsing: show small spinner next to row count
   - Style: rounded-lg border p-3, flex items-center gap-3

5. **Helper chips row** (always visible, below dropzone/pill):
   - Render 4 chips in a flex-wrap row
   - "Template mau" chip:
     - Icon: `Download` from lucide
     - onClick: `downloadCsvTemplate()`
     - Style: active chip — bg-primary/10, text-primary, hover:bg-primary/20, cursor-pointer
   - Other 3 chips:
     - Icons: `BookOpen`, `ShoppingCart`, `ShoppingCart` (or relevant icons)
     - Style: disabled — opacity-50, cursor-not-allowed
     - Title tooltip: "Coming soon"
     - Comment: `// TODO: Implement guide/WooCommerce/Shopify links`

6. **Error display**:
   - If `useDropzone` rejects file (wrong type, too large): show red text below dropzone
   - If parseResult has all error rows (e.g., missing required headers): show warning banner

7. **Export** the component as named export

## Todo List
- [ ] Create import-step-upload.tsx
- [ ] Implement dropzone with react-dropzone
- [ ] Implement file pill with row count
- [ ] Implement 4 helper chips (1 active + 3 disabled)
- [ ] Handle loading/error states
- [ ] Compile check passes

## Success Criteria
- Drag-drop and click-to-select both work for .csv files
- Non-.csv files are rejected with error message
- Files > 5MB rejected
- File pill shows correct name, size, parsed row count
- "Template mau" downloads a valid CSV template
- Component < 200 LOC

## Risk Assessment
| Risk | Impact | Mitigation |
|------|--------|------------|
| Large file causes slow parse display | Low | Show spinner during parse, 500-row limit |
| Dropzone styling conflicts | Low | Follow existing FileUploader styling |

## Security Considerations
- File type validation at dropzone level (accept prop)
- Size validation before parse attempt

## Next Steps
- Phase 06 orchestrates this component within the dialog wizard
