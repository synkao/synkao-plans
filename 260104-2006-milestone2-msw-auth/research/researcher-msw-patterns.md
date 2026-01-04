# MSW Mock Data Patterns & Best Practices Research
**Date:** 2026-01-04 | **Scope:** React/Next.js POD Order Management

## Current Structure Analysis

### Existing Setup ✓ Well-Designed
```
apps/web/src/mocks/
├── factories/mock-factory.ts      # ID generators, deterministic UUIDs
├── data/                          # Separated data layers
│   ├── users.data.ts
│   ├── tasks.data.ts
│   ├── orders.data.ts
│   └── [6 more data files]
├── handlers/                      # MSW HTTP handlers
│   ├── auth.ts
│   ├── tasks.ts
│   ├── orders.ts
│   └── index.ts (aggregator)
├── types.ts                       # TypeScript interfaces
├── browser.ts                     # setupWorker
└── server.ts                      # setupServer
```

**Strengths:** Deterministic ID generation, separated concerns, role-based mock users.

---

## Key Recommendations

### 1. Type-Safe Mock Handlers
**Current Issue:** No TypeScript typing for request/response validation at handler level.

**Pattern - Type Factory with Branded Types:**
```typescript
// mocks/factories/types.factory.ts
export const createMockResponse = <T extends {}>(data: T): T => data;

// Usage in handlers
export const authHandlers = [
  http.post<{ email: string; password: string }>('/api/auth/login', async ({ request }) => {
    const body = await request.json();
    const user = validateUserLogin(body);
    return HttpResponse.json(createMockResponse(user));
  }),
];
```

### 2. Multi-Tenant Context via Headers
**For auth testing with different tenants/roles:**

```typescript
// mocks/handlers/context.ts
import { HttpRequest } from 'msw';

export interface RequestContext {
  tenantId?: string;
  userId?: string;
  role?: 'ADMIN' | 'STAFF' | 'DESIGNER';
}

export function extractContext(request: HttpRequest): RequestContext {
  return {
    tenantId: request.headers.get('x-tenant-id') || undefined,
    userId: request.headers.get('x-user-id') || undefined,
    role: request.headers.get('x-user-role') as any,
  };
}

// Usage
http.get('/api/workspace/:id', ({ request, params }) => {
  const ctx = extractContext(request);
  if (!ctx.role || ctx.role === 'DESIGNER') {
    return HttpResponse.json({ error: 'Unauthorized' }, { status: 403 });
  }
  // ...
});
```

### 3. Data Scenario Management
**Current:** Single hardcoded dataset. **Recommendation:** Environment-based scenarios.

```typescript
// mocks/scenarios/index.ts
export const scenarios = {
  'empty': { users: [], tasks: [], orders: [] },
  'standard': { users: mockUsers, tasks: mockTasks, orders: mockOrders },
  'heavy-load': { users: mockUsers, tasks: [...Array(100).keys()].map(i => generateTask(i)) },
};

// mocks/server.ts
const activeScenario = process.env.MSW_SCENARIO || 'standard';
export const activeData = scenarios[activeScenario];
```

### 4. Factory Pattern Enhancement
**Current `mock-factory.ts`:** Good ID generation. **Enhance with:**

```typescript
// mocks/factories/index.ts (comprehensive)
import { faker } from '@faker-js/faker';

export class MockDataBuilder {
  static user(overrides?: Partial<MockUser>) {
    return {
      id: faker.datatype.uuid(),
      email: faker.internet.email(),
      name: faker.name.fullName(),
      role: faker.helpers.arrayElement(['ADMIN', 'STAFF', 'DESIGNER']),
      avatar: `https://api.dicebear.com/9.x/avataaars/svg?seed=${faker.random.word()}`,
      ...overrides,
    };
  }

  static task(workspaceId: string, overrides?: Partial<MockTask>) {
    return {
      id: faker.datatype.uuid(),
      taskCode: taskCode(faker.number.int({ min: 1, max: 999 })),
      status: faker.helpers.arrayElement(['TODO', 'IN_PROGRESS', 'REVIEW']),
      priority: faker.helpers.arrayElement(['LOW', 'NORMAL', 'HIGH', 'URGENT']),
      workspaceId,
      ...overrides,
    };
  }
}
```

### 5. Request Handling Best Practices
**MSW Docs Emphasis:** Avoid direct request assertions; use composable handlers.

✓ **Do:**
```typescript
export const createTaskHandler = (overridePath?: string) =>
  http.post(overridePath || '/api/tasks', async ({ request }) => {
    const body = await request.json();
    const task = mockDataBuilder.task(body.workspaceId, body);
    return HttpResponse.json(task);
  });

// Composable, testable, reusable
```

✗ **Avoid:**
```typescript
// Direct assertions inside handlers
http.post('/api/tasks', async ({ request }) => {
  expect(request.method).toBe('POST'); // Bad pattern
});
```

---

## Implementation Checklist

- [ ] Add context extraction utility (headers-based tenant/user)
- [ ] Create `MockDataBuilder` factory with faker integration
- [ ] Implement scenario-based mock data switching
- [ ] Add request/response type validation at handler level
- [ ] Document auth mock patterns (JWT, refresh tokens)
- [ ] Create reusable handler generators for CRUD operations

---

## References
- MSW Official: https://mswjs.io/docs/best-practices
- MSW HTTP Handlers: https://mswjs.io/docs/api/http
- Mock Data Scenarios: Best practice for testing complex auth flows

## Unresolved Questions
- Should JTW token validation be mocked or real signature checking?
- Multi-workspace filtering strategy: filter at handler or data layer?
- Should rate-limiting be simulated in mock handlers?
