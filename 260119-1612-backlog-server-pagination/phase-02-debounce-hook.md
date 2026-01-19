# Phase 2: Debounce Hook

## Overview
Create reusable debounce hook for search input.

## Tasks

- [ ] Create use-debounced-value.ts hook
- [ ] Export from hooks index

---

## Implementation

**File:** `apps/web/src/hooks/use-debounced-value.ts` (NEW)

```ts
import { useState, useEffect } from 'react';

/**
 * Debounce a value with specified delay
 * @param value - Value to debounce
 * @param delay - Delay in milliseconds (default: 300)
 * @returns Debounced value
 */
export function useDebouncedValue<T>(value: T, delay = 300): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}
```

---

## Export

**File:** `apps/web/src/hooks/index.ts`

Check if exists, add export:
```ts
export { useDebouncedValue } from './use-debounced-value';
```

---

## Usage Example

```tsx
const [search, setSearch] = useState('');
const debouncedSearch = useDebouncedValue(search, 300);

// Use debouncedSearch for API calls
const { data } = useBacklogTasks({ search: debouncedSearch || undefined });
```

---

## Validation

- [ ] Hook compiles without errors
- [ ] Value updates after delay
- [ ] Rapid changes only trigger one update
