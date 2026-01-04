# Phase 00: Fix TODO Tasks Workspace

## Context

- Parent: [plan.md](plan.md)
- Dependencies: None
- Docs: [codebase-summary.md](../../docs/codebase-summary.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P1 |
| Effort | 5m |
| Implementation Status | Done |
| Review Status | Done |

**Objective:** Fix TODO tasks không có workspaceId, khiến chúng không hiển thị trong Kanban board khi filter theo workspace.

## Key Insights

1. TODO tasks hiện có `workspaceId: undefined` (line 27, 34)
2. Kanban filter theo workspace → TODO không hiển thị
3. Logic cũ: "TODO = backlog (không thuộc workspace)"
4. Logic mới: "Tất cả tasks thuộc main workspace"

## Requirements

### Fix workspaceId Assignment

- Remove `hasWorkspace` condition
- All tasks get `WORKSPACE_IDS.main`

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `mocks/data/tasks.data.ts` | Modify | Remove hasWorkspace, assign main to all |

## Implementation Steps

### Step 1: Update tasks.data.ts

**Before:**
```typescript
// Line 27
const hasWorkspace = status !== 'TODO';

// Line 34
workspaceId: hasWorkspace ? WORKSPACE_IDS.main : undefined,
```

**After:**
```typescript
// Remove hasWorkspace variable
// Line 34
workspaceId: WORKSPACE_IDS.main,
```

## Todo List

- [x] Remove hasWorkspace condition from tasks.data.ts
- [x] Verify TypeScript compiles
- [ ] Verify TODO tasks appear in Kanban (manual test)

## Success Criteria

- [x] All 15 TODO tasks have workspaceId = WORKSPACE_IDS.main
- [ ] TODO column shows in Kanban board (manual test)
- [x] TypeScript compiles without errors

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Breaking existing filters | Low | Main workspace is default anyway |

## Security Considerations

None - mock data only.
