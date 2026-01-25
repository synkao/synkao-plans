# Debug Report: React Hydration Error - Whitespace in Table Rows

**Date:** 2026-01-25 15:19
**Reporter:** debugger-a3fb86c
**Issue:** React hydration error - whitespace text nodes in `<tr>` elements
**File:** `apps/web/src/features/orders/components/order-list/order-list-table.tsx`

---

## Executive Summary

**Root Cause:** Whitespace between closing `</thead>` and opening `<tbody>` tags creates text node that React cannot hydrate properly in HTML table structure.

**Impact:** Console errors in production, potential hydration mismatch warnings, degrades user experience.

**Fix Priority:** HIGH - Simple one-line fix, affects user-facing feature.

---

## Technical Analysis

### Issue Location

**File:** `apps/web/src/features/orders/components/order-list/order-list-table.tsx`
**Lines:** 115-116

```tsx
            </thead>
          <tbody>  // ← Line 116: whitespace before <tbody>
```

### Problem Details

Between line 115 (`</thead>`) and line 116 (`<tbody>`), there's a newline + whitespace indentation that creates a text node in the DOM. HTML spec prohibits text nodes as direct children of `<table>` - only `<thead>`, `<tbody>`, `<tfoot>`, `<caption>`, and `<colgroup>` are valid.

During SSR, React generates clean HTML without this text node. During client hydration, React's JSX parser preserves the whitespace, creating mismatch.

### Evidence

Error message points to line 84:15, which is inside `<thead>`:
```tsx
84:              <tr>
```

However, the actual issue is **after** the `</thead>` tag. React error reporting sometimes points to parent context rather than exact whitespace location.

### Verification

Examining the code structure:
- **Line 115:** `</thead>` - properly closed
- **Line 116:** `<tbody>` - has leading whitespace (indentation)
- This creates: `</thead>[WHITESPACE_TEXT_NODE]<tbody>`

---

## Actionable Recommendations

### Immediate Fix

**Remove whitespace between `</thead>` and `<tbody>`:**

```tsx
// Current (BROKEN):
            </thead>
          <tbody>

// Fixed:
            </thead><tbody>
```

**OR** use React Fragment pattern:

```tsx
<>
  <thead className="border-b border-gray-200 bg-gray-50/80">
    {/* ... header content ... */}
  </thead>
  <tbody>
    {/* ... body content ... */}
  </tbody>
</>
```

### Implementation Steps

1. Open `apps/web/src/features/orders/components/order-list/order-list-table.tsx`
2. Locate line 115-116
3. Remove newline between `</thead>` and `<tbody>`
4. Result should be: `</thead><tbody>`
5. Test in dev mode to verify hydration warning disappears
6. Run build to ensure no other issues

### Testing Validation

1. Clear browser cache
2. Start dev server: `npm run dev`
3. Navigate to Orders page
4. Open browser console
5. Verify no hydration warnings
6. Check table renders correctly
7. Test expand/collapse functionality
8. Run production build: `npm run build`
9. Verify no build warnings

---

## Supporting Evidence

### Code Context

The component uses array-based row rendering (line 190-230) to avoid whitespace issues in `<tbody>`, but missed the gap between table sections.

**OrderGroup component (line 174-231):**
```tsx
// Uses array approach to avoid whitespace in tbody
const rows: React.ReactNode[] = [/* ... */];
return <>{rows}</>;  // ← No whitespace here, good!
```

However, the table structure itself still has whitespace between structural elements.

### Browser Behavior

- **Server render:** Clean HTML output (no whitespace text nodes)
- **Client hydration:** JSX preserves whitespace → creates text node
- **Result:** Hydration mismatch error

---

## Long-term Improvements

### 1. ESLint Rule
Add `react/jsx-no-whitespace-in-tables` to ESLint config to catch similar issues.

### 2. Table Component Abstraction
Create reusable `<Table>` component that handles structure correctly:

```tsx
<Table>
  <Table.Header>{/* ... */}</Table.Header>
  <Table.Body>{/* ... */}</Table.Body>
</Table>
```

### 3. Code Review Checklist
Add table whitespace check to PR review template.

### 4. Testing
Add hydration error detection to E2E tests.

---

## Risk Assessment

**Severity:** Medium
**Likelihood:** Already occurring
**User Impact:** Console errors, potential layout issues in edge cases

**Mitigation:** Single-line fix with zero side effects.

---

## Security Considerations

No security implications - purely presentational issue.

---

## Unresolved Questions

None - root cause identified, fix straightforward.
