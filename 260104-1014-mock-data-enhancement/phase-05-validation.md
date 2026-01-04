# Phase 05: Validation & Testing

## Context

- Parent: [plan.md](plan.md)
- Dependencies: Phase 00-03
- Docs: [codebase-summary.md](../../docs/codebase-summary.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P2 |
| Effort | 30m |
| Implementation Status | Pending |
| Review Status | Pending |

**Objective:** Verify mock data integrity, relationships, và UI rendering.

## Key Insights

1. Build check ensures TypeScript correctness
2. Relationship validation prevents runtime errors
3. UI testing confirms data displays correctly

## Requirements

### Build Verification

- `pnpm build:web` passes
- No TypeScript errors
- No console warnings

### Relationship Validation

- All taskIds reference valid tasks
- All userIds reference valid users
- All designVersionIds (if present) reference valid versions

### UI Testing

- Open TaskDetailDrawer with TODO task
- Verify all sections render
- Check Comments section shows comments
- Check Timeline shows COMMENT_ADDED events

## Implementation Steps

### Step 1: TypeScript Build

```bash
cd apps/web && pnpm build
```

### Step 2: Add validation helper (optional)

```typescript
// In mock-factory.ts or separate validation file
export function validateMockData() {
  const taskIds = new Set(mockTasks.map(t => t.id));
  const userIds = new Set(mockUsers.map(u => u.id));

  // Validate comments
  mockComments.forEach(c => {
    console.assert(taskIds.has(c.taskId), `Invalid taskId: ${c.taskId}`);
    console.assert(userIds.has(c.userId), `Invalid userId: ${c.userId}`);
  });

  // Validate reference files
  mockReferenceFiles.forEach(r => {
    console.assert(taskIds.has(r.taskId), `Invalid taskId: ${r.taskId}`);
  });

  console.log('Mock data validation passed!');
}
```

### Step 3: UI Manual Testing

1. Run dev server: `pnpm dev:web`
2. Navigate to Design Kanban
3. Click TODO task card → Opens drawer
4. Verify sections:
   - Task Info: Shows correct data
   - Print Specs: Shows specs from order item
   - Reference Files: Shows files (if any)
   - Comments: Shows comments (without version links)
   - Timeline: Shows events including COMMENT_ADDED

## Todo List

- [ ] Run TypeScript build
- [ ] Fix any compilation errors
- [ ] Manual UI testing with TODO task
- [ ] Verify all drawer sections render
- [ ] Check Console for errors

## Success Criteria

- [ ] Build passes without errors
- [ ] No runtime errors in console
- [ ] TODO task drawer shows all sections
- [ ] Comments render correctly
- [ ] Timeline shows COMMENT_ADDED events

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Build failure | High | Fix incrementally |
| UI errors | Medium | Check console, fix data |

## Security Considerations

None - mock data for development only.
