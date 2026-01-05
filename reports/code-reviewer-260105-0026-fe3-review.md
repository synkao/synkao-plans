# Code Review: FE-3 Implementation (Issues #17 & #18)

## Scope
- **Files reviewed**: 11 files (Phase 1: Backlog, Phase 2: Workspace)
- **LOC analyzed**: ~700 lines
- **Focus**: Recent changes - backlog bulk actions + workspace selector
- **Updated plans**: None

## Overall Assessment

Implementation chất lượng tốt, tuân thủ React best practices, có TypeScript typing đầy đủ. Tuy nhiên có **2 critical linting errors** và vài issues về error handling.

---

## Critical Issues

### 1. **React Compiler Errors (sidebar.tsx - existing)**
```
Error: Cannot call impure function Math.random() during render
```
**Impact**: Lỗi React Compiler, không phải code mới nhưng cần fix.
**Location**: `src/components/ui/sidebar.tsx:611`
**Fix**: Move Math.random() vào useEffect hoặc dùng useState với initial value.

### 2. **React Hooks Error (use-kanban-state.ts - existing)**
```
Error: Calling setState synchronously within effect causes cascading renders
```
**Impact**: Performance issue, không phải FE-3 code.
**Location**: `src/features/design/hooks/use-kanban-state.ts:23`
**Fix**: Dùng useMemo thay vì useEffect cho derived state.

---

## High Priority Findings

### 1. **Unused error variables - backlog/page.tsx**
Lines 87, 108, 128:
```typescript
catch (error) {
  toast.error('Failed to ...');
}
```
**Issue**: Biến `error` unused, mất info debugging.
**Fix**:
```typescript
catch (error) {
  console.error('Bulk action failed:', error);
  toast.error(...);
}
```

### 2. **Missing input sanitization**
**Files**: `create-workspace-modal.tsx`, `backlog-action-bar.tsx`
**Issue**: User input (workspace name, description) không sanitize trước khi hiển thị.
**Risk**: XSS nếu backend không sanitize.
**Fix**: Thêm DOMPurify hoặc verify backend đã sanitize.

### 3. **Error handling thiếu response body parsing**
**Location**: `backlog/page.tsx:82, 103, 124`
```typescript
if (!response.ok) throw new Error('Failed to...');
```
**Issue**: Không parse error message từ API response.
**Fix**:
```typescript
if (!response.ok) {
  const errorData = await response.json();
  throw new Error(errorData.error || 'Failed to...');
}
```

---

## Medium Priority Improvements

### 1. **State update warning - backlog-action-bar.tsx:56-60**
```typescript
if (selectedCount === 0 && isVisible) {
  setIsVisible(false);  // State update during render
}
```
**Issue**: Trigger re-render trong render phase.
**Fix**: Dùng useEffect hoặc remove state (dùng selectedCount trực tiếp).

### 2. **Accessibility - missing aria-labels**
- `BacklogActionBar` dropdowns thiếu `aria-label`
- Checkboxes trong table thiếu `aria-labelledby`

### 3. **Performance - useMemo dependencies**
**Location**: `backlog-table.tsx:60-96`
```typescript
useMemo(() => { ... }, [tasks, sortField, sortDirection]);
```
**OK nhưng**: Consider React.memo cho SortableHeader component.

### 4. **Type safety - MockWorkspace fallback**
**Location**: `backlog/page.tsx:85, 106`
```typescript
workspace?.name || 'workspace'
```
**Issue**: Type đã nullable, nhưng logic fallback tốt.

---

## Low Priority Suggestions

### 1. **Unused imports**
- `workspaces.ts:4` - `WORKSPACE_IDS` unused
- `constants.ts:4` - `TaskSource` unused
- `utils.ts:3` - `DesignTaskStatus` unused

### 2. **Image optimization warnings**
- `<img>` tags nên dùng `next/image` (6 warnings)
- **Location**: backlog-action-bar.tsx:123, drawer/*.tsx, kanban/*.tsx

### 3. **Component naming pattern**
All components có naming consistent, structure tốt, exports clean.

---

## Positive Observations

✅ **TypeScript types**: Đầy đủ, strict, có export interfaces cho props
✅ **Component structure**: Clean separation (backlog/, workspace/)
✅ **Error boundaries**: Toast messages clear, UX friendly
✅ **Loading states**: isSubmitting handled correctly
✅ **Form validation**: Zod schema comprehensive
✅ **MSW handlers**: Response structure đúng, validation tốt
✅ **React patterns**: Hooks usage đúng, no prop drilling
✅ **Code organization**: index.ts exports clear, file naming kebab-case

---

## Recommended Actions

### Immediate (Before merge)
1. ~~Fix 2 critical React Compiler errors~~ (Existing issues, not FE-3)
2. Add `console.error()` cho 3 catch blocks unused error vars
3. Verify backend sanitizes workspace name/description input

### Short-term (Next PR)
4. Fix state update warning trong BacklogActionBar
5. Add aria-labels cho dropdowns và checkboxes
6. Parse error response bodies cho better error messages
7. Remove unused imports

### Nice-to-have
8. Replace `<img>` với `next/image` (6 files)
9. Add React.memo cho SortableHeader
10. Consider DOMPurify cho client-side sanitization

---

## Metrics
- **Type Coverage**: 100% (all props typed)
- **Test Coverage**: N/A (not measured)
- **Linting**: 2 errors (existing), 17 warnings (mostly img tags)
- **Build Status**: ✅ No blocking issues cho FE-3 code

---

## Security Checklist
✅ No SQL injection risk (MSW handlers)
✅ No eval/innerHTML/dangerouslySetInnerHTML
⚠️ XSS risk - depends on backend sanitization
✅ No sensitive data in code
✅ CORS handled by MSW

---

## Unresolved Questions

1. Backend có sanitize user input (workspace name/description) không?
2. FE-3 plan file ở đâu? (Không tìm thấy trong plans/)
3. Có cần add E2E tests cho bulk actions không?
4. Error tracking (Sentry) có capture bulk action failures không?
