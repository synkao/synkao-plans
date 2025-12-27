---
title: "Phase 04: Testing & Verification"
status: pending
priority: P1
effort: 45min
---

# Phase 04: Testing & Verification

**Parent:** [plan.md](./plan.md)
**Dependencies:** Phase 03 (handlers)

## Overview

Verify mock data integration hoạt động đúng, không breaking existing UI components.

## Requirements

1. TypeScript compilation passes
2. Storybook stories render without errors
3. Console không có error messages
4. API responses match expected structure

## Related Files

- All files from Phase 01-03
- Storybook stories trong `apps/web/src/**/*.stories.tsx`
- Components sử dụng mock data

## Implementation Steps

### Step 4.1: TypeScript Compilation Check

```bash
# From apps/web directory
cd apps/web
pnpm typecheck
```

**Expected:** No errors

**If errors occur:**
- Check import paths
- Verify type exports from types.ts
- Fix any missing optional fields

### Step 4.2: Run Storybook

```bash
# From apps/web directory
pnpm storybook
```

**Check các stories:**
- [ ] KanbanBoard stories load correctly
- [ ] TaskCard stories render with new data structure
- [ ] OrderList stories show orders with addresses
- [ ] DesignReview stories work with MockDesign

### Step 4.3: Console Error Audit

Trong Storybook, open DevTools Console và check:

**Expected:** No red errors related to:
- Missing properties on mock data
- Type mismatches
- Failed API calls

**Common issues to look for:**
- `undefined is not an object` - missing optional field
- `Cannot read property 'x' of undefined` - missing relationship
- `404 Not Found` - handler path mismatch

### Step 4.4: API Response Verification

Test các endpoints manually trong browser console hoặc Storybook:

```javascript
// Test order list
fetch('/api/orders').then(r => r.json()).then(console.log)
// Expected: { orders: [...], total: 15 }

// Test order with address
fetch('/api/orders/00000001-0001-4000-8000-000000000001').then(r => r.json()).then(console.log)
// Expected: Order object with shippingAddress

// Test kanban board
fetch('/api/workspaces/11111111-1111-4111-8111-111111111111/board').then(r => r.json()).then(console.log)
// Expected: { columns: [{id, name, status, tasks: [...]}] }

// Test status transition (should fail - invalid)
fetch('/api/tasks/00000003-0001-4000-8000-000000000001', {
  method: 'PATCH',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ status: 'APPROVED', version: 1 })
}).then(r => r.json()).then(console.log)
// Expected: 400 error if current status doesn't allow APPROVED

// Test optimistic lock conflict
fetch('/api/tasks/00000003-0001-4000-8000-000000000001', {
  method: 'PATCH',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ priority: 'HIGH', version: 0 }) // wrong version
}).then(r => r.json()).then(console.log)
// Expected: 412 Precondition Failed
```

### Step 4.5: Data Relationship Integrity Check

```javascript
// Verify all task orderIds point to valid orders
const { tasks } = await fetch('/api/tasks').then(r => r.json());
const { orders } = await fetch('/api/orders').then(r => r.json());
const orderIds = new Set(orders.map(o => o.id));
const orphanTasks = tasks.filter(t => !orderIds.has(t.orderId));
console.log('Orphan tasks:', orphanTasks.length); // Expected: 0

// Verify all designs point to valid tasks
const taskIds = new Set(tasks.map(t => t.id));
// Note: Need to check designs data directly or via task endpoint
```

### Step 4.6: Backward Compatibility Check

Verify deprecated type aliases work:

```typescript
// In any component file, test imports
import type { MockTask, MockDesignVersion } from '@/mocks/types';

// These should work (aliases to new types)
const task: MockTask = { /* ... */ };
const design: MockDesignVersion = { /* ... */ };
```

### Step 4.7: Create Verification Checklist

**Manual verification checklist:**

| Check | Status | Notes |
|-------|--------|-------|
| `pnpm typecheck` passes | [ ] | |
| Storybook starts without build errors | [ ] | |
| KanbanBoard renders columns | [ ] | |
| Tasks show in correct columns | [ ] | |
| Order list shows 15 orders | [ ] | |
| Orders have addresses | [ ] | |
| Task drag/drop works | [ ] | |
| Status transitions validate | [ ] | |
| Version conflict returns 412 | [ ] | |
| No console errors | [ ] | |

### Step 4.8: Document Known Issues

Nếu có issues không thể fix ngay, document tại đây:

```markdown
## Known Issues

| Issue | Severity | Workaround | Ticket |
|-------|----------|------------|--------|
| Example issue | LOW | Temporary fix | #123 |
```

### Step 4.9: Update Codebase Summary

Sau khi hoàn thành, update `docs/codebase-summary.md` section về MSW mock data:

```markdown
## Mock Service Worker (MSW)

### Data Structure (v1.2)
- 9 entity types: Tenant, User, Workspace, Column, Order, OrderItem, DesignTask, Design
- Multi-tenant support via tenantId
- Optimistic locking via version field
- Full CRUD handlers với state machine validation

### Handlers
- `/api/tasks` - Task CRUD + Kanban operations
- `/api/orders` - Order CRUD + status transitions
- `/api/workspaces` - Workspace + Column management
- `/api/designs` - Design upload/review flow
```

## Success Criteria

- [ ] All 10 verification checks pass
- [ ] No TypeScript errors
- [ ] No console errors in Storybook
- [ ] All stories render correctly
- [ ] API responses match spec

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Breaking existing components | HIGH | Check all stories before merge |
| Type mismatch in components | MEDIUM | Fix components to use new types |
| Missing handler routes | LOW | Add as needed based on component usage |

## Post-Implementation

After verification passes:
1. Commit changes with message: `feat(msw): update mock data to spec v1.2`
2. Update any documentation references
3. Notify team of new mock data structure
