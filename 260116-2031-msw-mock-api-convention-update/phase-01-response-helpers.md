---
phase: 1
title: "Create Response Helper Utilities"
status: completed
effort: 30m
reviewed: 2026-01-16
score: 9/10
---

# Phase 01: Create Response Helper Utilities

## Overview

Create reusable response helper functions following DRY principle. All handlers will use these helpers for consistent response formatting.

## Context

**Convention Reference:** `docs/frontend/mock-api-convention.md` Section 3

## Related Code Files

- **Create:** `apps/web/src/mocks/utils/response-helpers.ts`

## Implementation Steps

### 1. Create utils directory and response-helpers.ts

```typescript
// apps/web/src/mocks/utils/response-helpers.ts
import { HttpResponse } from 'msw';

// Response Types
export interface PaginationMeta {
  page: number;
  limit: number;
  total: number;
  total_pages: number;
}

export interface ApiError {
  code: string;
  message: string;
  details?: Array<{ field: string; message: string }>;
}

// Parse pagination params from URL
export function parsePagination(url: URL, defaultLimit = 20): { page: number; limit: number } {
  const page = parseInt(url.searchParams.get('page') || '1', 10);
  const limit = Math.min(parseInt(url.searchParams.get('limit') || String(defaultLimit), 10), 100);
  return { page, limit };
}

// Calculate pagination metadata
export function getPaginationMeta(total: number, page: number, limit: number): PaginationMeta {
  return {
    page,
    limit,
    total,
    total_pages: Math.ceil(total / limit),
  };
}

// Paginate array
export function paginateArray<T>(items: T[], page: number, limit: number): T[] {
  const start = (page - 1) * limit;
  return items.slice(start, start + limit);
}

// Success response - single resource
export function successResponse<T>(data: T, status = 200) {
  return HttpResponse.json({ data }, { status });
}

// Success response - collection with pagination
export function listResponse<T>(items: T[], pagination: PaginationMeta) {
  return HttpResponse.json({ data: items, pagination });
}

// Success response - created
export function createdResponse<T>(data: T) {
  return HttpResponse.json({ data }, { status: 201 });
}

// Success response - action (message only)
export function actionResponse(message: string) {
  return HttpResponse.json({ message });
}

// Success response - bulk action
export function bulkActionResponse(message: string, affectedCount: number, affectedIds: string[]) {
  return HttpResponse.json({
    message,
    data: {
      affected_count: affectedCount,
      affected_ids: affectedIds,
    },
  });
}

// Error response
export function errorResponse(code: string, message: string, status: number) {
  return HttpResponse.json(
    { error: { code, message } },
    { status }
  );
}

// Validation error response
export function validationErrorResponse(details: Array<{ field: string; message: string }>) {
  return HttpResponse.json(
    {
      error: {
        code: 'VALIDATION_FAILED',
        message: 'Request validation failed',
        details,
      },
    },
    { status: 400 }
  );
}

// Common error helpers
export const errors = {
  notFound: (resource: string) => errorResponse(`${resource.toUpperCase()}_NOT_FOUND`, `${resource} not found`, 404),
  unauthorized: () => errorResponse('UNAUTHORIZED', 'Authentication required', 401),
  invalidCredentials: () => errorResponse('INVALID_CREDENTIALS', 'Invalid email or password', 401),
  forbidden: () => errorResponse('INSUFFICIENT_PERMISSIONS', 'No permission to perform this action', 403),
  conflict: (code: string, message: string) => errorResponse(code, message, 409),
  businessRule: (code: string, message: string) => errorResponse(code, message, 422),
};
```

### 2. Export from utils index

```typescript
// apps/web/src/mocks/utils/index.ts
export * from './response-helpers';
```

## Todo List

- [x] Create `apps/web/src/mocks/utils/` directory
- [x] Create `response-helpers.ts` with all helper functions
- [x] Create `index.ts` to export helpers
- [x] Verify TypeScript compilation

## Success Criteria

- [x] All response helper functions created
- [x] Functions handle pagination parsing/calculation
- [~] Error codes match convention doc Section 6 (95% - missing 5 error codes)
- [x] No TypeScript errors

## Code Review Findings

**Date:** 2026-01-16 | **Score:** 9/10

**Medium Priority:**
- Missing 5 error codes from Section 6.2: `tokenExpired`, `tokenInvalid`, `invalidStatus`, `invalidDateRange`, `missingField`

**Low Priority:**
- No network delay simulation (can defer to Phase 02)
- Consider adding JSDoc examples

**Report:** `reports/code-reviewer-260116-2057-phase-01-response-helpers.md`
