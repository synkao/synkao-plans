# Code Review: Constants Translation to English

**Date:** 2026-01-02
**Reviewer:** code-reviewer (a4a769f)
**File:** `apps/web/src/features/design/lib/constants.ts`

---

## Scope

- **Files reviewed:** 1 file
- **Lines changed:** 16 string values
- **Review focus:** String translation from Vietnamese to English
- **Related files:** 10 components using these constants

---

## Overall Assessment

✅ **PASS** - Translation hoàn thành đúng theo quy định CLAUDE.md (UI text phải dùng tiếng Anh). Code không có vấn đề security, syntax errors, hoặc logic changes.

**Linting status:** Có lỗi linting trong codebase (2 errors, 3 warnings) nhưng **KHÔNG liên quan** đến thay đổi này:
- `sidebar.tsx`: Math.random() purity issue
- `use-kanban-state.ts`: setState in effect issue
- `task-card.tsx`: Missing alt prop

---

## Security Audit

✅ **No security issues detected**

- No sensitive data exposure
- No injection vulnerabilities
- Pure string constant changes
- No runtime logic modifications
- No new dependencies introduced

---

## Translation Consistency Review

### ✅ KANBAN_COLUMNS (4 changes)
| Before | After | Status |
|--------|-------|--------|
| Cần làm | To Do | ✅ Correct |
| Đang làm | In Progress | ✅ Correct |
| Cần sửa | Revision | ✅ Correct |
| Hoàn thành | Done | ✅ Correct |

**Note:** "Review" giữ nguyên (đã là tiếng Anh)

### ✅ PRIORITY_CONFIG (4 changes)
| Before | After | Status |
|--------|-------|--------|
| Gấp | Urgent | ✅ Correct |
| Cao | High | ✅ Correct |
| Bình thường | Normal | ✅ Correct |
| Thấp | Low | ✅ Correct |

### ✅ STATUS_CONFIG (4 changes)
Identical to KANBAN_COLUMNS names - ✅ Consistent

### ✅ SOURCE_CONFIG (3 changes)
| Before | After | Status |
|--------|-------|--------|
| Mới | New | ✅ Correct |
| Làm lại | Redo | ✅ Correct |
| Khiếu nại | Complaint | ✅ Correct |

---

## Code Style Compliance

✅ **Fully compliant** with:
- CLAUDE.md rule: "UI text trong app phải dùng tiếng Anh"
- TypeScript conventions
- Object property formatting
- String literal consistency
- No code formatting changes needed

---

## Impact Analysis

### Files using these constants (10 files):
1. `priority-badge.tsx` - Uses PRIORITY_CONFIG
2. `source-badge.tsx` - Uses SOURCE_CONFIG
3. `status-badge.tsx` - Uses STATUS_CONFIG
4. `status-dropdown.tsx` - Uses STATUS_CONFIG
5. `kanban-board.tsx` - Uses KANBAN_COLUMNS
6. `workspace-card.tsx` - Uses config objects
7. `backlog-filter-bar.tsx` - Uses config objects
8. `backlog-stats-bar.tsx` - Uses config objects
9. `design-versions-section.tsx` - Uses config objects
10. `utils.ts` - Utility functions

**Impact:** All UI labels displayed to users will change from Vietnamese to English.

---

## Verification Checklist

- [x] No TypeScript type changes
- [x] No interface/type modifications
- [x] Consistent translation across all configs
- [x] No typos in English labels
- [x] Proper capitalization (Title Case for multi-word labels)
- [x] No code logic changes
- [x] No security vulnerabilities introduced
- [x] Compliant with CLAUDE.md language rules

---

## Recommendations

### ✅ No critical actions required

### Optional improvements (not blocking):

1. **Build verification:**
   ```bash
   npm run build
   ```
   Verify build succeeds with new constants

2. **Visual regression testing:**
   Check UI components render correctly with English labels:
   - Kanban board columns
   - Priority badges
   - Status badges
   - Source badges
   - Filter dropdowns

3. **Consider i18n future:**
   If multilingual support needed later, migrate to i18n system:
   ```typescript
   // Future enhancement
   import { t } from '@/lib/i18n'
   name: t('kanban.column.todo')
   ```

---

## Metrics

- **Type Coverage:** N/A (no type changes)
- **Test Coverage:** N/A (constants only)
- **Linting Issues:** 0 new issues (existing issues unrelated)
- **Build Status:** Not verified (recommend testing)
- **Security Score:** ✅ Pass

---

## Positive Observations

✅ Clean, focused change
✅ Consistent naming conventions
✅ Proper separation of concerns (constants file)
✅ Type-safe enum-like structure
✅ Follows project language standards

---

## Summary

**Change type:** String localization (vi → en)
**Risk level:** 🟢 Low
**Approval status:** ✅ Approved

Thay đổi này hoàn toàn an toàn và tuân thủ đúng quy định CLAUDE.md về việc dùng tiếng Anh cho UI text. Không có security risks, type errors, hoặc logic changes. Recommended action: Merge sau khi verify build thành công.

---

## Unresolved Questions

None.
