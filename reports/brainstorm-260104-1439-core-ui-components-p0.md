# Brainstorm Report: Core UI Components (P0)

**Issues:** #7, #9, #10
**Priority:** P0 (Must have)
**Date:** 2026-01-04
**Status:** Brainstorm Complete

---

## Problem Statement

Cần implement 3 core UI components cho Synkao POD Management:
1. **#7** Glass Card variants - Card với glass effect
2. **#9** Form components - Input, Select, FileUploader, DatePicker
3. **#10** Data Table - TanStack Table với sort/filter/pagination

## Decisions Made

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Validation | react-hook-form + Zod | Type-safe, shadcn/ui pattern |
| Multi-select | cmdk + Popover | Lightweight, accessible |
| Dark mode | Light only | Đơn giản, hardcode CSS |
| Glass Card | Separate component | Separation of concerns |
| FileUploader | react-dropzone | Stable, drag-drop built-in |
| DatePicker | date-fns + react-day-picker | shadcn/ui pattern |
| Table use | Multiple entities | Generic component |
| Order | #7 → #9 → #10 | Từ simple → complex |

## Implementation Plan

### Phase 1: Issue #7 - Glass Card (~3h)

**Dependencies:** None (dùng Tailwind classes)

**Files tạo mới:**
```
apps/web/src/components/ui/glass-card.tsx
```

**Variants:**
| Variant | Description | CSS |
|---------|-------------|-----|
| default | Base glass effect | `bg-white/70 backdrop-blur-[10px] border-white/20` |
| hover | Tăng opacity on hover | `hover:bg-white/80` |
| selected | Border highlight | `border-primary ring-2` |
| clickable | Cursor + full hover | `cursor-pointer hover:shadow-lg` |

**Component API:**
```tsx
<GlassCard variant="clickable" selected={isSelected} onClick={handleClick}>
  <GlassCardHeader>...</GlassCardHeader>
  <GlassCardContent>...</GlassCardContent>
</GlassCard>
```

---

### Phase 2: Issue #9 - Form Components (~4h)

**Dependencies cần install:**
```bash
pnpm add react-hook-form @hookform/resolvers zod cmdk react-dropzone date-fns react-day-picker
```

**Files tạo mới:**
```
apps/web/src/components/ui/form.tsx           # Form wrapper
apps/web/src/components/ui/form-field.tsx     # Label + input + error
apps/web/src/components/ui/combobox.tsx       # Searchable select
apps/web/src/components/ui/multi-select.tsx   # Multi-select với tags
apps/web/src/components/ui/file-uploader.tsx  # Drag-drop uploader
apps/web/src/components/ui/textarea.tsx       # TextArea
apps/web/src/components/ui/date-picker.tsx    # DatePicker
apps/web/src/components/ui/calendar.tsx       # Calendar primitive
```

**Input variants:**
- default: Normal state
- error: Red border + error message
- disabled: Opacity 50%, no pointer events

**Combobox features:**
- Single select với search
- Clear button
- Empty state message
- Keyboard navigation

**Multi-select features:**
- Tag chips cho selected items
- Remove individual tags
- Search/filter options
- Max selection limit (optional)

**FileUploader features:**
- Drag-drop zone
- Click to browse
- Preview thumbnails
- Progress indicator
- Multiple files support
- File type validation

---

### Phase 3: Issue #10 - Data Table (~4h)

**Dependencies cần install:**
```bash
pnpm add @tanstack/react-table
```

**Files tạo mới:**
```
apps/web/src/components/data-table/data-table.tsx
apps/web/src/components/data-table/data-table-column-header.tsx
apps/web/src/components/data-table/data-table-pagination.tsx
apps/web/src/components/data-table/data-table-toolbar.tsx
apps/web/src/components/data-table/data-table-row-actions.tsx
apps/web/src/components/data-table/data-table-empty-state.tsx
apps/web/src/components/data-table/index.ts
```

**Features:**
| Feature | Description |
|---------|-------------|
| Sorting | Click header to sort asc/desc |
| Filtering | Per-column filter inputs |
| Pagination | Page size selector, prev/next |
| Row selection | Checkbox column, select all |
| Empty state | Custom message khi no data |
| Loading | Skeleton rows |

**Component API:**
```tsx
<DataTable
  columns={orderColumns}
  data={orders}
  pagination
  searchKey="customerName"
  onRowClick={handleRowClick}
  emptyMessage="No orders found"
/>
```

---

## Dependencies Summary

```json
{
  "dependencies": {
    "react-hook-form": "^7.x",
    "@hookform/resolvers": "^3.x",
    "zod": "^3.x",
    "cmdk": "^1.x",
    "react-dropzone": "^14.x",
    "date-fns": "^3.x",
    "react-day-picker": "^8.x",
    "@tanstack/react-table": "^8.x"
  }
}
```

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| cmdk styling conflicts | Medium | Scope styles với data attributes |
| TanStack Table learning curve | Low | Follow shadcn/ui examples |
| FileUploader S3 integration | Medium | Chỉ làm UI, backend integration sau |
| Bundle size increase | Low | Tree-shaking, dynamic imports |

## Success Metrics

- [ ] All components render without errors
- [ ] Keyboard navigation works
- [ ] Form validation displays errors correctly
- [ ] Table sorting/filtering works
- [ ] FileUploader accepts drag-drop
- [ ] All components pass a11y basic checks

## Next Steps

1. Confirm approach với user
2. Tạo implementation plan chi tiết cho từng issue
3. Implement theo thứ tự #7 → #9 → #10

---

## Resolved Questions

1. ✅ **FileUploader preview:** Chỉ images (không preview PDF, docs, etc.)
2. ✅ **Table server-side:** Cần server-side pagination ngay từ đầu với mock API data
3. ✅ **Form styles:** Thử apply glass effect cho form inputs
