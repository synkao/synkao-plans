# Code Review: Mock Data Placeholder Images

**Date**: 2026-01-03
**Reviewer**: code-reviewer
**Commit**: a28b13b (last 5 commits analyzed)

---

## Scope

**Files reviewed**: 5
- `apps/web/src/mocks/types.ts`
- `apps/web/src/mocks/data/users.data.ts`
- `apps/web/src/mocks/data/order-items.data.ts`
- `apps/web/src/mocks/data/tasks.data.ts`
- `apps/web/src/mocks/data/design-versions.data.ts`

**Lines changed**: ~30 additions/modifications
**Focus**: Mock data placeholder image URLs
**Build status**: ✅ Successful (Next.js compiled, TypeScript passed)

---

## Overall Assessment

Changes add placeholder images từ external services (DiceBear avatars, Picsum photos) cho mock data. Implementation đơn giản, functional, comply với YAGNI/KISS. URLs correct, no security issues with placeholder services.

**Quality**: Good
**Compliance**: YAGNI ✅ | KISS ✅ | DRY ✅

---

## Critical Issues

None.

---

## High Priority Findings

None.

---

## Medium Priority Improvements

### M1: URL Pattern Consistency - Design vs Version URLs

**File**: `tasks.data.ts`, `design-versions.data.ts`

**Issue**: Cả 2 đều dùng Picsum nhưng seed patterns khác nhau:
- Tasks: `seed/design-${n}`
- Versions: `seed/version-${versionIndex}`

**Current**:
```typescript
// tasks.data.ts:41
designUrl: hasDesign ? `https://picsum.photos/seed/design-${n}/800/600` : undefined,

// design-versions.data.ts:33
fileUrl: `https://picsum.photos/seed/version-${versionIndex}/800/600`,
```

**Recommendation**: Consider dùng task-based seed cho versions để related images có similarity:
```typescript
// Option 1: Link version to task
fileUrl: `https://picsum.photos/seed/task-${task.taskCode}-v${v}/800/600`,

// Option 2: Keep current (acceptable)
// If versions cần look different từ main design
```

**Impact**: Medium - UX clarity. Versions của same task nên visually related.

---

## Low Priority Suggestions

### L1: Avatar URL Encoding

**File**: `users.data.ts:6`

**Current**:
```typescript
const avatarUrl = (name: string) =>
  `https://api.dicebear.com/9.x/avataaars/svg?seed=${encodeURIComponent(name)}`;
```

**Observation**: Good practice dùng `encodeURIComponent()`. No issues.

---

### L2: Missing Alt Text for Avatar Images

**Lint warning** detected:
```
task-card.tsx:36:9  warning  Image elements must have an alt prop
```

**Action needed**: Add alt text cho avatar images in components using mock data.

**Example**:
```typescript
<img src={user.avatar} alt={`${user.name} avatar`} />
```

---

### L3: DRY - Picsum Base URL

**Files**: `order-items.data.ts`, `tasks.data.ts`, `design-versions.data.ts`

**Current**: Hardcoded `https://picsum.photos/seed/...` in 3 files.

**Suggestion**: Extract constant nếu URLs cần change later:
```typescript
// mocks/constants.ts
export const PICSUM_BASE = 'https://picsum.photos/seed';

// Usage
productImageUrl: `${PICSUM_BASE}/item-${itemIndex}/400/400`,
```

**Priority**: Low - chỉ 3 occurrences, acceptable duplication.

---

## Positive Observations

✅ **Type safety**: `productImageUrl?: string` correctly optional
✅ **URL patterns**: Semantic seeds (`item-`, `design-`, `version-`) easy debug
✅ **Dimensions**: Appropriate sizes (400x400 products, 800x600 designs)
✅ **Encoding**: `encodeURIComponent()` for user names
✅ **YAGNI compliance**: No over-engineering, simple string templates
✅ **KISS compliance**: Straightforward URL generation
✅ **Build passing**: TypeScript compilation successful
✅ **Service reliability**: DiceBear & Picsum are established placeholder services

---

## Recommended Actions

### Priority Order

1. **[Medium]** Review design-version URL seed pattern - decide if versions nên link to task visually
2. **[Low]** Add alt text cho avatar images (fix lint warning)
3. **[Optional]** Extract Picsum base URL constant nếu anticipate changes

### Code Fixes

#### Fix 1: Alt Text (Low Priority)
```typescript
// In component using avatar
<img
  src={user.avatar}
  alt={`${user.name} avatar`}
  className="..."
/>
```

#### Fix 2: Version URL Pattern (Medium - Optional)
```typescript
// design-versions.data.ts:33
// Option A: Link to task
fileUrl: `https://picsum.photos/seed/task-${task.taskCode}-v${v}/800/600`,

// Option B: Keep current (acceptable)
fileUrl: `https://picsum.photos/seed/version-${versionIndex}/800/600`,
```

---

## Security Review

✅ **No vulnerabilities** detected:
- Placeholder services (DiceBear, Picsum) are public APIs
- No API keys or secrets embedded
- URLs use HTTPS
- No user input injection (names encoded properly)
- No sensitive data exposure

---

## Metrics

- **Type Coverage**: 100% (all fields properly typed)
- **Linting Issues**: 4 total in codebase
  - 1 unused eslint-disable (unrelated)
  - 1 Math.random purity issue (unrelated)
  - 1 unused variable (unrelated)
  - 1 missing alt text (related - low priority)
- **Build**: ✅ Success
- **Test Coverage**: N/A (mock data)

---

## Unresolved Questions

1. Version images nên visually relate to parent task không? Current seed pattern makes them independent.
2. Có plan migrate từ placeholder images sang real CDN URLs không? Document migration path?
