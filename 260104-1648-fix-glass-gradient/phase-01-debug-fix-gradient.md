# Phase 01: Debug & Fix Gradient

## Context

- **Parent Plan:** [plan.md](plan.md)
- **Dependencies:** None
- **Docs:** [Brainstorm Report](../reports/brainstorm-260104-1633-global-glass-gradient.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P1 |
| Status | completed |
| Estimated | 30m |

## Problem

Gradient không hiển thị vì CSS specificity conflict.

## Key Insights

1. **Tailwind v4 CSS Layers:**
   - `@layer base` < `@layer components` < `@layer utilities`
   - Utilities should win, nhưng `@apply` trong base set trực tiếp property

2. **Current Code:**
   ```css
   @layer base {
     body {
       @apply bg-background text-foreground;
     }
   }
   ```
   - `bg-background` = `background: var(--color-background)`
   - Conflict với `bg-app-gradient` trong layout.tsx

3. **Root Body:**
   ```tsx
   <body className="... bg-app-gradient">
   ```

## Requirements

- [x] Remove `bg-background` from base layer body rule
- [x] Verify gradient displays correctly
- [ ] Test scroll behavior (fixed background)
- [ ] Test glass components overlay

## Architecture

```
globals.css
├── @layer base
│   └── body { text-foreground only }  ← CHANGE
└── @layer utilities
    └── .bg-app-gradient { ... }       ← No change

layout.tsx
└── <body className="bg-app-gradient"> ← No change
```

## Related Code Files

| File | Purpose |
|------|---------|
| `apps/web/src/app/globals.css` | CSS variables, utilities |
| `apps/web/src/app/layout.tsx` | Root layout with gradient class |
| `apps/web/src/components/ui/glass-card.tsx` | Glass effect component |

## Implementation Steps

### Step 1: Fix CSS Conflict

Edit `globals.css` line 160-162:

**Before:**
```css
body {
  @apply bg-background text-foreground;
}
```

**After:**
```css
body {
  @apply text-foreground;
}
```

### Step 2: Verify in Browser

1. Run `pnpm dev`
2. Check http://localhost:3000
3. Verify gradient visible
4. Test scroll (gradient should stay fixed)

### Step 3: Test Components

1. Check `/design` page - GlassCard overlay
2. Check `/login` page - Auth glass card
3. Check sidebar glass effect

## Todo List

- [x] Edit globals.css - remove bg-background
- [x] Type check (build passes)

## Success Criteria

- [ ] Gradient visible on page load
- [ ] Gradient fixed during scroll
- [ ] Glass cards render correctly on gradient
- [ ] No TypeScript errors
- [ ] No console errors

## Risk Assessment

| Risk | Level | Mitigation |
|------|-------|------------|
| Other pages affected | Low | bg-background removed, gradient becomes default |
| Dark mode broken | Low | Dark mode not implemented yet |

## Security Considerations

None - CSS only change.

## Next Steps

After fix verified → Phase 02: Documentation
