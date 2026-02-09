# Phase 06 — Dialog Wizard Orchestrator

## Context Links
- Parent: [plan.md](./plan.md)
- Depends on: [Phase 01](./phase-01-setup-and-types.md) through [Phase 05](./phase-05-step-result-component.md)
- Current dialog: `apps/web/src/features/orders/components/order-list/import-orders-dialog.tsx`
- Dialog UI: `apps/web/src/components/ui/dialog.tsx`

## Overview
- **Priority:** P1
- **Status:** pending
- **Description:** Rewrite `ImportOrdersDialog` as 3-step wizard orchestrator. Manages wizard state, step transitions, and coordinates child step components.

## Key Insights
- Rewrite existing file (not create new) — `import-orders-dialog.tsx`
- State managed with `useState` — simple enough, no need for useReducer/Zustand
- Stepper UI: 3 circles connected by lines, active/completed/pending states
- Footer buttons change per step: dynamic labels + disabled conditions
- On close/cancel: reset all state. On "Import them": reset to step 1
- Mock import function: `setTimeout(2000)` returning fake ImportResult

## Requirements

### Functional
- Dialog modal ~680px width (`sm:max-w-[680px]`)
- Header: "Import don hang" title + mini stepper visualization
- Body: renders current step component
- Footer: dynamic button pair per step
- Step navigation rules:
  - Step 1 → 2: only when file parsed + has valid rows
  - Step 2 → 3: click "Import" triggers mock import
  - Step 3 → close: "Dong" closes dialog
  - Step 3 → 1: "Import them" resets wizard
  - Back: step 2 → 1 via "Quay lai"
- Close resets all wizard state
- After mock import completes: invalidate orders query

### Non-functional
- File < 200 LOC (step components handle their own rendering)
- Clean state reset on dialog close

## Architecture

### Wizard State
```typescript
// All state in parent dialog:
const [step, setStep] = useState<ImportWizardStep>('upload');
const [file, setFile] = useState<File | null>(null);
const [parseResult, setParseResult] = useState<ImportParseResult | null>(null);
const [isParsing, setIsParsing] = useState(false);
const [previewFilter, setPreviewFilter] = useState<ImportPreviewFilter>('all');
const [isImporting, setIsImporting] = useState(false);
const [importResult, setImportResult] = useState<ImportResult | null>(null);
```

### Step-to-Footer Mapping
| Step | Left Button | Right Button |
|------|-------------|-------------|
| upload | Huy (close) | Tiep tuc (disabled if !parseResult or 0 valid) |
| preview | Quay lai (→ upload) | Import (disabled if 0 valid, triggers import) |
| result | Dong (close) | Import them (→ reset to upload) |

### Mock Import Function
```typescript
async function mockImportOrders(validRows: ImportValidatedRow[]): Promise<ImportResult> {
  await new Promise(resolve => setTimeout(resolve, 2000));
  const failCount = Math.min(2, Math.floor(validRows.length * 0.05));
  return {
    success: validRows.length - failCount,
    failed: failCount,
    errors: failCount > 0
      ? [{ rowIndex: validRows[1]?.rowIndex ?? 2, message: 'Duplicate order number' }]
      : [],
  };
}
```

## Related Code Files

### Modify (rewrite)
- `apps/web/src/features/orders/components/order-list/import-orders-dialog.tsx`

### Modify (add mock + hook)
- `apps/web/src/features/orders/lib/orders-client.ts` — add `mockImportOrders` function
- `apps/web/src/features/orders/hooks/use-orders.ts` — add `useImportOrders` mutation hook

### Reference
- `apps/web/src/components/ui/dialog.tsx`
- All step components from Phase 03-05

## Implementation Steps

1. **Add mock import function to `orders-client.ts`:**
   ```typescript
   export async function mockImportOrders(
     validRows: ImportValidatedRow[]
   ): Promise<ImportResult> {
     // TODO: Replace with real API call — POST /api/v1/orders/import
     await new Promise(resolve => setTimeout(resolve, 2000));
     const failCount = Math.min(2, Math.floor(validRows.length * 0.05));
     return {
       success: validRows.length - failCount,
       failed: failCount,
       errors: failCount > 0
         ? [{ rowIndex: validRows[1]?.rowIndex ?? 2, message: 'Duplicate order number' }]
         : [],
     };
   }
   ```

2. **Add `useImportOrders` hook to `use-orders.ts`:**
   ```typescript
   export function useImportOrders() {
     const queryClient = useQueryClient();
     return useMutation({
       mutationFn: (validRows: ImportValidatedRow[]) => mockImportOrders(validRows),
       onSuccess: () => {
         queryClient.invalidateQueries({ queryKey: orderKeys.lists() });
       },
     });
   }
   ```

