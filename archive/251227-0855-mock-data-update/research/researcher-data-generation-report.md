# Mock Data Generation Patterns - TypeScript Research Report

**Date:** 2025-12-27 | **Project:** Synkao POD
**Focus:** TypeScript factory patterns, referential integrity, US address generation, deterministic UUIDs

---

## 1. Factory Pattern Implementations

### Library Options (Ranked by TypeScript Support)

**factory.ts** (Production-Ready)
- Type-safe factories via Partial<T> pattern
- Sequence-based value generation (auto-incremented IDs)
- Async factory support with Promise returns
- Transform() method for derived values
- Immutable data (prevents cross-test contamination)
- Minimal setup overhead

**factory.io** (Class-Based)
- ORM-friendly (TypeORM integration)
- Modern class-based approach
- Better for entity hierarchies
- ~500KB footprint

**Typical Data** (In-Memory DB)
- Combines factories + lightweight query interface
- Mock Service Worker compatible
- React Testing Library ready
- Full relationship querying

### Factory Pattern Core Pattern

```typescript
// Type-safe factory using Partial<T>
type User = { id: string; email: string; role: 'admin' | 'user' };

const userFactory = factory<User>({
  id: (seq) => `user-${seq}`,
  email: (seq) => `user${seq}@example.com`,
  role: 'user'
});

// Usage
const testUser = userFactory.build();  // id: 'user-1', email: 'user1@example.com'
const users = userFactory.buildList(5);
```

---

## 2. Complex Entity Value Objects (Address, Money)

### Address Factory Pattern

```typescript
type Address = {
  street: string;
  city: string;
  state: string;  // 2-letter US code
  zipCode: string;  // 5-digit
  country: 'US' | 'CA' | 'MX';
};

const addressFactory = factory<Address>({
  street: (seq) => `${seq} Main Street`,
  city: () => faker.location.city(),
  state: () => faker.location.state({ abbreviated: true }),
  zipCode: () => faker.location.zipCode('#####'),
  country: 'US'
});
```

### Money Value Object Factory

```typescript
type Money = {
  amount: number;      // cents to avoid float issues
  currency: 'USD' | 'CAD';
  formatted: string;   // derived
};

const moneyFactory = factory<Money>({
  amount: (seq) => seq * 1000,  // $10, $20, $30...
  currency: 'USD',
  formatted: (seq, data) =>
    `$${(data.amount / 100).toFixed(2)}`
});
```

---

## 3. Referential Integrity Maintenance

### Parent-Child Factory Pattern

```typescript
type Order = { id: string; customerId: string };
type Customer = { id: string; name: string };

const customerFactory = factory<Customer>({
  id: (seq) => `cust-${seq}`,
  name: () => faker.person.fullName()
});

const orderFactory = factory<Order>({
  id: (seq) => `order-${seq}`,
  customerId: undefined  // Will be set per test
});

// In tests, maintain reference
const customer = customerFactory.build();
const order = orderFactory.build({
  customerId: customer.id  // Explicit FK relationship
});
```

### Bulk Generation with Integrity

```typescript
// Generate consistent datasets
function generateOrderDataset(count: number) {
  const customers = customerFactory.buildList(3);
  const orders = orderFactory.buildList(count, {
    customerId: () => faker.helpers.arrayElement(
      customers.map(c => c.id)
    )
  });
  return { customers, orders };
}
```

---

## 4. Deterministic UUID Generation

### Seeded Faker for Reproducibility

```typescript
import { faker } from '@faker-js/faker';

// Set seed for reproducible results
faker.seed(12345);

const id1 = faker.string.uuid();  // Always same
const id2 = faker.string.uuid();  // Always same per run

// Reset seed for new sequence
faker.seed(12345);
const id3 = faker.string.uuid();  // Equals id1
```

### Lightweight UUID Generation

```typescript
import { simpleFaker } from '@faker-js/faker';

// Minimal footprint (~5KB vs 500KB)
const userId = simpleFaker.string.uuid();
```

### Deterministic Sequential IDs (Preferred for Testing)

```typescript
// Instead of random UUIDs, use sequential for test clarity
const entityFactory = factory<Entity>({
  id: (seq) => `entity-${String(seq).padStart(5, '0')}`,
  // OR with UUID prefix
  id: (seq) => `${faker.string.uuid()}-${seq}`
});
```

---

## 5. Realistic US Address Generation

### faker.js US Address Utilities

```typescript
import { faker } from '@faker-js/faker/locale/en_US';

const address = {
  street: faker.location.streetAddress(),      // "123 Main St"
  city: faker.location.city(),                 // "Portland"
  state: faker.location.state({ abbreviated: true }), // "OR"
  zipCode: faker.location.zipCode('#####'),    // "97201"
  country: 'US'
};
```

### Valid US ZIP Code Ranges

```typescript
// ZIP codes by region (simplified)
const ZIP_RANGES = {
  'CA': ['90000-96999'],
  'TX': ['75000-79999'],
  'NY': ['10000-14999'],
  'OR': ['97000-97999']
};

// Constrained generation
const stateZips = {
  'CA': () => faker.string.numeric('9####'),  // 9xxxx range
  'NY': () => faker.string.numeric('#13##'),  // 1xxxx range
};
```

---

## 6. TypeScript Best Practices

### Type Safety Layers

```typescript
// 1. Domain types (immutable)
type Order = Readonly<{ id: string; total: Money }>;

// 2. Factory overrides (partial)
type OrderFactoryData = Partial<Order>;

// 3. Sequence function pattern
type FactoryFn<T> = (sequence: number, data: T) => T[keyof T];
```

### Composition Over Inheritance

```typescript
// Reuse factories via composition
const userFactory = factory<User>({ /* ... */ });
const adminFactory = factory<Admin>({
  ...userFactory.attrs(),
  role: 'admin'
});
```

---

## 7. Recommended Tech Stack for Synkao

| Task | Library | Reason |
|------|---------|--------|
| Core factories | factory.ts | Type-safe, lightweight, battle-tested |
| Fake data | @faker-js/faker | Locale support, comprehensive API, active maintenance |
| Lightweight UUID | simpleFaker | 5KB, no locale data overhead |
| Schema validation | Zod | Works with factories, strict types |
| E2E mocks | Typical Data | Full relational querying for complex scenarios |

---

## Unresolved Questions

1. Should Synkao use sequential IDs (e.g., `user-001`) or UUIDs for test clarity?
2. Need validation: Are US address generators sufficient or require actual USPS ZIP database?
3. Will referential integrity be enforced at factory level or test setup?
4. Size concern: Is 500KB Faker footprint acceptable for Next.js bundle impact?

