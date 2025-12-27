# Brainstorm Report: Light Functional Glassmorphism Update

**Date:** 2025-12-25
**Status:** Approved
**Scope:** Replace dark glass → Light Functional Glassmorphism

---

## Problem Statement

Design system hiện tại (dark glass) cần thay thế bằng Light Functional Glassmorphism theo mock mới. Thêm Phase Colors cho workflow tracking.

## Decisions Made

| Decision | Choice |
|----------|--------|
| Theme direction | Light-only (no dark mode) |
| Phase colors | Required (Order/Design/Fulfill/Complete/Cancelled) |
| Typography | Keep Inter (no Be Vietnam Pro) |

---

## Current State Analysis

### globals.css Structure
- Light mode: `:root` with OKLCH colors
- Dark mode: `.dark` class (sẽ bị remove/simplify)
- Glass utilities: `.glass`, `.glass-dark` classes
- Status/Priority/Semantic colors: đã có

### Key Differences: Current vs Mock

| Aspect | Current | Mock Target |
|--------|---------|-------------|
| Glass opacity | 60% (`oklch(1 0 0 / 60%)`) | 70% (`rgba(255,255,255,0.7)`) |
| Glass hover | 70% | 85% |
| Border color | `oklch(0 0 0 / 10%)` | `rgba(255,255,255,0.2)` |
| Background | White (`oklch(1 0 0)`) | Slate-50 (`#F8FAFC`) |
| Brand color | Black (`oklch(0.205 0 0)`) | Blue-500 (`#3B82F6`) |
| Phase colors | Not exist | 5 new colors needed |

---

## Implementation Plan

### Phase 1: Update Core Colors (globals.css)

**1.1 Background & Surface Colors**
```css
/* Current */
--background: oklch(1 0 0);

/* Target - Slate-50 */
--background: oklch(0.984 0.003 247);
--surface-secondary: oklch(0.968 0.004 247); /* Slate-100 #F1F5F9 */
```

**1.2 Brand/Primary Color → Blue-500**
```css
/* Current (black) */
--primary: oklch(0.205 0 0);

/* Target - Blue-500 #3B82F6 */
--primary: oklch(0.623 0.214 259);
--primary-foreground: oklch(1 0 0);
```

**1.3 Glass Effect Update**
```css
/* Target */
--glass-bg: oklch(1 0 0 / 70%);
--glass-bg-hover: oklch(1 0 0 / 85%);
--glass-border: oklch(0.623 0.214 259 / 30%); /* Brand color tint on hover */
```

### Phase 2: Add Phase Colors (NEW)

```css
/* ===== PHASE COLORS ===== */
--phase-order: oklch(0.554 0.046 257);     /* #64748B Slate-500 */
--phase-design: oklch(0.769 0.188 70);     /* #F59E0B Amber-500 */
--phase-fulfill: oklch(0.586 0.220 292);   /* #8B5CF6 Violet-500 */
--phase-complete: oklch(0.627 0.194 149);  /* #10B981 Emerald-500 */
--phase-cancelled: oklch(0.505 0.017 257); /* #6B7280 Gray-500 */
```

**Tailwind theme inline cần thêm:**
```css
--color-phase-order: var(--phase-order);
--color-phase-design: var(--phase-design);
--color-phase-fulfill: var(--phase-fulfill);
--color-phase-complete: var(--phase-complete);
--color-phase-cancelled: var(--phase-cancelled);
```

### Phase 3: Simplify Dark Mode

**Option A: Remove entirely**
- Delete `.dark { ... }` block
- Remove `@custom-variant dark`
- Pros: Simpler, less maintenance
- Cons: No dark mode support

**Option B: Keep minimal (Recommended)**
- Keep dark mode variables but match light theme
- Future-proof nếu cần dark mode sau
- Just update colors to match new design

**Recommendation:** Option A - Remove entirely (per YAGNI/KISS)

### Phase 4: Update Status Badges Colors

Mock uses solid light backgrounds:
```css
/* Status badge backgrounds (light variants) */
--status-todo-bg: oklch(0.97 0.02 259);      /* Blue-50 */
--status-in-progress-bg: oklch(0.98 0.03 70); /* Amber-50 */
--status-review-bg: oklch(0.97 0.02 292);     /* Violet-50 */
--status-revision-bg: oklch(0.97 0.03 25);    /* Red-50 */
--status-done-bg: oklch(0.97 0.02 149);       /* Emerald-50 */
```

---

## Files to Update

| File | Changes |
|------|---------|
| `apps/web/src/app/globals.css` | Core CSS variables, remove dark mode |
| `docs/synkao-design-system.md` | Update documentation |

---

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Existing components break | Medium | Không có components nào implement từ design cũ |
| Color contrast issues | Low | Mock đã được thiết kế với high contrast |
| Missing dark mode | Low | Users chủ yếu dùng ban ngày; có thể thêm sau |

---

## Implementation Estimate

| Task | Complexity |
|------|------------|
| Update globals.css | Simple - variable updates |
| Add phase colors | Simple - add new variables |
| Remove dark mode | Simple - delete code |
| Update docs | Simple - text changes |

---

## Success Criteria

- [ ] Light glass effect matches mock (70% opacity)
- [ ] Brand color is Blue-500
- [ ] Phase colors available: `bg-phase-order`, `bg-phase-design`, etc.
- [ ] No dark mode remnants
- [ ] Documentation updated

---

## Next Steps

1. Create implementation plan file
2. Update globals.css với new color values
3. Update synkao-design-system.md documentation
4. Visual testing với mock HTML comparison

---

## Unresolved Questions

None - all decisions confirmed by user.
