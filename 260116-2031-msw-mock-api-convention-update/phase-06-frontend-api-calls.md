---
phase: 6
title: "Update Frontend API Calls"
status: pending
effort: 45m
---

# Phase 06: Update Frontend API Calls

## Overview

Update all frontend API calls to use new URL patterns and handle new response format.

## Related Code Files

- **Modify:** `apps/web/src/lib/auth-client.ts`
- **Modify:** `apps/web/src/app/(main)/design/backlog/page.tsx`
- **Modify:** `apps/web/src/app/dev-tools/page.tsx`
- **Modify:** `apps/web/src/features/design/components/workspace/create-workspace-modal.tsx`

## Implementation Steps

### 1. Update auth-client.ts

Change `API_BASE` and response handling:

```typescript
const API_BASE = '/api/v1';

// POST /api/v1/auth/login
export async function login(credentials: LoginCredentials): Promise<LoginResponse> {
  const response = await fetch(`${API_BASE}/auth/login`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(credentials),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Login failed');
  }

  const result = await response.json();
  const data = result.data;  // Unwrap from { data: ... }
  setToken(data.token);
  return data;
}

// POST /api/v1/auth/logout
export async function logout(): Promise<void> {
  await fetch(`${API_BASE}/auth/logout`, {
    method: 'POST',
    headers: getAuthHeaders(),
  });
  removeToken();
}

// GET /api/v1/auth/me
export async function getMe(): Promise<MockUser> {
  const token = getToken();
  if (!token) {
    throw new Error('No token');
  }

  const response = await fetch(`${API_BASE}/auth/me`, {
    headers: getAuthHeaders(),
  });

  if (!response.ok) {
    if (response.status === 401) {
      removeToken();
      throw new Error('Unauthorized');
    }
    const error = await response.json();
    throw new Error(error.error?.message || 'Failed to get user');
  }

  const result = await response.json();
  return result.data;  // Unwrap
}

// POST /api/v1/auth/refresh
export async function refreshToken(): Promise<string> {
  const response = await fetch(`${API_BASE}/auth/refresh`, {
    method: 'POST',
    headers: getAuthHeaders(),
  });

  if (!response.ok) {
    removeToken();
    const error = await response.json();
    throw new Error(error.error?.message || 'Token refresh failed');
  }

  const result = await response.json();
  const newToken = result.data.token;
  setToken(newToken);
  return newToken;
}
```

### 2. Update backlog/page.tsx

Update bulk action URLs and request body:

```typescript
// handleAddToWorkspace
const response = await fetch('/api/v1/design/tasks/bulk/assign-workspace', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    task_ids: selectedTaskIds,  // snake_case
    workspace_id: workspaceId,
  }),
});

// handleAssignDesigner
const response = await fetch('/api/v1/design/tasks/bulk/assign-designer', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    task_ids: selectedTaskIds,
    assignee_id: designerId,
  }),
});

// handleToggleUrgent
const response = await fetch('/api/v1/design/tasks/bulk/toggle-urgent', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    task_ids: selectedTaskIds,
    is_urgent: isUrgent,
  }),
});
```

### 3. Update dev-tools/page.tsx

Update tasks fetch:

```typescript
const res = await fetch("/api/v1/design/tasks");
if (!res.ok) throw new Error("Failed to fetch tasks");
const result = await res.json();
return result.data;  // Unwrap from { data: [...] }
```

### 4. Update create-workspace-modal.tsx

Update workspace create URL:

```typescript
const response = await fetch('/api/v1/design/workspaces', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    name: data.name,
    ownerId: user?.id,  // or owner_id if BE expects snake_case
    description: data.description,
    color: data.color,
  }),
});

if (!response.ok) {
  const error = await response.json();
  throw new Error(error.error?.message || 'Failed to create workspace');
}

const result = await response.json();
const newWorkspace = result.data;  // Unwrap
```

### 5. Create API client utility (Optional - for DRY)

Consider creating shared fetch wrapper:

```typescript
// apps/web/src/lib/api-client.ts
export async function apiClient<T>(url: string, options?: RequestInit): Promise<T> {
  const response = await fetch(url, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
  });

  const result = await response.json();

  if (!response.ok) {
    throw new Error(result.error?.message || 'Request failed');
  }

  return result.data;
}
```

## Todo List

- [ ] Update auth-client.ts API_BASE to `/api/v1`
- [ ] Update auth-client.ts to unwrap `{ data: ... }` responses
- [ ] Update auth-client.ts error handling for `{ error: { message } }`
- [ ] Update backlog/page.tsx bulk action URLs
- [ ] Update backlog/page.tsx request body to snake_case
- [ ] Update dev-tools/page.tsx tasks URL
- [ ] Update dev-tools/page.tsx to unwrap response
- [ ] Update create-workspace-modal.tsx URL
- [ ] Update create-workspace-modal.tsx to unwrap response
- [ ] Test all API calls work correctly

## Success Criteria

- [ ] All frontend API calls use `/api/v1/{module}/*` URLs
- [ ] All responses properly unwrapped from `{ data: ... }`
- [ ] Error messages extracted from `{ error: { message } }`
- [ ] No TypeScript compilation errors
- [ ] Manual testing passes

## Risk Assessment

- **Breaking Changes:** Frontend will break if handlers not updated first
- **Mitigation:** Complete phases 1-5 before phase 6, test each phase

## Security Considerations

- Auth token handling unchanged, just URL paths
- Error messages don't expose sensitive info
