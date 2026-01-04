---
title: "Milestone 2: MSW Mock Data + Auth Integration"
description: "Update MSW mock data per spec v1.2, implement login page + auth flow"
status: pending
priority: P1
effort: 6h
branch: main
tags: [msw, auth, milestone-2, mock-data]
created: 2026-01-04
---

# Milestone 2: MSW Mock Data + Auth Integration

## Overview

Two issues remaining for Milestone 2:
- **#64**: MSW Mock Data Update - Align types/data with spec v1.2
- **#63**: Login + Auth Integration - Complete auth flow

## Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Multi-tenant | Hardcode 1 tenant | MVP simplicity |
| Token storage | localStorage | Client-only, no SSR auth needed |
| Optimistic locking | Skip in mock | Not needed for frontend mocking |
| Form validation | react-hook-form + zod | Already in stack |

## Phases

| Phase | Scope | Est. | Issues |
|-------|-------|------|--------|
| 1 | Types + Factory | 45min | #64 |
| 2 | Data Files | 1h | #64 |
| 3 | MSW Handlers | 1.5h | #64 |
| 4 | Verification | 45min | #64 |
| 5 | Auth Hooks | 45min | #63 |
| 6 | Login Page | 1h | #63 |
| 7 | Protected Routes | 45min | #63 |

## Dependencies

```
Phase 1 → Phase 2 → Phase 3 → Phase 4
                                 ↓
Phase 5 → Phase 6 → Phase 7
```

## File Structure Changes

```
apps/web/src/
├── mocks/
│   ├── types.ts                 # UPDATE: Full spec interfaces
│   ├── factories/
│   │   └── mock-factory.ts      # UPDATE: Add tenant/column IDs
│   ├── data/
│   │   ├── tenant.data.ts       # NEW: 1 hardcoded tenant
│   │   ├── columns.data.ts      # NEW: Kanban columns
│   │   ├── users.data.ts        # UPDATE: Add tenantId
│   │   ├── orders.data.ts       # UPDATE: Full spec fields
│   │   └── tasks.data.ts        # UPDATE: Full spec fields
│   └── handlers/
│       ├── context.ts           # NEW: x-tenant-id extraction
│       ├── auth.ts              # UPDATE: Token-aware /me
│       └── tasks.ts             # UPDATE: Use new types
├── hooks/
│   └── use-auth.ts              # NEW: Auth mutations
├── lib/
│   └── auth-client.ts           # NEW: Token storage
├── components/
│   └── auth/
│       └── auth-guard.tsx       # NEW: Route protection
└── app/
    ├── (auth)/login/page.tsx    # UPDATE: Add form logic
    └── (main)/layout.tsx        # UPDATE: Add auth guard
```

## Success Criteria

- [ ] Types align with spec v1.2 enums
- [ ] Hardcoded tenant propagates through all mock data
- [ ] Login form validates and calls MSW endpoint
- [ ] Token persists in localStorage
- [ ] Protected routes redirect to /login when no token
- [ ] `pnpm typecheck && pnpm build` pass

## Phase Files

- [Phase 1: Types + Factory](./phase-01-msw-types-factory.md)
- [Phase 2: Data Files](./phase-02-msw-data-files.md)
- [Phase 3: MSW Handlers](./phase-03-msw-handlers.md)
- [Phase 4: Verification](./phase-04-verification.md)
- [Phase 5: Auth Hooks](./phase-05-auth-hooks.md)
- [Phase 6: Login Page](./phase-06-login-page.md)
- [Phase 7: Protected Routes](./phase-07-protected-routes.md)

## Validation Summary

**Validated:** 2026-01-04
**Questions:** 4

### Confirmed Decisions
| Decision | Choice | Notes |
|----------|--------|-------|
| Enum Migration | Update directly | Find & replace DONE→APPROVED |
| Route Protection | Client-side guard | Accept loading flash, MSW compatible |
| Error UX | Inline only | No toast for login form |
| Type Errors | Fix in Phase 4 | Block #63 until build passes |

### Action Items
- [x] Plan validated - no changes required
- [ ] Grep 'DONE' status before Phase 1 to identify affected files

## Related Docs

- [Spec v1.2](../../docs/backend/synkao-models-spec.md)
- [MSW Research](./research/researcher-msw-patterns.md)
- [Auth Research](./research/researcher-auth-patterns.md)
