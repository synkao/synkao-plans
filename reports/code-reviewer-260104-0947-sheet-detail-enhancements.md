# Code Review: Sheet Detail Enhancements

**Reviewer:** code-reviewer
**Date:** 2026-01-04
**Commit Range:** Recent changes (last 3 days)
**Focus:** Security, Performance, Architecture, YAGNI/KISS/DRY

---

## Scope

**Files Reviewed:**
- Mock types & data: `types.ts`, `mock-factory.ts`, `*-files.data.ts`, `comments.data.ts`, `order-items.data.ts`
- UI components (13 files): `print-specs-section.tsx`, `edit-specs-modal.tsx`, `comments-section.tsx`, `reference-files-section.tsx`, `image-lightbox.tsx`, `file-upload-zone.tsx`, `quick-status-actions.tsx`, `version-upload-form.tsx`, `design-versions-section.tsx`, `task-info-section.tsx`, `task-detail-drawer.tsx`, v.v.

**Lines Analyzed:** ~1,350 lines total
**Build Status:** ✅ Build thành công
**TypeScript:** ✅ No type errors
**Main Branch:** main

---

## Overall Assessment

Code chất lượng **tốt** với kiến trúc rõ ràng, separation of concerns đúng mức. Implementation tuân thủ YAGNI/KISS principles. Không phát hiện lỗi critical security hay type safety. Có vài opportunities để cải thiện performance và code organization.

**Grade:** B+ (Good - Production Ready với minor improvements)

---

## Critical Issues

### ❌ None Found

Không phát hiện critical security vulnerabilities, data loss risks, hay breaking changes.

---

## High Priority Findings

### 1. **Memory Leak Risk - URL.createObjectURL**
**File:** `version-upload-form.tsx` (lines 26-52)
**Impact:** Memory leak khi modal đóng mà không revoke object URL

**Current Code:**
```tsx
const handleFilesSelected = (files: File[]) => {
  const file = files[0];
  if (file) {
    setSelectedFile(file);
    const url = URL.createObjectURL(file);
    setPreviewUrl(url);
  }
};
```

**Issue:** Nếu user chọn nhiều files liên tiếp, các URL cũ không được revoke.

**Fix:**
```tsx
const handleFilesSelected = (files: File[]) => {
  // Revoke previous URL before creating new one
  if (previewUrl) {
    URL.revokeObjectURL(previewUrl);
  }

  const file = files[0];
  if (file) {
    setSelectedFile(file);
    const url = URL.createObjectURL(file);
    setPreviewUrl(url);
  }
};
```

---

### 2. **Missing Memoization - Performance**
**File:** `image-lightbox.tsx` (lines 38-44)
**Impact:** Re-creates handlers on every render, có thể trigger unnecessary re-renders

**Current Code:**
```tsx
const handlePrev = useCallback(() => {
  if (hasPrev) onNavigate(currentIndex - 1);
}, [hasPrev, currentIndex, onNavigate]);
```

**Issue:** Dependencies `hasPrev` & `hasNext` là derived values, không cần trong deps array.

**Fix:**
```tsx
const handlePrev = useCallback(() => {
  if (currentIndex > 0) onNavigate(currentIndex - 1);
}, [currentIndex, onNavigate]);

const handleNext = useCallback(() => {
  if (currentIndex < images.length - 1) onNavigate(currentIndex + 1);
}, [currentIndex, images.length, onNavigate]);
```

---

### 3. **Stale Closure Bug**
**File:** `edit-specs-modal.tsx` (line 37)
**Impact:** `initialPairs` chỉ compute lần đầu, không update khi `specs` prop thay đổi giữa các lần mở modal

**Current Code:**
```tsx
const initialPairs = useMemo(() => specsToLocalPairs(specs), [specs]);
const [pairs, setPairs] = useState<SpecPair[]>(initialPairs);
```

**Issue:** `useState(initialPairs)` chỉ chạy lần mount đầu tiên. Nếu modal đóng rồi mở lại với specs khác, state không reset.

**Current Workaround:** Code đã có `handleOpenChange` reset state, nhưng implementation có thể cleaner.

**Better Fix:**
```tsx
// Remove useMemo, directly reset in handleOpenChange
const [pairs, setPairs] = useState<SpecPair[]>([]);

const handleOpenChange = (isOpen: boolean) => {
  if (isOpen) {
    setPairs(specsToLocalPairs(specs));
  }
  onOpenChange(isOpen);
};
```

---

## Medium Priority Improvements

### 4. **Code Duplication - Mock Data Generators**
**Files:** `reference-files.data.ts`, `comments.data.ts`

**Pattern lặp lại:**
```tsx
// reference-files.data.ts (lines 20-42)
let refIndex = 1;
tasksForRefs.forEach((task, taskIdx) => {
  const fileCount = 1 + (taskIdx % 3);
  for (let f = 0; f < fileCount; f++) {
    mockReferenceFiles.push({ ... });
    refIndex++;
    if (refIndex > 15) break;
  }
  if (refIndex > 15) return;
});

// comments.data.ts (lines 36-63) - Tương tự
```

