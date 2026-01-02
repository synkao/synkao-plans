# MSW Best Practices Research Report

**Date:** 2025-12-27 | **Focus:** TypeScript Patterns for Mock Service Worker

---

## 1. State Machine Validation in Handlers

**Pattern:** Generic Type Constraints with Union Types

MSW validates state through **generic arguments** on HTTP handlers:

```typescript
// Type-safe state validation via generics
http.get<Params, RequestBodyType, ResponseBodyType>(
  path,
  ({ request, params }) => {
    // TS ensures params match declared type
    // Request body type-checked at compile time
  }
)
```

**Implementation:**
- Use TypeScript's generic parameters: `<Params, RequestBody, ResponseBody>`
- Leverage union types for valid state transitions
- `HttpResponseResolver<never, State, Response>` constrains resolver signatures
- Type mismatch catches invalid state mutations at compile time

**CRUD Pattern with Validation:**
```typescript
type ValidOrder = { id: string; status: 'pending' | 'shipped'; version: number }
type UpdatePayload = Partial<ValidOrder>

http.patch<{ id: string }, UpdatePayload, ValidOrder>(
  '/orders/:id',
  validateAndRespond  // Type-locked to UpdatePayload → ValidOrder
)
```

---

## 2. Optimistic Locking Patterns

**REST API Implementation (ETag-based):**

1. **Version Field Tracking:**
   - Use `version` or `updatedAt` timestamp in state
   - Client includes `If-Match: <version>` header
   - Server validates: `clientVersion === serverVersion`

2. **Conflict Resolution:**
   ```typescript
   http.patch<{ id: string }, UpdatePayload, Response>(
     '/resource/:id',
     async ({ request, params }) => {
       const clientVersion = request.headers.get('if-match')
       const resource = await db.get(params.id)

       if (clientVersion !== resource.version) {
         return HttpResponse.json(
           { error: 'Conflict' },
           { status: 412 } // Precondition Failed
         )
       }

       return HttpResponse.json({
         ...resource,
         version: resource.version + 1
       })
     }
   )
   ```

3. **MSW-Specific Mock State:**
   - Store version metadata in mock data store
   - Simulate conflict scenarios in test cases
   - Return 412 Precondition Failed for version mismatches

---

## 3. Multi-Tenant Data Filtering

**Safety Pattern:** Tenant Context + Automatic Filtering

```typescript
interface TenantContext {
  tenantId: string
  userId: string
}

http.get<void, never, Order[]>(
  '/orders',
  ({ request, context }: { context?: TenantContext }) => {
    const tenantId = request.headers.get('x-tenant-id')

    // ENFORCE: No query returns cross-tenant data
    const filtered = allOrders.filter(
      o => o.tenantId === tenantId
    )

    return HttpResponse.json(filtered)
  }
)
```

**Best Practices:**
- Require explicit tenant ID in headers (`x-tenant-id`)
- Filter at handler entry point (not in resolver)
- Use middleware pattern for shared filtering logic
- Mock data structure includes tenant context by default
- Tests verify isolation: same resource, different tenants → different results

---

## 4. CRUD Handler Patterns with Validation

**Type-Safe CRUD Template:**

```typescript
type Entity = { id: string; tenantId: string; version: number }

// CREATE
http.post<void, CreatePayload, Entity>(
  '/entities',
  ({ request }) => {
    const body = await request.json() as CreatePayload
    validate(body) // Type-checked payload
    return HttpResponse.json(createEntity(body))
  }
)

// READ
http.get<{ id: string }, never, Entity>(
  '/entities/:id',
  ({ params }) => {
    const entity = db.get(params.id)
    return entity ? HttpResponse.json(entity) : HttpResponse.json(null, { status: 404 })
  }
)

// UPDATE with Optimistic Lock
http.patch<{ id: string }, PatchPayload, Entity>(
  '/entities/:id',
  ({ request, params }) => {
    const { version, ...updates } = await request.json()
    const current = db.get(params.id)

    if (current.version !== version) {
      return HttpResponse.json({ error: 'Conflict' }, { status: 412 })
    }

    return HttpResponse.json({ ...current, ...updates, version: version + 1 })
  }
)

// DELETE
http.delete<{ id: string }, never, void>(
  '/entities/:id',
  ({ params }) => {
    db.delete(params.id)
    return HttpResponse.json(null, { status: 204 })
  }
)
```

**Higher-Order Resolver Pattern:**
```typescript
function createValidatedHandler<T extends { tenantId: string }>(
  resolver: HttpResponseResolver<never, CreatePayload, T>
): HttpRequestHandler {
  return http.post('/entities', async (info) => {
    const body = await info.request.json()

    // Shared validation layer
    if (!body.tenantId) throw new Error('Missing tenantId')

    return resolver({ ...info, request: info.request } as any)
  })
}
```

---

## 5. Key Takeaways

| Pattern | Mechanism | Use Case |
|---------|-----------|----------|
| **State Validation** | TypeScript generics + union types | Prevent invalid data shapes |
| **Optimistic Locking** | Version/ETag + 412 response | Concurrent update conflicts |
| **Multi-Tenant** | Mandatory tenant header + filter at entry | Data isolation safety |
| **CRUD Type Safety** | Generic parameters + HttpResponseResolver | End-to-end type coverage |

---

## References

- [MSW TypeScript Best Practices](https://mswjs.io/docs/best-practices/typescript/)
- [Type-Safe API Mocking](https://dev.to/kettanaito/type-safe-api-mocking-with-mock-service-worker-and-typescript-21bf)
- [Optimistic Locking in REST APIs](https://sookocheff.com/post/api/optimistic-locking-in-a-rest-api/)
- [Multi-Tenant Data Isolation](https://securingbits.com/multi-tenant-data-isolation-patterns)

---

**Unresolved Questions:**
- XState integration for state machine validation (not native MSW feature)
- GraphQL-specific optimistic locking patterns (needs separate exploration)
- Performance implications of tenant filtering on large datasets
