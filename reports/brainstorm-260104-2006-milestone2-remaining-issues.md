# Brainstorm Report: Milestone 2 Remaining Issues

**Date:** 2026-01-04
**Issues:** #64 (MSW Update), #63 (Login + Auth)
**Status:** Ready for implementation plan

---

## Context

Milestone 2 (FE-2: Core Components) còn 2 issues P1:
- **#64**: Update mock data structure theo synkao-models-spec v1.2
- **#63**: Login page + auth integration

## Decisions Made

| Topic | Decision | Rationale |
|-------|----------|-----------|
| **Multi-tenant** | Single tenant (hardcode) | MVP simplicity, đủ cho demo |
| **Types Scope** | Full spec fields | Future-proof, tránh rework |
| **Login Design** | Tự thiết kế | Dùng Glass Card + Gradient Background components |
| **Token Storage** | localStorage | Simple, persist, OK cho internal app |
| **Optimistic Locking** | Bỏ qua | Mock always success, implement sau |

---

## Gap Analysis

### Issue #64: MSW Mock Data Update

| Current | Spec v1.2 | Action |
|---------|-----------|--------|
| MockUser: 5 fields | User: 11 fields | Add TenantID, Phone, Status, Permissions, etc. |
| MockOrder: 7 fields | Order: 25 fields | Major update: Address, Money, Tracking fields |
| MockTask: 12 fields | DesignTask: 26 fields | Add Workspace, Column, Version, Review fields |
| No Tenant | Tenant model | Add MockTenant with hardcoded default |
| No Column | Column model | Add MockColumn for Kanban |
| CRUD handlers basic | Proper REST | Add PATCH, presigned URL flow |

### Issue #63: Login + Auth

| Existing | Needed | Notes |
|----------|--------|-------|
| `auth.store.ts` ✅ | Use existing | setUser, clearAuth ready |
| MSW auth handlers ✅ | Verify works | POST login, GET /me exists |
| No login page | Create | Glass morphism styling |
| No auth hooks | Create | useLogin, useLogout, useCurrentUser |
| No auth guard | Create | Protected route wrapper |

---

## Implementation Order

```
#64 (MSW Update) → #63 (Login)
```

**Rationale:** Login cần mock data auth endpoints hoạt động đúng. #64 đảm bảo mock infra ổn định trước.

---

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Types breaking changes | High | Update imports incrementally, run typecheck |
| MSW handlers regression | Medium | Test existing features after update |
| Login form validation edge cases | Low | Use zod + react-hook-form |

---

## Next Steps

1. Create detailed implementation plan via `/plan`
2. Implement #64 in 4 phases as specified in issue
3. Implement #63 after #64 complete
4. Verify build passes after each phase

---

## Unresolved Questions

None - all key decisions made.
