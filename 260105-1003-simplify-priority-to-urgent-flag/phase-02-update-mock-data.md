# Phase 02: Update Mock Data & Handlers

## Context

- **Parent Plan:** [plan.md](./plan.md)
- **Dependencies:** [Phase 01](./phase-01-update-types-constants.md)
- **Related Docs:** [Brainstorm Report](../reports/brainstorm-260105-1003-simplify-priority-to-urgent-flag.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-05 |
| Priority | P1 |
| Status | ✅ Done |
| Review | ✅ Completed |
| Effort | 30m |
| Completed | 2026-01-05 |

**Description:** Cập nhật mock data generators và MSW handlers để dùng isUrgent boolean.

## Related Files

| File | Change Type |
|------|-------------|
| `apps/web/src/mocks/data/tasks.data.ts` | UPDATE generator |
| `apps/web/src/mocks/handlers/workspaces.ts` | UPDATE bulk action handler |

## Implementation Steps

### Step 1: Update `src/mocks/data/tasks.data.ts`

```typescript
// BEFORE (line 83)
const priorities = ['LOW', 'NORMAL', 'HIGH', 'URGENT'];
// ...
priority: priorities[idx % 4]!,

// AFTER
// Xóa priorities array
// ...
isUrgent: idx % 5 === 0, // 20% tasks urgent
```

### Step 2: Update `src/mocks/handlers/workspaces.ts`

```typescript
// BEFORE (line 107-125)
// POST /api/tasks/bulk/set-priority
http.post('/api/tasks/bulk/set-priority', async ({ request }) => {
  const body = await request.json() as {
    taskIds: string[];
    priority: PriorityType;
  };
  // ...
});

// AFTER
// POST /api/tasks/bulk/toggle-urgent
http.post('/api/tasks/bulk/toggle-urgent', async ({ request }) => {
  const body = await request.json() as {
    taskIds: string[];
    isUrgent: boolean;
  };
  // ...update tasks với isUrgent
});
```

## Todo List

- [ ] Update `tasks.data.ts` - đổi priority → isUrgent trong generator
- [ ] Update `workspaces.ts` - đổi set-priority endpoint → toggle-urgent
- [ ] Verify mock data structure

## Success Criteria

- [ ] Mock tasks có `isUrgent: boolean` thay vì `priority: PriorityType`
- [ ] Bulk action endpoint hoạt động với isUrgent
- [ ] MSW không throw errors

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Stale data in browser | Low | Low | Clear localStorage/cache |

## Security Considerations

None - mock data only.

## Next Steps

After completion → Phase 03: Update Components
