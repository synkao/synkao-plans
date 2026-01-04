---
title: "Core UI Components P0"
description: "Glass Card, Form Components, Data Table - 3 core components cho Synkao POD"
status: completed
priority: P0
effort: 13h
branch: main
tags: [ui, components, frontend, glass-effect, forms, data-table]
created: 2026-01-04
completed: 2026-01-04
---

# Core UI Components P0

## Context
- **Issues:** #7, #9, #10
- **Brainstorm:** [brainstorm-260104-1439-core-ui-components-p0.md](../reports/brainstorm-260104-1439-core-ui-components-p0.md)
- **Codebase:** apps/web/src/components/ui/

## Overview

Implement 3 P0 components cần thiết cho POD management system:
1. **Glass Card** (#7) - Card với glass morphism effect
2. **Form Components** (#9) - Form inputs với validation
3. **Data Table** (#10) - Generic table với TanStack Table

## Phases

| Phase | Issue | Component | Effort | Dependencies |
|-------|-------|-----------|--------|--------------|
| 1 | #7 | Glass Card | 3h | None |
| 2 | #9 | Form Components | 4h | cmdk, react-dropzone, date-fns |
| 3 | #10 | Data Table | 4h | @tanstack/react-table |
| 4 | - | Demo Components | 2h | Phase 1-3 complete |

## Dependencies to Install

```bash
pnpm add -F @synkao/web cmdk react-dropzone date-fns react-day-picker @tanstack/react-table
```

> Note: react-hook-form, @hookform/resolvers, zod đã có sẵn

## File Structure

```
apps/web/src/components/
├── ui/
│   ├── glass-card.tsx          # Phase 1
│   ├── combobox.tsx            # Phase 2
│   ├── multi-select.tsx        # Phase 2
│   ├── file-uploader.tsx       # Phase 2
│   ├── date-picker.tsx         # Phase 2
│   └── calendar.tsx            # Phase 2
└── data-table/                 # Phase 3
    ├── data-table.tsx
    ├── data-table-column-header.tsx
    ├── data-table-pagination.tsx
    ├── data-table-toolbar.tsx
    ├── data-table-row-actions.tsx
    ├── data-table-empty-state.tsx
    └── index.ts
```

## Success Criteria

- [x] Glass Card renders với 4 variants
- [x] Form validation hiển thị errors đúng
- [x] Combobox keyboard navigation works
- [x] FileUploader drag-drop functional
- [x] DatePicker calendar opens/selects
- [x] DataTable sort/filter/paginate works
- [x] All components accessible (a11y)
- [x] Demo page showcase tất cả components

## Detailed Phase Plans

1. [Phase 1: Glass Card](./phase-01-glass-card.md)
2. [Phase 2: Form Components](./phase-02-form-components.md)
3. [Phase 3: Data Table](./phase-03-data-table.md)
4. [Phase 4: Demo Components](./phase-04-demo-components.md)

## Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| cmdk styling conflicts | Medium | Scope với data attributes |
| Bundle size increase | Low | Tree-shaking, dynamic imports |
| FileUploader S3 integration | Medium | UI only, backend sau |

## Next Steps

1. Review plan với team
2. Implement Phase 1-3 (Components)
3. Implement Phase 4 (Demo showcase)
4. Final review trên design-system page
