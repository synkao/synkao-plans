# Phase 01: Update CSS Variables

## Context

- **Parent Plan:** [plan.md](plan.md)
- **Reference:** [Brainstorm Report](reports/brainstorm-260103-1851-enhanced-contrast-colors.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-03 |
| Priority | P2 |
| Implementation | completed |
| Review | completed |
| Completed | 2026-01-03 |

## Key Insights

- Current lightness range: 0.97-1.0 (too narrow)
- Target lightness range: 0.85-1.0 (better separation)
- Sidebar gets subtle blue tint (chroma: 0.012-0.015)

## Requirements

1. Update background layers với lower lightness
2. Add blue tint cho sidebar
3. Strengthen borders
4. Maintain text accessibility

## Related Files

- `apps/web/src/app/globals.css` - Target file

## Implementation Steps

### Step 1: Background Layers

Update trong `:root` section:

```css
--background: oklch(0.96 0.003 247);
--muted: oklch(0.94 0.003 247);
--secondary: oklch(0.94 0.003 247);
--accent: oklch(0.92 0.003 247);
```

### Step 2: Border Variables

```css
--border: oklch(0.88 0.005 247);
--input: oklch(0.88 0.005 247);
```

### Step 3: Sidebar (Blue Tint)

```css
--sidebar: oklch(0.92 0.012 247);
--sidebar-accent: oklch(0.88 0.015 247);
--sidebar-border: oklch(0.85 0.010 247);
```

### Step 4: Verify Foreground Colors

Check `--muted-foreground`, `--accent-foreground`, `--secondary-foreground` contrast với new backgrounds.

## Todo List

- [ ] Update background layers (4 vars)
- [ ] Update border variables (2 vars)
- [ ] Update sidebar variables (3 vars)
- [ ] Verify foreground contrast
- [ ] Visual test

## Success Criteria

- [ ] All 9 variables updated correctly
- [ ] No CSS syntax errors
- [ ] Layers visually distinguishable
- [ ] Text readable on all backgrounds

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Text contrast issues | Medium | Check foreground colors |
| Glass effect visibility | Low | Adjust opacity if needed |

## Security Considerations

None - CSS only changes.

## Next Steps

1. Implement changes
2. Visual test all pages
3. Adjust if needed
