# Phase 07 — Permission & Integration

## Context Links
- Parent: [plan.md](./plan.md)
- Depends on: [Phase 06](./phase-06-dialog-wizard-orchestrator.md)
- Auth store: `apps/web/src/stores/auth.store.ts`
- RBAC spec: `orders.import` — Owner/Staff only
- Page header: `apps/web/src/app/(main)/orders/orders-page-header.tsx`

## Overview
- **Priority:** P1
- **Status:** pending
- **Description:** Add permission check to conditionally render Import button. Update barrel exports. Final integration and compile check.

## Key Insights
- `usePermissions()` from auth store returns `string[]` of permission codes
- `orders.import` is the permission for import — granted to Owner and Staff, not Designer
- Simply wrap Import button in conditional render check
- Also update all barrel export files for new components/modules

## Requirements

### Functional
- Import button hidden for users without `orders.import` permission
- Barrel exports updated for all new files
- Full compile check passes
- Manual smoke test: dialog opens, 3 steps work, close resets

### Non-functional
- No new hook needed — reuse existing `usePermissions()`
- Zero extra dependencies

## Architecture

```
OrdersPageHeader
  └─ usePermissions() → permissions.includes('orders.import')
       └─ true → render Import button + ImportOrdersDialog
       └─ false → hide Import button entirely
```

## Related Code Files

### Modify
- `apps/web/src/app/(main)/orders/orders-page-header.tsx` — add permission check
- `apps/web/src/features/orders/components/order-list/index.ts` — no change needed (ImportOrdersDialog already exported)
- `apps/web/src/features/orders/lib/index.ts` — add import-csv-parser export
- `apps/web/src/features/orders/types/index.ts` — add import-orders.types export (done in Phase 01)

## Implementation Steps

1. **Update `orders-page-header.tsx`:**
   - Import `usePermissions` from `@/stores/auth.store`
   - Add at component top: `const permissions = usePermissions();`
   - Add: `const canImport = permissions.includes('orders.import');`
   - Wrap Import button in: `{canImport && (<Button ...>Import</Button>)}`
   - Also conditionally render ImportOrdersDialog: `{canImport && (<ImportOrdersDialog ... />)}`

   Updated component:
   ```tsx
   export function OrdersPageHeader() {
     const [importDialogOpen, setImportDialogOpen] = useState(false);
     const permissions = usePermissions();
     const canImport = permissions.includes('orders.import');

     return (
       <>
         <PageHeader
           title="Orders"
           description="Manage your print-on-demand orders"
           breadcrumbs={[{ label: 'Dashboard', href: '/' }, { label: 'Orders' }]}
           actions={
             <div className="flex gap-2">
               {canImport && (
                 <Button variant="outline" onClick={() => setImportDialogOpen(true)}>
                   <Upload className="mr-2 h-4 w-4" />
                   Import
                 </Button>
               )}
               <Button>
                 <Plus className="mr-2 h-4 w-4" />
                 New Order
               </Button>
             </div>
           }
         />
         {canImport && (
           <ImportOrdersDialog
             open={importDialogOpen}
             onOpenChange={setImportDialogOpen}
           />
         )}
       </>
     );
   }
   ```

2. **Verify barrel exports** — confirm all new files are exported:
   - `features/orders/types/index.ts` → exports `import-orders.types`
   - `features/orders/lib/index.ts` → exports `import-csv-parser`
   - `features/orders/components/order-list/index.ts` → `ImportOrdersDialog` already exported
   - No need to export step sub-components (internal to dialog only)

3. **Run full compile check:**
   ```bash
   cd apps/web && pnpm tsc --noEmit
   ```

4. **Run lint:**
   ```bash
   cd apps/web && pnpm lint
   ```

5. **Manual smoke test** (dev server):
   - Login as Owner → Import button visible
   - Login as Designer → Import button hidden
   - Click Import → dialog opens at Step 1
   - Upload a .csv → file pill appears with row count
   - Click Tiep tuc → Step 2 shows preview table
   - Click Import → Step 3 shows loading then result
   - Click Import them → resets to Step 1
   - Click Dong → dialog closes, state reset
   - ESC / overlay click → dialog closes, state reset

## Todo List
- [ ] Add permission check to orders-page-header.tsx
- [ ] Verify all barrel exports are correct
- [ ] Run tsc --noEmit (compile check)
- [ ] Run pnpm lint
- [ ] Manual smoke test

## Success Criteria
- Import button only visible for Owner/Staff roles
- Designer role cannot see Import button
- All TypeScript compilation passes with zero errors
- Lint passes
- Full wizard flow works end-to-end in browser

## Risk Assessment
| Risk | Impact | Mitigation |
|------|--------|------------|
| Permissions array empty on first load | Low | Button won't render until permissions loaded — acceptable UX |
| Auth store not populated in some edge cases | Low | Same behavior as other permission-gated features |

## Security Considerations
- Permission check is FE-only — backend will enforce `orders.import` permission on real API
- No sensitive data exposed even if button is shown (mock import only)

## Next Steps
- Feature complete for Phase 1 (mock import)
- Future: replace `mockImportOrders` with real API call when backend ready
- Future: add .xlsx support, WooCommerce/Shopify guide links
