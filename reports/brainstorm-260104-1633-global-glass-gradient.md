# Brainstorm: Global Glass Gradient Background

**Date:** 2026-01-04
**Style:** Glassmorphism + Soft Gradient

---

## Problem Statement

User wants to apply the blue-indigo-violet gradient background (seen in GlassCard demo) across the entire project for visual consistency.

## Current State

| Issue | Detail |
|-------|--------|
| Inconsistent gradients | Each page uses different gradient colors |
| No global standard | Gradients defined inline per page |
| Missing design token | No CSS variable for app gradient |

**Current gradient variations:**
- Auth: `from-slate-50 via-white to-blue-50`
- API test: `from-slate-100 via-slate-50 to-white`
- GlassCard demo: `from-blue-100 via-indigo-50 to-violet-50` ← Target
- Form section: `from-emerald-100 via-teal-50 to-cyan-50`
- Data table: `from-amber-100 via-orange-50 to-rose-50`

---

## Style Analysis

**Target Style:** Glassmorphism + Soft Gradient Background

| Property | Value | Purpose |
|----------|-------|---------|
| Gradient | `from-blue-100 via-indigo-50 to-violet-50` | Soft, professional, calm |
| Direction | `to-br` (135deg) | Modern diagonal flow |
| Glass cards | `bg-white/70 backdrop-blur-[10px]` | Frosted glass effect |
| Attachment | `fixed` | Stays stable on scroll |

---

## Evaluated Approaches

### Option A: CSS Variable + Utility Class ✅ Selected

Add to `globals.css`:
```css
:root {
  --app-gradient: linear-gradient(135deg,
    oklch(0.93 0.02 260) 0%,
    oklch(0.97 0.015 270) 50%,
    oklch(0.97 0.02 285) 100%
  );
}

.bg-app-gradient {
  background: var(--app-gradient);
  background-attachment: fixed;
}
```

Apply in `layout.tsx`:
```tsx
<body className="... bg-app-gradient">
```

**Pros:** DRY, maintainable, themeable, follows existing pattern
**Cons:** Adds CSS variable (minimal)

### Option B: Tailwind Direct Class ❌ Rejected

**Reason:** Hardcoded, difficult to maintain across changes

### Option C: Fixed Background Div ❌ Rejected

**Reason:** Adds unnecessary DOM element

---

## Final Decision: Scope A - Full Viewport

User đã chọn sau khi xem demo.

### Scope A: Full Viewport (Root Body)

```
┌──────────────────────────────────┐
│ ░░░░░░░ GRADIENT EVERYWHERE ░░░░ │
│ ┌────────┬───────────────────────┤
│ │ Sidebar│     Main Content      │
│ │ (glass)│                       │
│ │        │                       │
│ └────────┴───────────────────────┤
└──────────────────────────────────┘
```

Apply: `apps/web/src/app/layout.tsx` body
**Pros:** Unified look, sidebar glass looks premium
**Cons:** May be too much color

### Scope B: Content Area Only

```
┌──────────────────────────────────┐
│ ┌────────┬───────────────────────┤
│ │ Sidebar│░░░ GRADIENT HERE ░░░░│
│ │ (solid)│                       │
│ │        │                       │
│ └────────┴───────────────────────┤
└──────────────────────────────────┘
```

Apply: `apps/web/src/app/(main)/layout.tsx` main element
**Pros:** Cleaner sidebar, focused gradient
**Cons:** Less cohesive

---

## Implementation ✅ Completed

| Step | Status |
|------|--------|
| Add `--app-gradient` variable to `:root` | ✅ |
| Add `.bg-app-gradient` utility class | ✅ |
| Apply to root body in `layout.tsx` | ✅ |
| Remove inline gradients (auth, api-test) | ✅ |
| Type check passed | ✅ |
| Demo pages removed | ✅ |

### Files Changed:
- [globals.css](apps/web/src/app/globals.css) - Added CSS variable + utility
- [layout.tsx](apps/web/src/app/layout.tsx) - Applied `bg-app-gradient`
- [(auth)/layout.tsx](apps/web/src/app/(auth)/layout.tsx) - Removed inline gradient
- [api-test/page.tsx](apps/web/src/app/api-test/page.tsx) - Removed inline gradient

---

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Glass sidebar conflict | Sidebar already uses own glass bg, should overlay fine |
| OKLCH browser support | 95%+ modern browsers, acceptable |
| Fixed attachment mobile | Some mobile browsers ignore, fallback to scroll is acceptable |

---

## Success Criteria

- [ ] Gradient visible across all pages
- [ ] Gradient stays fixed on scroll
- [ ] GlassCard components render correctly on gradient
- [ ] Sidebar glass effect works without conflict
- [ ] No visual regression in existing pages

---

## References

- [GlassCard Component](apps/web/src/components/ui/glass-card.tsx)
- [Current globals.css](apps/web/src/app/globals.css)
- [Previous glass theme brainstorm](plans/reports/brainstorm-251224-1422-shadcn-glass-theme.md)
