# Debug Report: Design Backlog Empty Data

**Issue ID:** Design Backlog No Data
**Reported:** 2026-01-05
**Status:** Root Cause Identified
**Severity:** High - Feature không hoạt động

---

## 1. Executive Summary

Màn hình Design Backlog (URL: `/design/backlog`) hiển thị 0 tasks mặc dù có 40 tasks TODO trong mock data. Root cause: commit `5b2e6fb` đã remove logic điều kiện `hasWorkspace`, khiến TẤT CẢ tasks được gán `workspaceId`, vi phạm điều kiện filter backlog.

**Business Impact:**
- User không thể xem danh sách tasks chờ assign
- Feature bulk actions (assign workspace/designer, set priority) không dùng được
- Workflow backlog → workspace bị gián đoạn

**Root Cause:**
Mock data generator hardcode `workspaceId = WORKSPACE_IDS.main` cho TẤT CẢ tasks, trong khi backlog filter yêu cầu tasks phải KHÔNG CÓ workspaceId.

---

## 2. Technical Analysis

### 2.1 Component Structure

**Page:** `/apps/web/src/app/(main)/design/backlog/page.tsx`

**Data Flow:**
```
mockTasks (mocks/data/tasks.data.ts)
  ↓
backlogTasks filter (line 31-33)
  ↓
filteredTasks (line 36-56)
  ↓
BacklogTable component
```

**Filter Logic (line 31-33):**
```typescript
const backlogTasks = useMemo(() => {
  return mockTasks.filter((task) =>
    task.status === DesignTaskStatus.TODO && !task.workspaceId
  );
}, []);
```

Điều kiện backlog:
1. `status === TODO` ✅ (có 40 tasks TODO)
2. `!workspaceId` ❌ (TẤT CẢ tasks đều có workspaceId)

### 2.2 Mock Data Analysis

**File:** `/apps/web/src/mocks/data/tasks.data.ts`

**Task Distribution:**
- TODO: 40 tasks
- IN_PROGRESS: 8 tasks
- REVIEW: 6 tasks
- REVISION: 3 tasks
- APPROVED: 3 tasks
- **Total:** 60 tasks

**Current Code (line 69):**
```typescript
workspaceId: WORKSPACE_IDS.main,  // HARDCODED cho TẤT CẢ
```

**Previous Code (commit `bfab81d`):**
```typescript
const hasWorkspace = status !== 'TODO';
workspaceId: hasWorkspace ? WORKSPACE_IDS.main : undefined,
```

### 2.3 Git History

**Commit Timeline:**

| Commit | Date | Change |
|--------|------|--------|
| `bfab81d` | ~Jan 4 | Mock data có logic điều kiện `hasWorkspace` |
| `5b2e6fb` | Jan 5 01:25 | Remove logic, hardcode `workspaceId = WORKSPACE_IDS.main` |

**Diff Extract:**
```diff
- workspaceId: hasWorkspace ? WORKSPACE_IDS.main : undefined,
+ workspaceId: WORKSPACE_IDS.main,
```

**Impact:** Logic backlog bị phá vỡ khi hardcode workspaceId.

### 2.4 Component Behavior

**BacklogStatsBar** (`backlog-stats-bar.tsx`):
- Input: `backlogTasks` (empty array)
- Counts by priority: ALL = 0
- Display: Urgent 0, High 0, Normal 0, Low 0 ✅ Match screenshot

**BacklogTable** (`backlog-table.tsx`):
- Input: `filteredTasks` (empty array)
- Render: Empty table ✅ Match screenshot

**BacklogActionBar:**
- Hidden khi `selectedIds.size === 0`
- Không hiển thị ✅ Correct

### 2.5 API Calls

**Analysis:** Page KHÔNG gọi API, sử dụng mock data trực tiếp.

```typescript
import { mockTasks, mockWorkspaces, getDesigners } from '@/mocks/data';
```

**MSW Handlers:**
- `/api/tasks/bulk/assign-workspace` ✅ Có
- `/api/tasks/bulk/assign-designer` ✅ Có
- `/api/tasks/bulk/set-priority` ✅ Có
- **Note:** Chỉ có bulk actions, KHÔNG có GET endpoint

