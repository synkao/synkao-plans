---
phase: 2
title: "Update Auth Handlers"
status: pending
effort: 30m
---

# Phase 02: Update Auth Handlers

## Overview

Update auth handlers to use `/api/v1/auth/*` URLs and new response format.

## Context

**Convention Reference:** `docs/frontend/mock-api-convention.md` Section 5.1

## Related Code Files

- **Modify:** `apps/web/src/mocks/handlers/auth.ts`

## URL Mapping

| Current | New |
|---------|-----|
| `POST /api/auth/login` | `POST /api/v1/auth/login` |
| `POST /api/auth/logout` | `POST /api/v1/auth/logout` |
| `POST /api/auth/refresh` | `POST /api/v1/auth/refresh` |
| `GET /api/auth/me` | `GET /api/v1/auth/me` |
| `GET /api/users` | `GET /api/v1/users/` |
| `GET /api/users/:id` | `GET /api/v1/users/:id` |

## Implementation Steps

### 1. Update auth.ts imports

```typescript
import { http, HttpResponse } from 'msw';
import { mockUsers, findUser } from '../data';
import { extractContext } from './context';
import {
  successResponse,
  listResponse,
  actionResponse,
  parsePagination,
  getPaginationMeta,
  paginateArray,
  errors
} from '../utils';
```

### 2. Update login handler

```typescript
// POST /api/v1/auth/login
http.post('/api/v1/auth/login', async ({ request }) => {
  const body = await request.json() as { email: string; password: string };

  if (!body.email || !body.password) {
    return errors.invalidCredentials();
  }

  const user = mockUsers.find((u) => u.email === body.email) ?? defaultUser;
  const token = `mock-jwt-${user.id}`;

  return successResponse({ token, user });
}),
```

### 3. Update logout handler

```typescript
// POST /api/v1/auth/logout
http.post('/api/v1/auth/logout', () => {
  return actionResponse('Logged out successfully');
}),
```

### 4. Update refresh handler

```typescript
// POST /api/v1/auth/refresh
http.post('/api/v1/auth/refresh', ({ request }) => {
  const ctx = extractContext(request);
  if (!ctx.userId) {
    return errors.unauthorized();
  }
  return successResponse({ token: `mock-jwt-${ctx.userId}` });
}),
```

### 5. Update me handler

```typescript
// GET /api/v1/auth/me
http.get('/api/v1/auth/me', ({ request }) => {
  const ctx = extractContext(request);

  if (!ctx.userId) {
    return errors.unauthorized();
  }

  const user = findUser(ctx.userId);
  if (!user) {
    return errors.unauthorized();
  }

  return successResponse(user);
}),
```

### 6. Update users list handler

```typescript
// GET /api/v1/users/
http.get('/api/v1/users/', ({ request }) => {
  const url = new URL(request.url);
  const { page, limit } = parsePagination(url);
  const role = url.searchParams.get('role');

  let filtered = mockUsers;
  if (role) {
    filtered = mockUsers.filter(u => u.role === role);
  }

  const paginated = paginateArray(filtered, page, limit);
  const pagination = getPaginationMeta(filtered.length, page, limit);

  return listResponse(paginated, pagination);
}),
```

### 7. Update user by id handler

```typescript
// GET /api/v1/users/:id
http.get('/api/v1/users/:id', ({ params }) => {
  const user = findUser(params.id as string);
  if (!user) {
    return errors.notFound('User');
  }
  return successResponse(user);
}),
```

## Todo List

- [ ] Import response helpers
- [ ] Update all URLs to `/api/v1/auth/*` and `/api/v1/users/*`
- [ ] Wrap single responses in `{ data: ... }`
- [ ] Add pagination to users list
- [ ] Use error helpers for errors
- [ ] Test handlers work

## Success Criteria

- [ ] All auth endpoints use `/api/v1/auth/*` URL
- [ ] Login returns `{ data: { token, user } }`
- [ ] Me returns `{ data: User }`
- [ ] Users list returns `{ data: [], pagination }`
- [ ] Errors return `{ error: { code, message } }`