3. **Rewrite `import-orders-dialog.tsx`:**

4. **Props interface** (keep same as current for backward compat):
   ```typescript
   interface ImportOrdersDialogProps {
     open: boolean;
     onOpenChange: (open: boolean) => void;
   }
   ```
   - Remove `onImport` prop — dialog now handles import internally via hook

5. **Stepper component** (inline, ~20 lines):
   - 3 circles (numbered 1-2-3) connected by lines
   - Active step: `bg-primary text-primary-foreground`
   - Completed step: `bg-primary/20 text-primary` with checkmark
   - Pending step: `bg-muted text-muted-foreground`
   - Labels: "Upload", "Preview", "Ket qua"
   - Render as flex row with `items-center gap-2`

6. **File selection handler:**
   ```typescript
   const handleFileSelect = async (selectedFile: File) => {
     setFile(selectedFile);
     setIsParsing(true);
     try {
       const result = await parseImportCsv(selectedFile);
       setParseResult(result);
     } catch {
       setParseResult(null);
     } finally {
       setIsParsing(false);
     }
   };
   ```

7. **File remove handler:**
   ```typescript
   const handleFileRemove = () => {
     setFile(null);
     setParseResult(null);
   };
   ```

8. **Import handler:**
   ```typescript
   const handleImport = async () => {
     if (!parseResult) return;
     setStep('result');
     setIsImporting(true);
     const validRows = parseResult.rows.filter(r => r.status !== 'error');
     const result = await importMutation.mutateAsync(validRows);
     setImportResult(result);
     setIsImporting(false);
   };
   ```

9. **Reset handler** (used for close + "Import them"):
   ```typescript
   const resetWizard = () => {
     setStep('upload');
     setFile(null);
     setParseResult(null);
     setIsParsing(false);
     setPreviewFilter('all');
     setIsImporting(false);
     setImportResult(null);
   };
   ```

10. **Dialog close handler:**
    ```typescript
    const handleClose = () => {
      resetWizard();
      onOpenChange(false);
    };
    ```

11. **Body rendering** — switch on `step`:
    - `'upload'`: `<ImportStepUpload file={file} parseResult={parseResult} isParsing={isParsing} onFileSelect={handleFileSelect} onFileRemove={handleFileRemove} />`
    - `'preview'`: `<ImportStepPreview parseResult={parseResult!} activeFilter={previewFilter} onFilterChange={setPreviewFilter} />`
    - `'result'`: `<ImportStepResult isImporting={isImporting} importResult={importResult} />`

12. **Footer rendering** — conditional per step (see mapping table above)

13. **Dialog content:**
    - `<DialogContent className="sm:max-w-[680px]">`
    - Header with title + stepper
    - Body (step component)
    - Footer with dynamic buttons

14. **Update `orders-page-header.tsx`** — remove `onImport` prop from ImportOrdersDialog usage:
    ```tsx
    <ImportOrdersDialog
      open={importDialogOpen}
      onOpenChange={setImportDialogOpen}
    />
    ```
    - Remove the `handleImport` function (no longer needed)

15. **Compile check**

## Todo List
- [ ] Add mockImportOrders to orders-client.ts
- [ ] Add useImportOrders to use-orders.ts
- [ ] Rewrite import-orders-dialog.tsx as wizard
- [ ] Implement mini stepper UI
- [ ] Implement step navigation + footer buttons
- [ ] Implement file handlers (select, remove, parse)
- [ ] Implement import handler with mutation
- [ ] Implement reset/close logic
- [ ] Update orders-page-header.tsx (remove onImport prop)
- [ ] Compile check passes

## Success Criteria
- Dialog opens as 3-step wizard
- Step navigation works correctly (forward, back, reset)
- File selection triggers CSV parse
- Import triggers mock delay + shows result
- "Import them" resets to step 1
- Close (overlay/X/ESC/Huy) resets all state
- Orders list refreshes after import via query invalidation
- File < 200 LOC

## Risk Assessment
| Risk | Impact | Mitigation |
|------|--------|------------|
| Dialog too tall on small screens | Medium | Max-height with scroll on body |
| State gets out of sync | Low | Single state location in parent, no derived state |
| Back navigation loses file | Low | State preserved when going back |

## Security Considerations
- No sensitive data in wizard state
- Mock import does not call any real API

## Next Steps
- Phase 07 adds permission check to hide Import button for designers