### 2.6 Type Definitions

**MockTask Interface** (`mocks/types.ts`):
```typescript
export interface MockTask {
  id: string;
  tenantId: string;
  taskCode: string;
  orderId: string;
  orderItemId?: string;
  workspaceId?: string;  // OPTIONAL - undefined = backlog
  columnId?: string;
  // ...
}
```

Type definition đúng (workspaceId optional), nhưng mock data không tuân thủ.

---

## 3. Evidence

### 3.1 Mock Data Verification

**Command:**
```bash
grep -n "workspaceId" apps/web/src/mocks/data/tasks.data.ts
```

**Result:**
```
69:    workspaceId: WORKSPACE_IDS.main,
```

### 3.2 Task Count Verification

**Command:**
```bash
grep -c "DesignTaskStatus.TODO" apps/web/src/mocks/data/tasks.data.ts
```

**Expected:** 40 tasks with TODO status
**Result:** Confirmed, line 19 defines `Array(40).fill(DesignTaskStatus.TODO)`

### 3.3 Filter Result

**Executed in Browser Console (simulated):**
```javascript
const backlogTasks = mockTasks.filter(
  task => task.status === 'TODO' && !task.workspaceId
);
console.log(backlogTasks.length); // Output: 0
```

---

## 4. Supporting Files

### 4.1 Affected Components

1. **Page:** `apps/web/src/app/(main)/design/backlog/page.tsx`
   - Line 31-33: Filter logic
   - Line 150: StatsBar nhận empty array
   - Line 163: Table nhận empty array

2. **Mock Data:** `apps/web/src/mocks/data/tasks.data.ts`
   - Line 69: Hardcoded workspaceId
   - Line 18-24: Status distribution

3. **Factory:** `apps/web/src/mocks/factories/mock-factory.ts`
   - Line 16-19: WORKSPACE_IDS definition

### 4.2 Related Components

- `backlog-stats-bar.tsx`: Hiển thị stats (0/0/0/0) - hoạt động đúng với empty data
- `backlog-table.tsx`: Render empty table - hoạt động đúng với empty data
- `backlog-filter-bar.tsx`: Filter controls - không ảnh hưởng vì data đã empty
- `backlog-action-bar.tsx`: Bulk actions - hidden đúng logic

---

## 5. Root Cause Summary

**Primary Issue:**
Mock data generator file (`tasks.data.ts` line 69) hardcode `workspaceId = WORKSPACE_IDS.main` cho TẤT CẢ tasks, bao gồm TODO tasks.

**Conflicting Logic:**
Backlog filter (`page.tsx` line 32) yêu cầu tasks phải có `!workspaceId` (undefined/null).

**Result:**
0 tasks match điều kiện → Empty backlog.

**Why It Happened:**
Commit `5b2e6fb` refactor mock data, remove logic điều kiện `hasWorkspace = status !== 'TODO'`, dẫn đến regression.

---

## 6. Recommended Solution

### Option 1: Fix Mock Data (Preferred)

**File:** `apps/web/src/mocks/data/tasks.data.ts`

**Change:**
```typescript
// Line 69: Restore conditional logic
workspaceId: status !== DesignTaskStatus.TODO ? WORKSPACE_IDS.main : undefined,
```

**Pros:**
- Restore business logic: TODO tasks = backlog (no workspace)
- Match với commit `bfab81d` behavior
- Ít thay đổi code

**Cons:**
- Cần test lại Kanban board (tasks cần có workspace để hiển thị)

### Option 2: Change Filter Logic

**File:** `apps/web/src/app/(main)/design/backlog/page.tsx`

**Change:**
```typescript
// Line 32: Remove workspaceId check
return mockTasks.filter((task) => task.status === DesignTaskStatus.TODO);
```

**Pros:**
- Không thay đổi mock data
- Quick fix

**Cons:**
- Sai logic business: Backlog tasks nên chưa có workspace
- Có thể conflict với workspace management logic

### Option 3: Partial Mock Data

