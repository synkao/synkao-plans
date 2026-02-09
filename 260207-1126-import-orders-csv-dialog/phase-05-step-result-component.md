# Phase 05 — Step Result Component

## Context Links
- Parent: [plan.md](./plan.md)
- Depends on: [Phase 01](./phase-01-setup-and-types.md)
- Types: `ImportResult` from `import-orders.types.ts`

## Overview
- **Priority:** P1
- **Status:** pending
- **Description:** Step 3 of wizard — loading state during "import", then result summary with success/failed counts and error details.

## Key Insights
- Two states: loading (during mock import) and complete (showing results)
- Loading: spinner + progress bar animating 0-100% over ~2s
- Result: success/failed count cards + optional error detail list
- "Import them" button resets entire wizard to Step 1

## Requirements

### Functional
- Loading state: centered spinner icon + "Dang import..." text + animated progress bar
- Result state: two stat cards (success green, failed red) + error detail list
- Error detail: scrollable list showing row number + error message
- If all success and 0 failed: show success illustration/icon

### Non-functional
- Component < 150 LOC
- Clean transition from loading to result (no flash)

## Architecture

```
Props:
  isImporting: boolean
  importResult: ImportResult | null

Renders (isImporting=true):
  ┌───────────────────────┐
  │    [Spinner Icon]     │
  │   Dang import...      │
  │   [Progress Bar 67%]  │
  └───────────────────────┘

Renders (isImporting=false, importResult):
  ┌───────────────────────┐
  │ [✓ 45 thanh cong]     │
  │ [✗ 3 that bai]        │
  │                       │
  │ Chi tiet loi:         │
  │ - Dong 12: missing... │
  │ - Dong 45: invalid... │
  └───────────────────────┘
```

## Related Code Files

### Create
- `apps/web/src/features/orders/components/order-list/import-step-result.tsx`

## Implementation Steps

1. **Create `import-step-result.tsx`**

2. **Define props:**
   ```typescript
   interface ImportStepResultProps {
     isImporting: boolean;
     importResult: ImportResult | null;
   }
   ```

3. **Loading state** (when `isImporting === true`):
   - Centered layout: `flex flex-col items-center justify-center py-12 gap-4`
   - Spinner: `Loader2` icon from lucide with `animate-spin` class, `size-10 text-primary`
   - Text: `<p className="text-muted-foreground">Dang import don hang...</p>`
   - Progress bar: use shadcn `<Progress>` component
     - Animate value from 0 to 90 using `useEffect` + `setInterval` (every 200ms, +~5%)
     - Cap at 90% until import completes (real progress not available for mock)
     - Width: `w-64`

4. **Result state** (when `isImporting === false && importResult !== null`):
   - Stat cards row: flex gap-4 justify-center
   - Success card:
     - `bg-emerald-50 border-emerald-200 rounded-lg p-4 text-center min-w-[120px]`
     - `CheckCircle2` icon emerald, `text-2xl font-bold text-emerald-700` for count
     - Label: "Thanh cong"
   - Failed card (only if failed > 0):
     - `bg-red-50 border-red-200 rounded-lg p-4 text-center min-w-[120px]`
     - `XCircle` icon red, `text-2xl font-bold text-red-700` for count
     - Label: "That bai"

5. **Error details** (only if `importResult.errors.length > 0`):
   - Section heading: "Chi tiet loi" with `text-sm font-medium text-muted-foreground mt-4`
   - Scrollable list: `max-h-[150px] overflow-y-auto space-y-1 mt-2`
   - Each error: `text-xs text-red-600` — `Dong {rowIndex}: {message}`

6. **All-success state** (failed === 0):
   - Show `CheckCircle2` icon large (size-16) in emerald
   - Text: "Tat ca don hang da duoc import thanh cong!"

7. **Export** as named export

## Todo List
- [ ] Create import-step-result.tsx
- [ ] Implement loading state with spinner + progress bar
- [ ] Implement result stat cards (success + failed)
- [ ] Implement error details list
- [ ] Implement all-success celebration state
- [ ] Compile check passes

## Success Criteria
- Loading state shows spinner + animated progress bar
- Result correctly shows success/failed counts
- Error details list is scrollable and shows row-level errors
- All-success state shows positive feedback
- Component < 150 LOC

## Risk Assessment
| Risk | Impact | Mitigation |
|------|--------|------------|
| Progress bar animation jank | Low | Use CSS transitions, simple setInterval |
| Flash between loading/result | Low | Conditional render, no intermediate state |

## Security Considerations
- Error messages from mock are controlled — but sanitize via JSX escaping

## Next Steps
- Phase 06 orchestrates loading/result state transitions
