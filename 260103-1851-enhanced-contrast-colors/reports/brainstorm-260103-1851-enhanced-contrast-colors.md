# Brainstorm Report: Enhanced Contrast Color Scheme

**Date:** 2026-01-03
**Objective:** Tăng tương phản giữa các layers để phân biệt rõ ràng hơn

---

## Problem Statement

Color palette hiện tại có lightness quá gần nhau (0.97-1.0), làm các sections khó phân biệt:
- Card, Background, Muted, Secondary đều gần trắng
- Sidebar gần như blend vào background
- Border quá nhạt

---

## Current vs Proposed Values

### Background Layers

| Variable | Current | Proposed | Change |
|----------|---------|----------|--------|
| `--background` | `oklch(0.984 0.003 247)` | `oklch(0.96 0.003 247)` | L: -0.024 |
| `--card` | `oklch(1 0 0)` | `oklch(1 0 0)` | No change |
| `--muted` | `oklch(0.97 0 0)` | `oklch(0.94 0.003 247)` | L: -0.03 |
| `--secondary` | `oklch(0.97 0 0)` | `oklch(0.94 0.003 247)` | L: -0.03 |
| `--accent` | `oklch(0.97 0 0)` | `oklch(0.92 0.003 247)` | L: -0.05 |

### Sidebar (with Blue Tint)

| Variable | Current | Proposed | Change |
|----------|---------|----------|--------|
| `--sidebar` | `oklch(0.985 0 0)` | `oklch(0.92 0.012 247)` | L: -0.065, added tint |
| `--sidebar-accent` | `oklch(0.97 0 0)` | `oklch(0.88 0.015 247)` | L: -0.09 |
| `--sidebar-border` | `oklch(0.922 0 0)` | `oklch(0.85 0.010 247)` | L: -0.072 |

### Borders

| Variable | Current | Proposed | Change |
|----------|---------|----------|--------|
| `--border` | `oklch(0.922 0 0)` | `oklch(0.88 0.005 247)` | L: -0.042 |
| `--input` | `oklch(0.922 0 0)` | `oklch(0.88 0.005 247)` | L: -0.042 |

---

## Lightness Stack (Visual)

```
Layer           Lightness   Visual
──────────────────────────────────────
Card            1.00        ████████████████████
Background      0.96        ██████████████████
Muted/Secondary 0.94        █████████████████
Sidebar         0.92        ████████████████
Accent          0.92        ████████████████
Border          0.88        ██████████████
Sidebar-accent  0.88        ██████████████
Sidebar-border  0.85        █████████████
```

---

## Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Contrast method | Lightness difference | Most effective, universal |
| Sidebar style | Tối hơn background + blue tint | Professional look, clear separation |
| Card style | Pure white (L=1.0) | Nổi bật nhất, content focus |
| Border strength | L=0.88 (was 0.922) | Visible but not harsh |

---

## Implementation Scope

### Files to modify
- [globals.css](apps/web/src/app/globals.css) - Update CSS variables

### Variables to update (12 total)
1. `--background`
2. `--muted`
3. `--secondary`
4. `--accent`
5. `--border`
6. `--input`
7. `--sidebar`
8. `--sidebar-accent`
9. `--sidebar-border`
10. `--sidebar-ring`
11. `--accent-foreground` (adjust for new accent)
12. `--secondary-foreground` (adjust for new secondary)

---

## Risks & Considerations

| Risk | Mitigation |
|------|------------|
| Text contrast trên darker backgrounds | Verify foreground colors still accessible |
| Glass effect visibility | May need to adjust `--glass-bg` opacity |
| Hover states | Verify hover colors still provide feedback |

---

## Success Criteria

- [ ] All layers visually distinguishable
- [ ] Sidebar clearly separated from content
- [ ] Card "pops" from background
- [ ] Nested sections (muted areas) have subtle differentiation
- [ ] Borders visible without being harsh
- [ ] WCAG AA contrast ratios maintained for text

---

## Next Steps

1. Update CSS variables in globals.css
2. Visual test all pages/components
3. Verify accessibility (contrast ratios)
4. Adjust glass effect if needed