**File:** `apps/web/src/mocks/data/tasks.data.ts`

**Change:**
```typescript
// Set 50% TODO tasks không có workspace (backlog)
// 50% TODO tasks có workspace (đã assign nhưng chưa start)
const hasWorkspace = status !== DesignTaskStatus.TODO || idx % 2 === 0;
workspaceId: hasWorkspace ? WORKSPACE_IDS.main : undefined,
```

**Pros:**
- Mock data realistic hơn
- Test được cả 2 cases

**Cons:**
- Phức tạp hơn
- Cần clarify business logic

---

## 7. Preventive Measures

### 7.1 Testing Strategy

**Unit Test for Filter:**
```typescript
describe('Backlog Filter', () => {
  it('should return TODO tasks without workspace', () => {
    const tasks = [
      { status: 'TODO', workspaceId: undefined },
      { status: 'TODO', workspaceId: 'ws-1' },
      { status: 'IN_PROGRESS', workspaceId: undefined },
    ];
    const backlog = tasks.filter(t => t.status === 'TODO' && !t.workspaceId);
    expect(backlog).toHaveLength(1);
  });
});
```

### 7.2 Mock Data Validation

Thêm validation trong mock factory:
```typescript
// Validate backlog tasks
const backlogTasks = mockTasks.filter(t => t.status === 'TODO' && !t.workspaceId);
console.assert(backlogTasks.length > 0, 'Mock data must have backlog tasks');
```

### 7.3 Documentation

Update `codebase-summary.md`:
- Clarify: TODO tasks without workspaceId = backlog
- Document mock data business rules

---

## 8. Implementation Priority

| Fix | Priority | Effort | Risk |
|-----|----------|--------|------|
| Option 1 (Restore logic) | P0 | 5 min | Low |
| Add validation | P1 | 10 min | Low |
| Unit tests | P2 | 30 min | None |

**Recommended:** Option 1 + Validation

---

## 9. Test Plan

### 9.1 Verification Steps

**After fix:**
1. Check backlog page loads data ✓
2. Verify stats bar shows counts ✓
3. Test filter controls ✓
4. Test bulk actions ✓
5. Test Kanban board still works ✓

### 9.2 Regression Check

- Workspace selector vẫn hoạt động
- Kanban columns vẫn nhận tasks
- Task detail drawer vẫn mở được

---

## 10. Questions

**Q1:** Business logic cho TODO tasks trong workspace là gì?
- Có TODO tasks đã được assign workspace nhưng chưa start không?
- Backlog = TODO + no workspace, hay chỉ TODO?

**Q2:** Kanban board cần workspaceId cho TẤT CẢ tasks?
- Nếu có, Option 3 (partial mock) phù hợp hơn
- Nếu không, Option 1 OK

**Q3:** Real API sẽ trả data như thế nào?
- Endpoint GET `/api/tasks?status=TODO&workspaceId=null`?
- Hay endpoint riêng `/api/tasks/backlog`?

---

## Appendix

### A. File Locations

```
apps/web/src/
├── app/(main)/design/backlog/page.tsx          # Backlog page
├── features/design/components/backlog/
│   ├── backlog-table.tsx                       # Table component
│   ├── backlog-stats-bar.tsx                   # Stats display
│   ├── backlog-filter-bar.tsx                  # Filter controls
│   └── backlog-action-bar.tsx                  # Bulk actions
├── mocks/
│   ├── data/tasks.data.ts                      # Mock tasks (BUG HERE)
│   ├── factories/mock-factory.ts               # Constants
│   ├── handlers/workspaces.ts                  # MSW handlers
│   └── types.ts                                # Type definitions
```

### B. Related Commits

- `5b2e6fb`: feat(design): add workspace management and backlog bulk actions
- `bfab81d`: feat(design): implement workspace and backlog management components

### C. Environment

- Next.js 14 App Router
- Mock Service Worker (MSW) active
- No real backend calls
- Client-side filtering

---

**Report Generated:** 2026-01-05 09:11
**Investigator:** Claude Code Debugger
**Contact:** See git history for commit author