**Recommendation:** Extract thành helper function:
```tsx
// mock-factory.ts
export function generateMockItems<T>(
  config: {
    maxItems: number;
    tasks: MockTask[];
    itemsPerTask: (taskIdx: number) => number;
    createItem: (index: number, task: MockTask) => T;
  }
): T[] { ... }
```

**YAGNI Check:** ⚠️ Trừ khi có thêm 2+ nơi dùng pattern này, chưa cần extract ngay.

---

### 5. **Console.log Statements - Development Code**
**Files:** 7 files có `console.log` (41, 37, 57, 71, 85, 70, 46 lines)

**Examples:**
- `reference-files-section.tsx:41` - "Files selected"
- `version-upload-form.tsx:37` - "Upload new version"
- `comments-section.tsx:46` - "New comment"

**Recommendation:**
- Option 1: Remove before production deployment
- Option 2: Wrap trong debug flag `if (__DEV__) console.log(...)`
- Option 3: Use proper logging library (pino, winston)

**Current Status:** Acceptable cho demo/mock code nhưng nên cleanup trước production.

---

### 6. **Missing Error Boundaries**
**File:** `task-detail-drawer.tsx`

**Issue:** Nếu 1 section component throw error, toàn bộ drawer crash.

**Recommendation:** Wrap sections trong ErrorBoundary:
```tsx
<ErrorBoundary fallback={<SectionError />}>
  <CommentsSection task={task} />
</ErrorBoundary>
```

**YAGNI Check:** ⚠️ Có thể defer nếu chưa encounter production errors.

---

### 7. **Hard-coded Permission Check**
**File:** `reference-files-section.tsx` (line 25)

```tsx
const currentUser = mockUsers[0]; // Mock: assume first user is current
const canUpload = currentUser?.role === 'ADMIN' || currentUser?.role === 'STAFF';
```

**Issue:** Hard-coded mock user, không scalable.

**Recommendation:** Extract thành hook hoặc context:
```tsx
const { user, can } = useAuth();
const canUpload = can('upload:reference-files');
```

**Priority:** Medium - chờ implement auth system.

---

### 8. **Component Size Warning**
**Files exceeding 150 lines:**
- `image-lightbox.tsx` - 154 lines
- `task-info-section.tsx` - 148 lines
- `design-versions-section.tsx` - 148 lines
- `comments-section.tsx` - 153 lines

**Status:** Gần limit 200 lines (dev rule), acceptable. Chưa cần split ngay.

**Monitor:** Nếu components tiếp tục phát triển → extract sub-components.

---

## Low Priority Suggestions

### 9. **Accessibility - Keyboard Navigation**
**File:** `image-lightbox.tsx` (lines 52-72)

**Good:** Đã implement keyboard navigation (Arrow keys, Escape)
**Missing:**
- Focus trap trong modal
- ARIA labels cho navigation buttons
- Screen reader announcements

**Fix:**
```tsx
<Button
  aria-label={`Previous image ${currentIndex} of ${images.length}`}
  onClick={handlePrev}
  disabled={!hasPrev}
>
  <ChevronLeft className="h-8 w-8" />
</Button>
```

---

### 10. **TypeScript - Stricter Types**
**File:** `mock-factory.ts`

**Current:**
```tsx
const generateUUID = (prefix: string, index: number): string => { ... }
```

**Stricter:**
```tsx
type UUIDPrefix = '00000001' | '00000002' | '00000003' | '00000004' | '00000005' | '00000006';
const generateUUID = (prefix: UUIDPrefix, index: number): `${string}-${string}-${string}-${string}-${string}` => { ... }
```

**YAGNI:** ❌ Over-engineering, current types đủ.

---

### 11. **CSS Class Naming - Consistency**
**Pattern:** Sử dụng `cn()` utility tốt, nhưng có thể extract repeated class combinations.

**Example:**
```tsx
// Repeated pattern
className="text-sm text-muted-foreground"
className="text-xs text-muted-foreground"
```

**Recommendation:** Extract thành design tokens hoặc Tailwind @apply nếu repeat nhiều lần.
**YAGNI:** ❌ Chưa cần thiết, Tailwind philosophy khuyến khích inline classes.

---

## Positive Observations

### ✅ Excellent Practices Found

1. **Separation of Concerns:** Mock data, types, UI components tách biệt rõ ràng
2. **Type Safety:** Full TypeScript coverage, no `any` types
3. **Component Composition:** Proper use of atomic components (Button, Badge, Dialog)
4. **Naming Conventions:** Kebab-case files, PascalCase components, consistent
5. **File Size:** All components < 200 lines (dev rule compliance)
6. **Error Handling:** Graceful fallbacks (empty states, null checks)
7. **Build Success:** No TypeScript errors, Next.js build passed
8. **KISS Principle:** Components focused, single responsibility
9. **DRY:** Good reuse of ImageLightbox, FileUploadZone across multiple components
10. **React Hooks:** Proper use of useState, useCallback, useEffect với correct dependencies

---

## Security Audit

### ✅ No Vulnerabilities Found

