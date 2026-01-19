# Brainstorm Report: Backlog React Query Integration

**Date:** 2026-01-16
**Status:** Ready for Implementation

---

## Problem Statement

Backlog page ([page.tsx](apps/web/src/app/(main)/design/backlog/page.tsx)) hiện dùng local state + raw fetch:
- Import trực tiếp `mockTasks` từ mock data
- Raw `fetch()` cho bulk actions
- Không có cache, không có loading states chuẩn
- Không tận dụng React Query đã setup sẵn

---

## Existing Patterns (Reference)

| Component | Path |
|-----------|------|
| Auth Client | [auth-client.ts](apps/web/src/lib/auth-client.ts) |
| Auth Hooks | [use-auth.ts](apps/web/src/hooks/use-auth.ts) |
| Mock Handlers | [tasks.ts](apps/web/src/mocks/handlers/tasks.ts) |
| Response Helpers | [response-helpers.ts](apps/web/src/mocks/utils/response-helpers.ts) |

---

## API Endpoints (từ Mock)

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/v1/design/tasks` | GET | List tasks với filters |
| `/api/v1/design/tasks/:id` | GET | Single task |
| `/api/v1/design/tasks/:id` | PATCH | Update task |
| `/api/v1/design/tasks/bulk/assign-workspace` | POST | Bulk assign workspace |
| `/api/v1/design/tasks/bulk/assign-designer` | POST | Bulk assign designer |
| `/api/v1/design/tasks/bulk/toggle-urgent` | POST | Bulk toggle urgent |

### Filter Params

```typescript
interface TaskListParams {
  page?: number;
  limit?: number;
  status?: DesignTaskStatusType;
  workspace_id?: string | null;  // null = unassigned
  assignee_id?: string;
  is_urgent?: boolean;
}
```

---

## Decision: Server-side Filter

User đã chọn **server-side filter** cho backlog (status=TODO, workspace_id=null).

**Action Required:** Update mock handler để hỗ trợ `workspace_id=null` filter.

---

## Implementation Plan

### 1. Update Mock Handler
- Add support for `workspace_id=null` to filter unassigned tasks

### 2. Create Task Client
**File:** `features/design/lib/tasks-client.ts`

```typescript
export async function getTasks(params: TaskListParams)
export async function getTask(id: string)
export async function bulkAssignWorkspace(taskIds: string[], workspaceId: string)
export async function bulkAssignDesigner(taskIds: string[], assigneeId: string)
export async function bulkToggleUrgent(taskIds: string[], isUrgent: boolean)
```

### 3. Create Task Hooks
**File:** `features/design/hooks/use-tasks.ts`

```typescript
// Query key factory
export const taskKeys = {
  all: ['tasks'] as const,
  lists: () => [...taskKeys.all, 'list'] as const,
  list: (filters: TaskListParams) => [...taskKeys.lists(), filters] as const,
  details: () => [...taskKeys.all, 'detail'] as const,
  detail: (id: string) => [...taskKeys.details(), id] as const,
}

// Hooks
export function useTasks(params: TaskListParams)      // useQuery
export function useTask(id: string)                   // useQuery
export function useBulkAssignWorkspace()              // useMutation
export function useBulkAssignDesigner()               // useMutation
export function useBulkToggleUrgent()                 // useMutation
```

### 4. Update BacklogPage
- Replace `mockTasks` với `useTasks()` hook
- Replace raw fetch với mutation hooks
- Add loading/error states
- Keep client-side search/urgentOnly/source filters (UX snappy)

---

## File Structure

```
features/design/
├── lib/
│   └── tasks-client.ts       # NEW: Raw fetch functions
└── hooks/
    ├── use-tasks.ts          # NEW: React Query hooks
    └── index.ts              # UPDATE: Export new hooks
```

---

## Todo Checklist

- [ ] Update mock handler to support workspace_id=null filter
- [ ] Create tasks-client.ts with fetch functions
- [ ] Create use-tasks.ts with React Query hooks
- [ ] Update BacklogPage to use React Query hooks
- [ ] Run compile check and verify

---

## Notes

- Follow auth pattern: client layer → hook layer → component
- Use query key factory for cache invalidation
- Mutations invalidate `taskKeys.lists()` on success
- Keep client-side filters for search/urgent/source (immediate UX feedback)
