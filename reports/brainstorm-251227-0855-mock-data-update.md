# Brainstorm: Mock Data Update theo synkao-models-spec.md

**Date:** 2024-12-27
**Scope:** Incremental (4 features) + Multi-tenant + Full CRUD
**Features:** Kanban Board, Order Management, Design Review Flow, Backlog Management

---

## 1. Problem Statement

Mock data hiện tại (`apps/web/src/mocks/`) không align với spec mới (`docs/synkao-models-spec.md` v1.2). Cần cập nhật để:
- Support multi-tenant với `tenantId`
- Align status values với state machines trong spec
- Thêm fields thiếu cho 4 features đang develop
- Full CRUD handlers với validation và state machine enforcement

---

## 2. GAP Analysis

### 2.1 Entity-by-Entity Comparison

| Entity | Fields Hiện Tại | Fields Theo Spec | GAP |
|--------|----------------|------------------|-----|
| **User** | 5 fields | 11 fields | +tenantId, phone, status, lastLoginAt, timestamps |
| **Order** | 6 fields | 25 fields | **Thiếu 19 fields** (address, money, tracking, designStatus...) |
| **OrderItem** | 7 fields | 15 fields | +externalId, variantName, money, images, needsDesign, designStatus |
| **DesignTask** | 12 fields | 26 fields | +workspaceId, columnId, position, title, description, review fields, version |
| **Design** | 7 fields | 14 fields | +taskId→designId, fileType, fileSize, metadata, reviewNotes |
| **Workspace** | 4 fields | 9 fields | +tenantId, description, color, icon, isDefault, sortOrder |
| **Column** | 2 fields | 8 fields | +workspaceId, status, color, wipLimit, isSystem, timestamps |

### 2.2 Missing Entities

| Entity | Priority | Reason |
|--------|----------|--------|
| **Tenant** | HIGH | Multi-tenant requirement |
| **Column** (proper) | HIGH | Kanban positioning |
| **Event** | LOW | Audit log - defer |

### 2.3 Status Enum Changes

**Order Status:**
```
Hiện tại: PENDING → PROCESSING → SHIPPED → DELIVERED → FAILED
Spec:     PENDING → CONFIRMED → IN_DESIGN → READY_TO_FULFILL → SHIPPED → COMPLETED → CANCELLED
```

**Task Status:**
```
Hiện tại: TODO → IN_PROGRESS → REVIEW → REVISION → DONE
Spec:     PENDING → TODO → IN_PROGRESS → REVIEW → REVISION → APPROVED → BLOCKED
```

---

## 3. Proposed Approach

### Option A: Big Bang Update (NOT Recommended)
- Thay đổi tất cả entities cùng lúc
- **Cons:** Risk cao, khó debug, breaking changes nhiều

### Option B: Incremental by Layer (Recommended)
1. **Layer 1: Types & Factory** - Update types.ts, thêm factory mới
2. **Layer 2: Data Generation** - Update data files theo types mới
3. **Layer 3: Handlers** - Update/thêm handlers với validation
4. **Layer 4: Test & Fix** - Verify không breaking existing UI

### Recommended: Option B

---

## 4. Implementation Plan (Option B)

### Phase 1: Foundation (Types + Factory)

**Files cần thay đổi:**
- `mocks/types.ts` → Thêm/update types theo spec
- `mocks/factories/mock-factory.ts` → Thêm generators mới

**New Types cần thêm:**
```typescript
// Multi-tenant
interface MockTenant { id, name, slug, plan, status, settings, timestamps }

// Updated Order với full fields
interface MockOrder { ...25 fields từ spec }

// Updated DesignTask với Kanban support
interface MockDesignTask { ...26 fields từ spec }

// Proper Column entity
interface MockColumn { id, workspaceId, name, status, color, sortOrder, wipLimit }

// Updated Design (renamed từ DesignVersion)
interface MockDesign { ...14 fields }
```

### Phase 2: Data Generation

**Files cần thay đổi:**
- `mocks/data/tenants.data.ts` (NEW)
- `mocks/data/orders.data.ts` → Full fields
- `mocks/data/order-items.data.ts` → +money, images
- `mocks/data/tasks.data.ts` → +position, column, review
- `mocks/data/workspaces.data.ts` → +tenantId, columns
- `mocks/data/columns.data.ts` (NEW)
- `mocks/data/design-versions.data.ts` → Rename to designs.data.ts

**Data Distribution Suggestion:**
| Entity | Count | Logic |
|--------|-------|-------|
| Tenant | 1 | Single tenant for MVP mock |
| User | 6 (existing) | Add tenantId |
| Workspace | 2 (existing) | Add columns, tenantId |
| Column | 10 (5 per workspace) | System columns |
| Order | 15 (existing) | Add full fields |
| OrderItem | 45 (existing) | Add money, images |
| DesignTask | 45 (existing) | Add position, column assignment |
| Design | 20 (existing) | Add metadata |

### Phase 3: Handlers

**Priority handlers cần update:**
1. `tasks.ts` - Kanban operations (move, reorder)
2. `orders.ts` - Status transitions với validation
3. (NEW) `workspaces.ts` - Workspace CRUD
4. (NEW) `designs.ts` - Design upload/review flow

**Handler Patterns:**
```typescript
// State machine validation
function canTransition(from: Status, to: Status): boolean

// Optimistic locking
function checkVersion(current: number, expected: number): void

// Multi-tenant filter
function filterByTenant(data: T[], tenantId: string): T[]
```

### Phase 4: Verification

- Run existing Storybook stories
- Check console for errors
- Fix any broken data references

---

## 5. Risk Analysis

| Risk | Impact | Mitigation |
|------|--------|------------|
| Breaking existing UI | HIGH | Keep old field names, add new ones |
| Type mismatches | MEDIUM | Careful migration path for enums |
| Data relationship breaks | MEDIUM | Update factory first, then data |
| Handler logic errors | LOW | Add validation tests |

---

## 6. Effort Estimate

| Phase | Estimated |
|-------|-----------|
| Phase 1: Types + Factory | ~2-3 files, nhỏ |
| Phase 2: Data Generation | ~8 files, trung bình |
| Phase 3: Handlers | ~4-5 handlers, lớn nhất |
| Phase 4: Verification | Testing |

---

## 7. Recommended Implementation Order

```
1. types.ts (new types)
2. mock-factory.ts (new generators)
3. tenants.data.ts (new)
4. columns.data.ts (new)
5. workspaces.data.ts (update)
6. orders.data.ts (update)
7. order-items.data.ts (update)
8. tasks.data.ts (update)
9. designs.data.ts (rename + update)
10. tasks.ts handler (update)
11. orders.ts handler (update)
12. workspaces.ts handler (new)
13. designs.ts handler (new)
```

---

## 8. Decisions Made

| Question | Decision |
|----------|----------|
| Address structure | **International (US)** - POD customers là foreigners |
| Currency | **USD** - smallest unit (cents) |
| Carriers | **USPS, FedEx, UPS** - international shipping |
| File URLs | Placeholder URLs với CDN pattern |
| Approach | **Incremental by Layer** - 4 phases |

---

## 9. Next Steps

Nếu đồng ý approach, tạo implementation plan chi tiết với:
- Exact file changes
- Code snippets cho critical updates
- Task breakdown cho từng phase