**Checked:**
- ❌ No `dangerouslySetInnerHTML` usage
- ❌ No `eval()` calls
- ❌ No SQL injection vectors (mock data only)
- ✅ XSS Prevention: User input properly escaped (React default behavior)
- ✅ No credential leaks in code
- ✅ File upload uses react-dropzone (client-side only, demo)

**Notes:**
- File upload là demo only (console.log), không có actual server upload → no upload vulnerabilities
- Modal inputs sanitized by React
- No API calls → no CSRF/injection risks trong scope này

---

## Performance Analysis

### Metrics

**Build Time:** 8.6s (acceptable)
**Static Pages:** 14 pages generated
**Bundle Size:** Not measured (recommend lighthouse audit)

### Findings

✅ **Good:**
- Proper use of `useCallback` cho event handlers
- `useMemo` cho expensive computations (edit-specs-modal)
- Components lazy-loadable (dynamic imports possible)

⚠️ **Concerns:**
- `mockDesignVersions.filter()` runs on every render (design-versions-section line 34)
- `mockComments.filter().sort()` runs on every render (comments-section line 33)

**Fix:**
```tsx
const versions = useMemo(
  () => mockDesignVersions
    .filter((v) => v.taskId === task.id)
    .sort((a, b) => b.version - a.version),
  [task.id]
);
```

**Impact:** Low - mock arrays nhỏ (<50 items), performance impact minimal.

---

## Architecture Review

### Component Structure

```
drawer/
├── task-detail-drawer.tsx     (Orchestrator - 81 lines)
├── task-info-section.tsx      (148 lines)
├── print-specs-section.tsx    (68 lines)
├── comments-section.tsx       (153 lines)
├── reference-files-section.tsx (100 lines)
├── design-versions-section.tsx (148 lines)
└── shared/
    ├── image-lightbox.tsx     (154 lines)
    ├── file-upload-zone.tsx   (84 lines)
    ├── edit-specs-modal.tsx   (128 lines)
    ├── version-upload-form.tsx (121 lines)
    └── quick-status-actions.tsx (77 lines)
```

**Assessment:**
- ✅ Clear hierarchy: Orchestrator → Sections → Shared utilities
- ✅ Reusable components (ImageLightbox, FileUploadZone)
- ✅ Single responsibility principle
- ⚠️ Có thể group shared components vào subfolder: `drawer/shared/`

---

## Recommended Actions

### Must Fix (Before Production)

1. ❌ Fix memory leak: Revoke old object URLs trong `version-upload-form.tsx`
2. ❌ Remove console.log statements hoặc wrap trong debug flag

### Should Fix (Next Sprint)

3. ⚠️ Add memoization cho filtered/sorted arrays trong Comments & Versions sections
4. ⚠️ Fix stale closure bug trong `edit-specs-modal.tsx`
5. ⚠️ Optimize useCallback dependencies trong `image-lightbox.tsx`

### Optional (Future Improvements)

6. 💡 Extract mock data generator pattern nếu có thêm use cases
7. 💡 Add ErrorBoundary cho sections
8. 💡 Improve accessibility: ARIA labels, focus management
9. 💡 Create auth context thay hard-coded user check

---

## YAGNI/KISS/DRY Compliance

**YAGNI:** ✅ Pass - No over-engineering detected
- Components focused on current requirements
- No premature abstractions
- Features align với design specs

**KISS:** ✅ Pass - Simple, readable code
- Straightforward component logic
- No complex state machines
- Clear data flow

**DRY:** ⚠️ Minor violations
- Mock data generators có slight duplication (acceptable)
- ImageLightbox & FileUploadZone reused properly
- Some CSS classes repeated (Tailwind convention)

**Overall:** 9/10 - Excellent adherence to principles

---

## Metrics Summary

| Metric | Value | Status |
|--------|-------|--------|
| Total Files | 13 components + 6 mock files | ✅ |
| Total Lines | ~1,350 lines | ✅ |
| Max File Size | 154 lines | ✅ (< 200 limit) |
| TypeScript Errors | 0 | ✅ |
| Build Status | Success | ✅ |
| Console.logs | 7 instances | ⚠️ |
| Security Issues | 0 critical | ✅ |
| Performance Issues | 1 minor (memo) | ⚠️ |
| Memory Leaks | 1 potential | ❌ |

---

## Conclusion

Code quality **production-ready** với minor fixes needed. Implementation tuân thủ development rules, architectural patterns rõ ràng, type safety đầy đủ. Main concerns: memory leak risk và development console.logs cần cleanup trước deploy.

**Next Steps:**
1. Fix URL.createObjectURL memory leak
2. Remove/wrap console.log statements
3. (Optional) Add memoization cho performance optimization
4. Deploy to staging for integration testing

---

## Unresolved Questions

1. **Auth System Timeline:** Khi nào implement real authentication để replace hard-coded user checks?
2. **Logging Strategy:** Team có plan dùng logging library nào (pino, winston, sentry) cho production?
3. **Error Monitoring:** Có setup error tracking (Sentry, LogRocket) chưa?
4. **Bundle Size Target:** Có metrics target cho bundle size/performance không?
5. **Plan File:** Không tìm thấy plan file cho Sheet Detail Enhancements - có bị missing không?
