# Phase 02: Print Specs Section

## Context

- Parent: [plan.md](plan.md)
- Dependencies: Phase 01 (types)
- Docs: [codebase-summary.md](../../docs/codebase-summary.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P2 |
| Effort | 1h |
| Implementation Status | Pending |
| Review Status | Pending |

**Objective:** Create PrintSpecsSection component hiển thị print specifications (key-value) với edit modal.

## Key Insights

1. Pattern: Follow TaskInfoSection styling
2. UI: Key-value display, click Edit opens modal
3. Modal: Dynamic add/remove key-value pairs
4. Data: printSpecs từ MockOrderItem (Record<string, string>)

## Requirements

### PrintSpecsSection Component

- Hiển thị key-value list từ task's order item printSpecs
- Edit button opens modal
- Inline display với label + value pairs
- Icon: `FileText` from lucide-react

### EditSpecsModal Component

- Form với dynamic key-value pairs
- Add new pair button
- Remove pair button
- Save/Cancel actions
- Demo only: không persist changes

### Mock Data Example

```typescript
printSpecs: {
  "Print Size": "A3",
  "DPI": "300",
  "Color Mode": "CMYK",
  "Material": "Cotton 250gsm",
  "Bleed": "3mm"
}
```

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `drawer/print-specs-section.tsx` | Create | Key-value display |
| `drawer/edit-specs-modal.tsx` | Create | Edit form modal |
| `task-detail-drawer.tsx` | Modify | Add PrintSpecsSection |

## Implementation Steps

### Step 1: Create print-specs-section.tsx

```tsx
// Structure
export function PrintSpecsSection({ task }: { task: MockTask }) {
  // Get printSpecs from order item
  // Render key-value list
  // Edit button opens modal
}
```

### Step 2: Create edit-specs-modal.tsx

```tsx
// Features
- Dialog from shadcn/ui
- Dynamic form fields
- Add/remove pairs
- Form state management
```

### Step 3: Integrate into drawer

Add PrintSpecsSection after TaskInfoSection với Separator.

## Todo List

- [ ] Create print-specs-section.tsx
- [ ] Create edit-specs-modal.tsx
- [ ] Style key-value display
- [ ] Implement dynamic form
- [ ] Add to task-detail-drawer.tsx
- [ ] Test modal open/close

## Success Criteria

- [ ] Specs display correctly
- [ ] Edit modal opens
- [ ] Dynamic add/remove works
- [ ] Styling matches design system

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Empty specs | Low | Show "No specs" placeholder |
| Form complexity | Medium | Keep simple, no validation |

## Security Considerations

None - UI only, demo data.

## Next Steps

→ Phase 03: Reference Files Section
