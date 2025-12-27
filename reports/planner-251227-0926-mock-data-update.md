# Planner Report: MSW Mock Data Update

**Date:** 2025-12-27 | **Duration:** ~15min

---

## Summary

Đã tạo implementation plan chi tiết cho việc update MSW mock data theo `docs/synkao-models-spec.md` v1.2.

## Plan Structure

```
plans/251227-0855-mock-data-update/
├── plan.md                      # Overview + phase links
├── phase-01-types-and-factory.md  # Types + Factory (45min)
├── phase-02-data-generation.md    # Data files (1h)
├── phase-03-handlers.md           # CRUD + Validation (1.5h)
└── phase-04-verification.md       # Testing (45min)
```

**Total Effort:** 4 hours

## Key Deliverables

### Phase 01: Types & Factory
- 9 interfaces (thêm MockTenant, MockColumn)
- Status enums mới (Order 7, Task 7, Design 4)
- Value objects: Address (US), Money (USD cents)
- Backward-compatible aliases cho deprecated types

### Phase 02: Data Generation
- NEW: `tenants.data.ts` - 1 tenant
- NEW: `columns.data.ts` - 10 columns (5/workspace)
- UPDATE: 6 existing files với full fields
- Data counts: 15 orders, 45 items, ~35 tasks, ~20 designs

### Phase 03: Handlers
- NEW: `utils.ts` - state machine + error helpers
- NEW: `workspaces.ts` - Workspace/Column CRUD
- NEW: `designs.ts` - upload/review flow
- UPDATE: `tasks.ts` - Kanban ops, optimistic lock
- UPDATE: `orders.ts` - status transitions

### Phase 04: Verification
- TypeScript compilation check
- Storybook stories verification
- API response validation
- Relationship integrity check

## Critical Patterns Applied

| Pattern | Implementation |
|---------|---------------|
| State Machine | `canTransition()` function + transition maps |
| Optimistic Lock | `version` field + 412 Conflict response |
| Multi-tenant | `tenantId` required on all entities |
| CRUD Validation | Status-specific rules (e.g., SHIPPED needs tracking) |

## File Changes Summary

| Action | Count | Files |
|--------|-------|-------|
| NEW | 5 | tenants.data, columns.data, utils.ts, workspaces.ts, designs.ts |
| UPDATE | 9 | types.ts, mock-factory.ts, 6 data files, 2 handlers |
| RENAME | 1 | design-versions.data → designs.data |

## Risk Mitigation

- Backward-compatible type aliases
- Optional fields cho new properties
- Incremental by layer approach
- Storybook verification after each phase

---

## Plan File Paths

- Main: `/home/tuntran/application/github.com/synkao/synkao/plans/251227-0855-mock-data-update/plan.md`
- Phase 01: `/home/tuntran/application/github.com/synkao/synkao/plans/251227-0855-mock-data-update/phase-01-types-and-factory.md`
- Phase 02: `/home/tuntran/application/github.com/synkao/synkao/plans/251227-0855-mock-data-update/phase-02-data-generation.md`
- Phase 03: `/home/tuntran/application/github.com/synkao/synkao/plans/251227-0855-mock-data-update/phase-03-handlers.md`
- Phase 04: `/home/tuntran/application/github.com/synkao/synkao/plans/251227-0855-mock-data-update/phase-04-verification.md`
