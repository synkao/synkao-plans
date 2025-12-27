---
title: "MSW Mock Data Update - Align với synkao-models-spec v1.2"
description: "Cập nhật mock data structure để support multi-tenant, full CRUD handlers với validation và state machine"
status: pending
priority: P1
effort: 4h
branch: main
tags: [msw, mock-data, multi-tenant, crud]
created: 2025-12-27
---

# MSW Mock Data Update Plan

## Objective

Align mock data với `docs/synkao-models-spec.md` v1.2 để support 4 features: Kanban Board, Order Management, Design Review Flow, Backlog Management.

## Key Decisions

| Aspect | Decision |
|--------|----------|
| Region | International (US) - addresses, USD, USPS/FedEx/UPS |
| Approach | Incremental by Layer (4 phases) |
| Multi-tenant | tenantId required on all entities |
| Optimistic Locking | version field + 412 Conflict response |

## Phase Overview

| Phase | File | Focus | Effort |
|-------|------|-------|--------|
| 01 | [phase-01-types-and-factory.md](./phase-01-types-and-factory.md) | Types + Factory | 45min |
| 02 | [phase-02-data-generation.md](./phase-02-data-generation.md) | Data files | 1h |
| 03 | [phase-03-handlers.md](./phase-03-handlers.md) | CRUD + Validation | 1.5h |
| 04 | [phase-04-verification.md](./phase-04-verification.md) | Testing | 45min |

## Files Summary

**Update:**
- `mocks/types.ts` - 9 interfaces (thêm MockTenant, MockColumn, update 5 existing)
- `mocks/factories/mock-factory.ts` - thêm tenantUUID, columnUUID, address/money generators
- `mocks/data/*.data.ts` - 8 files (thêm tenants, columns; update 6 existing)
- `mocks/handlers/*.ts` - 4 handlers (update tasks, orders; thêm workspaces, designs)

**New Files:**
- `mocks/data/tenants.data.ts`
- `mocks/data/columns.data.ts`
- `mocks/handlers/workspaces.ts`
- `mocks/handlers/designs.ts`

## Status Enum Migrations

```
Order:    PENDING → CONFIRMED → IN_DESIGN → READY_TO_FULFILL → SHIPPED → COMPLETED → CANCELLED
Task:     PENDING → TODO → IN_PROGRESS → REVIEW → REVISION → APPROVED → BLOCKED
Design:   DRAFT → SUBMITTED → APPROVED → REJECTED
```

## Dependency Order

```
Phase 01 (types.ts, factory)
    ↓
Phase 02 (data files)
    ↓
Phase 03 (handlers)
    ↓
Phase 04 (verification)
```

## Risk Mitigation

- Keep backward-compatible field names where possible
- Add new fields as optional initially
- Run Storybook after each phase to catch breaks early

---

## Validation Summary

**Validated:** 2025-12-27
**Questions asked:** 4

### Confirmed Decisions

| Decision | User Choice | Impact |
|----------|-------------|--------|
| Type aliases | **Bỏ aliases, rename trực tiếp** | Phase 01: Remove MockTask, MockDesignVersion aliases. Update all imports. |
| Multi-tenant header | **Required - throw error** | Phase 03: All handlers require `x-tenant-id`, return 401 nếu thiếu |
| Optimistic locking | **Optional** | Phase 03: Check version chỉ khi client gửi trong body |
| File upload | **Presigned URL flow** | Phase 03: 2-step upload (GET presigned → PUT to mock S3) |

### Action Items (Plan Revision Needed)

- [ ] Phase 01: Remove backward compatibility aliases (MockTask, MockDesignVersion)
- [ ] Phase 03: Add mandatory tenant header check với 401 response
- [ ] Phase 03: Add presigned URL endpoint cho design upload (`POST /api/uploads/init`, `POST /api/uploads/complete`)
