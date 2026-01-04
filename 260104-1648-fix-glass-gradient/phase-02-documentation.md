# Phase 02: Documentation

## Context

- **Parent Plan:** [plan.md](plan.md)
- **Dependencies:** [Phase 01](phase-01-debug-fix-gradient.md)
- **Docs:** [Design Guidelines](../../docs/design-guidelines.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P2 |
| Status | pending |
| Estimated | 30m |

## Requirements

- [ ] Create/update `docs/design-guidelines.md`
- [ ] Document gradient system
- [ ] Document glass effect utilities
- [ ] Add usage examples

## Content to Document

### 1. App Gradient Background

```md
## App Gradient Background

Global gradient background sử dụng glassmorphism style.

### CSS Variable
```css
--app-gradient: linear-gradient(
  135deg,
  oklch(0.93 0.025 260) 0%,   /* blue-100 */
  oklch(0.97 0.015 275) 50%,  /* indigo-50 */
  oklch(0.96 0.02 290) 100%   /* violet-50 */
);
```

### Utility Class
```css
.bg-app-gradient {
  background: var(--app-gradient);
  background-attachment: fixed;
}
```

### Usage
Applied globally in root layout. Gradient fixed on scroll.
```

### 2. Glass Effect

```md
## Glass Effect (Glassmorphism)

### CSS Variables
```css
--glass-bg: oklch(1 0 0 / 70%);
--glass-bg-hover: oklch(1 0 0 / 85%);
--glass-border: oklch(0 0 0 / 10%);
--glass-blur: 10px;
--glass-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.1);
```

### Utility Class `.glass`
- Semi-transparent white background (70%)
- Backdrop blur 10px
- Subtle border
- Soft shadow

### GlassCard Component
Reusable component với variants: default, hover, selected, clickable.
```

## Related Code Files

| File | Purpose |
|------|---------|
| `docs/design-guidelines.md` | Target documentation file |
| `apps/web/src/app/globals.css` | Source of CSS variables |
| `apps/web/src/components/ui/glass-card.tsx` | Component reference |

## Implementation Steps

### Step 1: Check if file exists

```bash
ls -la docs/design-guidelines.md
```

### Step 2: Create/Update documentation

Add sections:
1. App Gradient Background
2. Glass Effect (Glassmorphism)
3. GlassCard Component usage

### Step 3: Verify links work

Ensure relative links to source files are correct.

## Todo List

- [ ] Check existing docs structure
- [ ] Create/update design-guidelines.md
- [ ] Add gradient documentation
- [ ] Add glass effect documentation
- [ ] Add GlassCard component usage

## Success Criteria

- [ ] Documentation file exists and is complete
- [ ] Code examples accurate
- [ ] Links to source files valid

## Risk Assessment

| Risk | Level | Mitigation |
|------|-------|------------|
| Doc outdated | Low | Link to source code for reference |

## Security Considerations

None - documentation only.

## Next Steps

After documentation → Plan complete, commit changes.
