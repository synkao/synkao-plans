# Phase 02: Light Glass + Phase Colors

## Context Links
- [Parent Plan](./plan.md)
- [Phase 01](./phase-01-initialize-shadcn.md)
- [Mock HTML](./synkao-design-system-demo.html)
- [Brainstorm Report](../reports/brainstorm-251225-1023-light-glass-design-system-update.md)

## Overview
- **Priority:** P0
- **Status:** Done (2025-12-25 11:00)
- **Effort:** 1h
- **Description:** Update CSS variables từ dark glass → Light Functional Glassmorphism + thêm Phase Colors

## Key Insights

### Light Functional Glassmorphism
- Higher opacity (70%) for better readability
- Light backgrounds (#F8FAFC Slate-50)
- Subtle blur (10px) - giữ nguyên
- High contrast text (dark on light)
- Optimized for daytime office use

### Color Format: OKLCH
Project sử dụng OKLCH color format. All HEX values need conversion:
- `#3B82F6` → `oklch(0.623 0.214 259)` (Blue-500)
- `#F8FAFC` → `oklch(0.984 0.003 247)` (Slate-50)

## Requirements

### Functional
- Update glass effect: 70% opacity, 85% hover
- Change brand color: Black → Blue-500
- Add 5 Phase Colors (Order/Design/Fulfill/Complete/Cancelled)
- Remove dark mode entirely

### Non-functional
- Maintain WCAG contrast ratios
- No breaking changes to existing utilities

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `apps/web/src/app/globals.css` | Modify | Update CSS variables |

## Implementation Steps

### Step 1: Update Background Colors

```css
:root {
  /* Background: White → Slate-50 */
  --background: oklch(0.984 0.003 247);
  --foreground: oklch(0.145 0 0);

  /* Card stays white */
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.145 0 0);
}
```

### Step 2: Update Brand/Primary Color

```css
:root {
  /* Primary: Black → Blue-500 */
  --primary: oklch(0.623 0.214 259);
  --primary-foreground: oklch(1 0 0);
}
```

### Step 3: Update Glass Effect

```css
:root {
  /* Glass: 60% → 70%, hover 85% */
  --glass-bg: oklch(1 0 0 / 70%);
  --glass-bg-hover: oklch(1 0 0 / 85%);
  --glass-border: oklch(0 0 0 / 10%);
  --glass-blur: 10px;
  --glass-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.1);
}
```

### Step 4: Add Phase Colors (NEW)

```css
:root {
  /* ===== PHASE COLORS ===== */
  --phase-order: oklch(0.554 0.046 257);     /* #64748B Slate-500 */
  --phase-design: oklch(0.769 0.188 70);     /* #F59E0B Amber-500 */
  --phase-fulfill: oklch(0.586 0.220 292);   /* #8B5CF6 Violet-500 */
  --phase-complete: oklch(0.627 0.194 149);  /* #10B981 Emerald-500 */
  --phase-cancelled: oklch(0.505 0.017 257); /* #6B7280 Gray-500 */
}

/* Add to @theme inline block */
@theme inline {
  --color-phase-order: var(--phase-order);
  --color-phase-design: var(--phase-design);
  --color-phase-fulfill: var(--phase-fulfill);
  --color-phase-complete: var(--phase-complete);
  --color-phase-cancelled: var(--phase-cancelled);
}
```

### Step 5: Remove Dark Mode

Delete entire `.dark { ... }` block and `@custom-variant dark`.

### Step 6: Update Glass Utility

```css
@layer utilities {
  .glass {
    background: var(--glass-bg);
    backdrop-filter: blur(var(--glass-blur));
    -webkit-backdrop-filter: blur(var(--glass-blur));
    border: 1px solid var(--glass-border);
    box-shadow: var(--glass-shadow);
  }

  .glass:hover {
    background: var(--glass-bg-hover);
    border-color: oklch(0.623 0.214 259 / 30%); /* Brand tint on hover */
  }

  /* Remove .glass-dark class */
}
```

### Step 7: Update layout.tsx

Remove `className="dark"` from `<html>` element.

## Todo List

- [x] Update background: Slate-50
- [x] Update primary: Blue-500
- [x] Update glass: 70%/85% opacity
- [x] Add phase colors (5 variables)
- [x] Add phase colors to @theme inline
- [x] Delete .dark block
- [x] Delete @custom-variant dark
- [x] Delete .glass-dark utility
- [x] Remove dark class from html element
- [x] Verify with `pnpm build`

## Success Criteria

- [x] Background is light (#F8FAFC)
- [x] Brand color is blue (#3B82F6)
- [x] Glass effect is 70% opacity
- [x] Phase colors work: `bg-phase-order`, `text-phase-design`, etc.
- [x] No dark mode code remains
- [x] Build passes

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| OKLCH conversion errors | Medium | Test each color visually |
| Existing code depends on .dark | Low | No components use dark mode yet |
| Tailwind class name conflicts | Low | Phase colors use unique prefix |

## Security Considerations
- N/A (CSS only)

## Next Steps
- Proceed to Phase 03: Update Documentation
