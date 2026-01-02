# Code Review: Phase 2 Design Module Implementation

**Reviewer:** code-reviewer
**Date:** 2026-01-02
**Scope:** Design Module - Kanban, Backlog, Drawer components
**Status:** ⚠️ Critical Issues Found - DO NOT MERGE

---

## Scope

### Files Reviewed
- **Kanban Components** (5 files): kanban-board.tsx, kanban-column.tsx, task-card.tsx, sortable-task-card.tsx, drag-overlay-card.tsx
- **Backlog Components** (3 files): backlog-table.tsx, backlog-filter-bar.tsx, backlog-stats-bar.tsx
- **Drawer Components** (5 files): task-detail-drawer.tsx, task-info-section.tsx, design-notes-section.tsx, design-versions-section.tsx, timeline-section.tsx
- **Pages** (2 files): design/backlog/page.tsx, design/workspace/[id]/page.tsx
- **Hooks** (2 files): use-kanban-state.ts, use-tasks-by-status.ts
- **Utils & Constants** (2 files): lib/utils.ts, lib/constants.ts

**Total:** 30 files, ~1,357 LOC analyzed

### Build Status
✅ Build successful - TypeScript compilation passes
❌ Lint fails - 7 errors, 2 warnings
⚠️ No tests - 0 test files found

---

## Overall Assessment

Code quality: **Medium-Low**
Implementation follows basic React patterns but contains **3 critical issues** and **4 high-priority issues** that must be fixed before merge. No security vulnerabilities or data loss risks detected. Performance adequate for mock demo.

---

## ❌ Critical Issues

### 1. **Dynamic Tailwind Classes Won't Work** (BREAKING)
**Impact:** Visual bugs - colored dots will not render correctly
**Severity:** CRITICAL
**Files affected:**
- `kanban-column.tsx:27`
- `status-dropdown.tsx:29, 41`

**Problem:**
```tsx
// ❌ WRONG - Tailwind purge will remove these
<div className={cn('h-3 w-3 rounded-full', `bg-${column.color}-500`)} />
<span className={`h-2 w-2 rounded-full bg-${config.color}-500`} />
```

Tailwind uses static analysis at build time. Dynamic string interpolation like `bg-${color}-500` will NOT generate CSS classes. These elements will render without background colors.

**Fix:**
Use mapping object with complete class names:

```tsx
// ✅ CORRECT
const COLOR_CLASSES: Record<string, string> = {
  blue: 'bg-blue-500',
  amber: 'bg-amber-500',
  violet: 'bg-violet-500',
  red: 'bg-red-500',
  emerald: 'bg-emerald-500',
  gray: 'bg-gray-500',
};

<div className={cn('h-3 w-3 rounded-full', COLOR_CLASSES[column.color])} />
```

Or use data attributes + arbitrary values:
```tsx
<div className="h-3 w-3 rounded-full" style={{ backgroundColor: `var(--${color}-500)` }} />
```

**References:**
- https://tailwindcss.com/docs/content-configuration#dynamic-class-names
- Constants already define color strings ('blue', 'amber', etc.) in `lib/constants.ts`

---

### 2. **React Component Created During Render** (PERFORMANCE)
**Impact:** Performance degradation, component state resets on every render
**Severity:** CRITICAL
**Files affected:** `backlog-table.tsx:94-104`

**Problem:**
```tsx
// ❌ WRONG - Component recreated on every render
export function BacklogTable({ tasks, onTaskClick }: BacklogTableProps) {
  const SortableHeader = ({ field, children }) => (  // ← Created inside render
    <th onClick={() => handleSort(field)}>
      {children}
    </th>
  );

  return <SortableHeader field="taskCode">Mã task</SortableHeader>;
}
```

**Why it's bad:**
- New component instance created every render → breaks React reconciliation
- Performance penalty: unnecessary re-renders
- Violates React rules: `react-hooks/static-components`

**Fix:**
Move component outside parent or convert to render function:

```tsx
// ✅ Option 1: Extract component outside
interface SortableHeaderProps {
  field: SortField;
  onSort: (field: SortField) => void;
  children: React.ReactNode;
}

function SortableHeader({ field, onSort, children }: SortableHeaderProps) {
  return (
    <th onClick={() => onSort(field)} className="...">
      <div className="flex items-center gap-1">
        {children}
        <ArrowUpDown className="h-3 w-3" />
      </div>
    </th>
  );
}

// ✅ Option 2: Render function (simpler for this case)
const renderSortableHeader = (field: SortField, children: React.ReactNode) => (
  <th onClick={() => handleSort(field)} className="...">
    <div className="flex items-center gap-1">
      {children}
      <ArrowUpDown className="h-3 w-3" />
    </div>
  </th>
);
```

**Lint errors:**
```
120:16  error  Cannot create components during render  react-hooks/static-components
121:16  error  Cannot create components during render  react-hooks/static-components
122:16  error  Cannot create components during render  react-hooks/static-components
123:16  error  Cannot create components during render  react-hooks/static-components
125:16  error  Cannot create components during render  react-hooks/static-components
```

