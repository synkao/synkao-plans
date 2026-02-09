# Phase 01: Remove Print Specs + Add Description Section

## Context Links

- Issue: #80 - Fix Design Task (TaskDetailDrawer)
- [plan.md](./plan.md)
- Drawer: `apps/web/src/features/design/components/drawer/`
- Mock types: `apps/web/src/mocks/types.ts`

## Overview

- **Priority**: P2
- **Status**: pending
- **Description**: Delete PrintSpecs section and EditSpecs modal entirely. Remove `printSpecs` from MockOrderItem type and mock data. Add inline read-only Description section in drawer between TaskInfoSection and ReferenceFilesSection.

## Key Insights

- `printSpecs` only referenced in 3 files: print-specs-section.tsx, edit-specs-modal.tsx, order-items.data.ts
- order-item-card-details.tsx has NO printSpecs references (confirmed via grep)
- MockTask already has `description?: string` (line 220 in types.ts) -- just need to populate mock data
- Description section is simple enough to inline in task-detail-drawer.tsx (no separate component needed, <20 LOC)

## Requirements

### Functional
- Print Specs section completely removed from drawer UI
- No compile errors or broken imports after removal
- Description section shows task.description text or "No description" italic placeholder
- Description positioned between TaskInfoSection and ReferenceFilesSection

### Non-functional
- Zero unused imports/exports after cleanup
- Mock data stays consistent (no orphaned references)

## Architecture

Simple deletion + inline addition. No new components, hooks, or state needed.

```
TaskDetailDrawer layout (after):
  SheetHeader
    taskCode + productName + customerName + QuickStatusActions
  Body
    TaskInfoSection
    Separator
    Description (inline, new)
    Separator
    ReferenceFilesSection
    Separator
    DesignVersionsSection
    Separator  ← was before PrintSpecs, now removed
    CommentsSection
    Separator
    TimelineSection
```

## Related Code Files

### Files to DELETE
- `apps/web/src/features/design/components/drawer/print-specs-section.tsx` (68 lines)
- `apps/web/src/features/design/components/drawer/edit-specs-modal.tsx` (128 lines)

### Files to MODIFY
1. `apps/web/src/features/design/components/drawer/task-detail-drawer.tsx`
   - Remove `PrintSpecsSection` import (line 13)
   - Remove `<PrintSpecsSection>` render + its `<Separator>` (lines 60-63)
   - Add Description section between TaskInfoSection and ReferenceFilesSection

2. `apps/web/src/features/design/components/drawer/index.ts`
   - Remove line 5: `export { PrintSpecsSection, type PrintSpecsSectionProps }`
   - Remove line 11: `export { EditSpecsModal, type EditSpecsModalProps }`

3. `apps/web/src/mocks/types.ts`
   - Remove line 332: `printSpecs?: Record<string, string>;`

4. `apps/web/src/mocks/data/order-items.data.ts`
   - Remove `printSpecsByType` constant (lines 29-34)
   - Remove `printSpecs: printSpecsByType[type]` assignment (line 134)

5. `apps/web/src/mocks/data/tasks.data.ts`
   - Add `description` field to generated tasks with varied fake descriptions

## Implementation Steps

### Step 1: Delete files
```bash
rm apps/web/src/features/design/components/drawer/print-specs-section.tsx
rm apps/web/src/features/design/components/drawer/edit-specs-modal.tsx
```

### Step 2: Clean up index.ts exports
Remove these two lines from `drawer/index.ts`:
```ts
// DELETE: export { PrintSpecsSection, type PrintSpecsSectionProps } from './print-specs-section';
// DELETE: export { EditSpecsModal, type EditSpecsModalProps } from './edit-specs-modal';
```

### Step 3: Update task-detail-drawer.tsx
- Remove `PrintSpecsSection` import
- Remove `<Separator />` + `<PrintSpecsSection task={task} />` block (the one between DesignVersionsSection and CommentsSection)
- Add Description section after TaskInfoSection:

```tsx
{/* Description */}
<Separator />
<div className="space-y-2">
  <div className="flex items-center gap-2">
    <FileText className="h-4 w-4 text-muted-foreground" />
    <h3 className="text-sm font-semibold text-foreground">Description</h3>
  </div>
  {task.description ? (
    <p className="text-sm text-foreground whitespace-pre-wrap">{task.description}</p>
  ) : (
    <p className="text-sm text-muted-foreground italic">No description</p>
  )}
</div>
```

Import `FileText` from lucide-react.

### Step 4: Remove printSpecs from MockOrderItem type
In `mocks/types.ts`, delete:
```ts
printSpecs?: Record<string, string>;
```

### Step 5: Remove printSpecs from mock data
In `mocks/data/order-items.data.ts`:
- Delete the `printSpecsByType` object (lines 29-34)
- Delete `printSpecs: printSpecsByType[type],` from the map function

### Step 6: Add descriptions to mock tasks
In `mocks/data/tasks.data.ts`, add a descriptions array and assign to tasks:

```ts
const descriptions = [
  'Design front print with customer logo centered. Use brand colors #FF5733 and #2E86C1.',
  'Full wrap mug design. Handle area must remain blank. Include bleed marks.',
  'Canvas art reproduction. Match Pantone 7462C exactly. High-res 600dpi required.',
  'Hoodie back print. Maximum print area 14x16 inches. DTG compatible.',
  'Poster layout A2 size. Trim marks required. CMYK color space only.',
  undefined, // some tasks have no description
];
```

In the map function, add:
```ts
description: descriptions[idx % descriptions.length],
```

### Step 7: Compile check
```bash
cd apps/web && pnpm tsc --noEmit
```

## Todo List

- [ ] Delete print-specs-section.tsx and edit-specs-modal.tsx
- [ ] Remove exports from drawer/index.ts
- [ ] Remove PrintSpecsSection import + render from task-detail-drawer.tsx
- [ ] Add inline Description section in task-detail-drawer.tsx
- [ ] Remove printSpecs field from MockOrderItem type
- [ ] Remove printSpecs data from order-items.data.ts
- [ ] Add description data to tasks.data.ts mock generation
- [ ] Run tsc --noEmit to verify no compile errors

## Success Criteria

- `pnpm tsc --noEmit` passes with zero errors
- No grep results for "printSpecs" or "PrintSpecs" or "EditSpecs" in src/
- Description section renders in drawer between TaskInfoSection and ReferenceFilesSection
- Tasks show varied description text in drawer

## Risk Assessment

- **Very Low**: Pure deletion + small inline addition. No logic changes.
- If any external consumer imports PrintSpecsSection or EditSpecsModal, compile will catch it.

## Security Considerations

- Description is read-only display; no user input processed
- `whitespace-pre-wrap` on description text -- ensure no HTML injection (React auto-escapes by default)

## Next Steps

- Proceed to Phase 02: Workspace Dropdown + Merge Status Button
