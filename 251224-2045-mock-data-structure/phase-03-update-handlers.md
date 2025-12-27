# Phase 03: Update Handlers

## Context Links
- Parent: [plan.md](./plan.md)
- Dependency: [phase-02-create-data-files.md](./phase-02-create-data-files.md)

## Overview
- **Priority:** High
- **Status:** Pending
- **Effort:** 30m
- **Description:** Update existing handlers để dùng imported data thay vì inline

## Key Insights
- Handlers đã có structure tốt, chỉ cần swap data source
- Keep handler logic unchanged
- Import data từ `../data`

## Requirements

### Functional
- auth.ts: Dùng mockUsers thay vì inline mockUser
- tasks.ts: Dùng mockTasks từ data file
- orders.ts: Dùng mockOrders từ data file
- (Optional) Add new endpoints cho order-items, design-versions

### Non-functional
- Maintain existing API contracts
- No breaking changes

## Related Code Files

| Action | File | Description |
|--------|------|-------------|
| Modify | `apps/web/src/mocks/handlers/auth.ts` | Import users data |
| Modify | `apps/web/src/mocks/handlers/tasks.ts` | Import tasks data |
| Modify | `apps/web/src/mocks/handlers/orders.ts` | Import orders data |

## Implementation Steps

### Step 1: Update auth.ts
```typescript
// Before
const mockUser: MockUser = { id: '1', ... };

// After
import { mockUsers } from '../data';
const defaultUser = mockUsers.find(u => u.role === 'DESIGNER')!;

// In login handler, can validate against mockUsers
```

### Step 2: Update tasks.ts
```typescript
// Before
const mockTasks: MockTask[] = [ ... inline data ... ];

// After
import { mockTasks } from '../data';

// Logic remains the same
```

### Step 3: Update orders.ts
```typescript
// Before
const mockOrders: MockOrder[] = [ ... inline data ... ];

// After
import { mockOrders, mockOrderItems } from '../data';

// Optional: Add order items endpoint
http.get('/api/orders/:id/items', ({ params }) => {
  const items = mockOrderItems.filter(i => i.orderId === params.id);
  return HttpResponse.json({ items });
});
```

### Step 4: (Optional) Add workspaces.ts handler
```typescript
// apps/web/src/mocks/handlers/workspaces.ts
import { http, HttpResponse } from 'msw';
import { mockWorkspaces, mockTasks, defaultColumns } from '../data';

export const workspacesHandlers = [
  http.get('/api/workspaces', () => {
    return HttpResponse.json({ workspaces: mockWorkspaces });
  }),

  http.get('/api/workspaces/:id', ({ params }) => {
    const workspace = mockWorkspaces.find(w => w.id === params.id);
    if (!workspace) {
      return HttpResponse.json({ error: 'Not found' }, { status: 404 });
    }
    return HttpResponse.json(workspace);
  }),

  // Board endpoint already in tasks.ts, can move here
];
```

### Step 5: Update handlers/index.ts
```typescript
import { authHandlers } from './auth';
import { tasksHandlers } from './tasks';
import { ordersHandlers } from './orders';
// import { workspacesHandlers } from './workspaces'; // if created

export const handlers = [
  ...authHandlers,
  ...tasksHandlers,
  ...ordersHandlers,
  // ...workspacesHandlers,
];
```

## Todo List

- [ ] Update auth.ts để import mockUsers
- [ ] Update tasks.ts để import mockTasks
- [ ] Update orders.ts để import mockOrders
- [ ] (Optional) Create workspaces.ts handler
- [ ] Update handlers/index.ts
- [ ] Test MSW hoạt động với browser

## Success Criteria

- [ ] No inline mock data trong handlers
- [ ] All existing endpoints vẫn hoạt động
- [ ] No TypeScript errors
- [ ] MSW intercepts requests correctly

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Breaking existing endpoints | High | Keep same response structure |
| Import path errors | Low | Use consistent relative paths |

## Security Considerations
- N/A (mock data only)

## Testing

### Manual Testing
1. Start dev server: `pnpm dev`
2. Open browser DevTools → Network
3. Verify API calls return mock data
4. Check Kanban board loads với 45 tasks

### Verification Checklist
- [ ] `/api/auth/me` returns user from mockUsers
- [ ] `/api/tasks` returns 45 tasks
- [ ] `/api/orders` returns 15 orders
- [ ] `/api/workspaces/:id/board` shows 5 columns với tasks

## Next Steps
- Test với browser
- Close Issue #4
