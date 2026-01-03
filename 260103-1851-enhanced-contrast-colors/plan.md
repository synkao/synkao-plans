---
title: "Enhanced Contrast Color Scheme"
description: "Tăng tương phản giữa các layers bằng lightness difference"
status: completed
priority: P2
effort: 30m
branch: main
tags: [ui, css, design-system, color]
created: 2026-01-03
completed: 2026-01-03
---

# Enhanced Contrast Color Scheme

## Overview

Cập nhật CSS variables để tăng tương phản giữa các layers (background, card, sidebar, borders).

**Problem:** Lightness quá gần nhau (0.97-1.0), các sections khó phân biệt.

**Solution:** Giảm lightness theo layers, thêm subtle blue tint cho sidebar.

## References

- [Brainstorm Report](reports/brainstorm-260103-1851-enhanced-contrast-colors.md)
- [globals.css](../../apps/web/src/app/globals.css)

## Implementation Phases

| Phase | Description | Status | Progress |
|-------|-------------|--------|----------|
| [Phase 01](phase-01-update-css-variables.md) | Update CSS variables | completed | 100% |

## Lightness Stack Target

```
Card            1.00  ████████████████████
Background      0.96  ██████████████████
Muted/Secondary 0.94  █████████████████
Sidebar         0.92  ████████████████
Border          0.88  ██████████████
Sidebar-border  0.85  █████████████
```

## Success Criteria

- [ ] Layers visually distinguishable
- [ ] Sidebar separated from content
- [ ] Borders visible but not harsh
- [ ] Text contrast accessible (WCAG AA)
