---
title: "Fix Global Glass Gradient Background"
description: "Debug và fix gradient không hiển thị, document design system"
status: pending
priority: P1
effort: 1h
branch: main
tags: [ui, css, gradient, glassmorphism, documentation]
created: 2026-01-04
---

# Fix Global Glass Gradient Background

## Overview

Global glass gradient đã implement nhưng không hiển thị. Cần debug, fix và document.

## Root Cause Analysis

**Issue:** CSS specificity conflict trong Tailwind CSS v4

```css
/* globals.css */
@layer base {
  body {
    @apply bg-background text-foreground;  /* ← Override gradient */
  }
}

@layer utilities {
  .bg-app-gradient { ... }  /* ← Bị override */
}
```

**Why:** Mặc dù utilities > base theo CSS Cascade Layers, nhưng `@apply` trong base layer set trực tiếp `background` property, gây conflict.

## Solution

Remove `bg-background` từ base layer, để class trong layout.tsx control background.

## Phases

| # | Phase | Status | Est |
|---|-------|--------|-----|
| 1 | [Debug & Fix Gradient](phase-01-debug-fix-gradient.md) | pending | 30m |
| 2 | [Documentation](phase-02-documentation.md) | pending | 30m |

## Files to Change

- `apps/web/src/app/globals.css` - Remove conflicting rule
- `docs/design-guidelines.md` - Document gradient system

## Success Criteria

- [ ] Gradient visible across entire app
- [ ] Fixed background on scroll
- [ ] GlassCard components work on gradient
- [ ] Documentation updated

## Related

- [Brainstorm Report](../reports/brainstorm-260104-1633-global-glass-gradient.md)
- [GlassCard Component](../../apps/web/src/components/ui/glass-card.tsx)