---

### 3. **Files Not Committed to Git**
**Impact:** Code exists only locally, not in version control
**Severity:** CRITICAL (for collaboration)

**Status:**
```
Untracked files:
  apps/web/src/features/design/components/
  apps/web/src/features/design/hooks/
  apps/web/src/features/design/index.ts
  apps/web/src/features/design/lib/
```

All 30 design module files are untracked. Must commit before review completion.

---

## 🔴 High Priority Findings

### 4. **No Tests - Zero Coverage**
**Severity:** HIGH
**Impact:** No verification of functionality, high regression risk

**Missing tests for:**
- Drag & drop functionality (kanban-board.tsx)
- Sorting logic (backlog-table.tsx:38-74)
- Filter logic (backlog/page.tsx:31-51)
- State management (use-kanban-state.ts)

**Recommendation:**
Add tests for core features before Phase 2 completion:
```tsx
// Priority test cases
describe('KanbanBoard', () => {
  it('should update task status on drag end');
  it('should not update if dropped on same column');
  it('should handle invalid drop targets');
});

describe('BacklogTable', () => {
  it('should sort by priority correctly');
  it('should toggle sort direction');
  it('should handle empty task list');
});
```

---

### 5. **useEffect Dependency Missing in Hook**
**Severity:** HIGH
**Files:** `use-kanban-state.ts:21-27`

**Problem:**
```tsx
useEffect(() => {
  if (workspaceId) {
    setTasks(mockTasks.filter((task) => task.workspaceId === workspaceId));
  } else {
    setTasks(mockTasks);
  }
}, [workspaceId]); // ❌ Missing mockTasks dependency
```

**Issue:** If `mockTasks` changes (e.g., from API update), effect won't re-run. Currently safe because `mockTasks` is static import, but breaks if data source becomes dynamic.

**Fix:**
```tsx
// ✅ Add mockTasks to deps
}, [workspaceId, mockTasks]);

// Or better: useMemo instead of useEffect
const tasks = useMemo(() => {
  if (workspaceId) {
    return mockTasks.filter((task) => task.workspaceId === workspaceId);
  }
  return mockTasks;
}, [workspaceId, mockTasks]);
```

---

### 6. **Type Safety: Unsafe Type Assertions**
**Severity:** HIGH
**Files:** `kanban-board.tsx:41-42`

**Problem:**
```tsx
const taskId = active.id as string;  // ❌ Unsafe cast
const newStatus = over.id as MockTask['status'];  // ❌ Unsafe cast
```

**Issue:** No runtime validation. If `over.id` is not a valid status, will corrupt data.

**Fix:**
```tsx
// ✅ Add validation
const newStatus = over.id as string;
if (!['TODO', 'IN_PROGRESS', 'REVIEW', 'REVISION', 'DONE'].includes(newStatus)) {
  console.warn('Invalid drop target:', newStatus);
  return;
}

// Or use type guard
function isValidStatus(status: string): status is MockTask['status'] {
  return ['TODO', 'IN_PROGRESS', 'REVIEW', 'REVISION', 'DONE'].includes(status);
}

const newStatus = over.id as string;
if (!isValidStatus(newStatus)) return;
```

---

### 7. **Performance: useMemo Dependencies Suboptimal**
**Severity:** HIGH
**Files:** `backlog-table.tsx:38-74`

**Problem:**
```tsx
const sortedTasks = useMemo(() => {
  const sorted = [...tasks];
  sorted.sort((a, b) => { /* complex sorting */ });
  return sorted;
}, [tasks, sortField, sortDirection]);  // Re-sorts on ANY task change
```

**Issue:** Large task arrays will re-sort frequently. For 100+ tasks, noticeable lag.

**Current:** ~1,357 LOC suggests small dataset, but backlog could grow.

**Optimization (if needed later):**
- Debounce search filter
- Virtual scrolling for >100 rows (react-window)
- Sort only visible page if paginated

**Decision:** Accept as-is for Phase 2 mock demo. Document for future optimization.

---

## 🟡 Medium Priority Improvements

### 8. **Code Duplication: Priority Sorting Logic**
**Files:** `backlog-table.tsx:57-62`

```tsx
// Duplicated priority order definition
const priorityOrder = { URGENT: 0, HIGH: 1, NORMAL: 2, LOW: 3 };
```

**Better:** Extract to constants
```tsx
// lib/constants.ts
export const PRIORITY_ORDER: Record<MockTask['priority'], number> = {
  URGENT: 0,
  HIGH: 1,
  NORMAL: 2,
  LOW: 3,
};
```

**Impact:** Low - only one instance currently, but violates DRY.

---

### 9. **Accessibility: Missing ARIA Labels**
**Files:** kanban-column.tsx, backlog-table.tsx

**Issues:**
- Checkbox inputs missing `aria-label` (backlog-table.tsx:113, 139)
- Draggable items missing `aria-grabbed` state
- No keyboard navigation for drag-drop (limitation of @dnd-kit)

**Fix:**
```tsx
<input
  type="checkbox"
  aria-label={selectedIds.size === tasks.length ? "Deselect all" : "Select all"}
  checked={selectedIds.size === tasks.length}
/>
```

**Impact:** Medium - affects screen reader users.

---

### 10. **Error Handling: No Boundary for Drag Operations**
**Files:** kanban-board.tsx

No error boundary wraps DndContext. If drag operation throws, entire board crashes.

**Fix:** Wrap with ErrorBoundary or add try-catch in handlers.

---

### 11. **State Management: Local State Not Persisted**
**Files:** use-kanban-state.ts

Task status changes lost on page refresh. Expected for mock demo, but needs documenting.

**Future:** Consider localStorage or backend sync.

---

## 🟢 Low Priority Suggestions

### 12. **Code Style: Inconsistent Optional Chaining**
```tsx
// Inconsistent
const assignee = task.assigneeId ? mockUsers.find(...) : null;  // Ternary
assignee?.name  // Optional chaining
```

Use optional chaining consistently for brevity.

---

### 13. **Magic Numbers: Hardcoded Values**
```tsx
<div className="h-24 items-center" />  // 24 = 96px
```

Extract to constants if reused (e.g., `THUMBNAIL_HEIGHT`).

---

### 14. **Vietnamese Strings Should Be i18n**
Hardcoded Vietnamese text in components (e.g., "Mã task", "Cần làm"). For internationalization, extract to i18n file.

**Decision:** Accept for now - spec requires Vietnamese UI.

---

## ✅ Positive Observations

1. **Clean Component Structure:** Well-organized folders (kanban/, backlog/, drawer/)
2. **Type Safety:** Consistent use of TypeScript, proper interface exports
3. **DRY Utilities:** Good extraction of `formatRelativeTime`, `groupTasksByStatus`
4. **Prop Drilling Avoided:** Proper use of callbacks (`onTaskClick`, `onTasksChange`)
5. **Accessibility Basics:** Semantic HTML (`<table>`, `<th>`, `<td>`)
6. **No Security Issues:** No XSS vulnerabilities detected (no `dangerouslySetInnerHTML`)
7. **Proper Key Props:** Correct `key={task.id}` in all map operations
8. **Clean Separation:** UI components don't contain business logic

---

## Recommended Actions

### Must Fix Before Merge (Critical)
1. **Fix dynamic Tailwind classes** in kanban-column.tsx, status-dropdown.tsx → Use mapping object
2. **Extract SortableHeader component** from backlog-table.tsx render → Move outside or use render function
3. **Commit all files to git** → `git add apps/web/src/features/design`

### Should Fix Before Phase 2 Complete (High Priority)
4. **Add validation for drag-drop status** in kanban-board.tsx → Type guard for `over.id`
5. **Fix useEffect dependencies** in use-kanban-state.ts → Add mockTasks or refactor to useMemo
6. **Add basic tests** for drag-drop and sorting → At least smoke tests
7. **Fix lint errors** → Run `npm run lint -- --fix` where possible

### Consider for Next Phase (Medium)
8. **Add ARIA labels** to interactive elements
9. **Add ErrorBoundary** around DndContext
10. **Extract PRIORITY_ORDER** to constants

### Optional Improvements (Low)
11. Document state persistence limitation
12. Standardize optional chaining style

---

## Metrics

- **Type Coverage:** ~95% (explicit types for all props/exports)
- **Test Coverage:** 0% (no test files)
- **Linting Issues:** 9 total (7 errors, 2 warnings)
  - 5 errors: `react-hooks/static-components` (SortableHeader)
  - 1 error: `react-hooks/purity` (unrelated to design module)
  - 1 error: `Math.random` in sidebar (unrelated)
- **Build Status:** ✅ Passes (no TypeScript errors)
- **Bundle Impact:** Not measured (acceptable for demo)

---

## Next Steps

1. **Developer:** Fix 3 critical issues (Tailwind classes, SortableHeader, git commit)
2. **Developer:** Address 4 high-priority issues (validation, deps, tests, lint)
3. **Code Reviewer:** Re-review after fixes applied
4. **QA:** Manual test drag-drop across all statuses
5. **Product:** Verify UI matches design spec

---

## Unresolved Questions

1. **Testing strategy:** Unit tests vs E2E for drag-drop? (dnd-kit testing is complex)
2. **Performance target:** Expected max task count in backlog? (impacts optimization priority)
3. **State persistence:** Should kanban state persist to localStorage for demo?
4. **i18n requirement:** Is Vietnamese-only acceptable or plan for multi-language?
5. **Error handling:** What should happen if drag operation fails? (silent fail vs user notification)

---

**Verdict:** 🔴 **DO NOT MERGE** - Critical issues must be fixed first
**Estimated fix time:** 2-3 hours for critical + high priority items
**Re-review required:** Yes, after fixes applied
