# Phase 02: Configure Glass Theme

## Context Links
- [Parent Plan](./plan.md)
- [Phase 01](./phase-01-initialize-shadcn.md)
- [Design System](../../docs/synkao-design-system.md)

## Overview
- **Priority:** P0
- **Status:** Pending
- **Effort:** 1.5h
- **Description:** Cấu hình CSS variables và theme colors theo Glass Morphism design system

## Key Insights

### Glass Morphism Characteristics
- Semi-transparent backgrounds (`rgba(255, 255, 255, 0.1)`)
- Backdrop blur (`blur(10px)`)
- Subtle borders (`rgba(255, 255, 255, 0.2)`)
- Soft shadows
- Works best on dark backgrounds

### Design System Colors
From `docs/synkao-design-system.md`:

**Primary:**
- `--primary: 222.2 47.4% 11.2%` (Dark blue)

**Status Colors:**
- TODO: `221 83% 53%` (Blue)
- IN_PROGRESS: `38 92% 50%` (Amber)
- REVIEW: `271 91% 65%` (Violet)
- REVISION: `0 84% 60%` (Red)
- DONE: `142 76% 36%` (Green)

**Priority Colors:**
- LOW: Gray
- NORMAL: Blue
- HIGH: Amber
- URGENT: Red

## Requirements

### Functional
- All shadcn CSS variables defined
- Glass effect CSS variables
- Status/Priority color utilities
- Dark mode support (primary mode)
- Inter + JetBrains Mono fonts

### Non-functional
- Performance: No layout shift from fonts
- Accessibility: Sufficient contrast ratios

## Architecture

### CSS Variable Structure
```css
:root {
  /* shadcn base */
  --background, --foreground, --card, --primary, --secondary...

  /* Glass effect */
  --glass-bg, --glass-border, --glass-blur, --glass-shadow...

  /* Status colors */
  --status-todo, --status-in-progress, --status-review...

  /* Priority colors */
  --priority-low, --priority-normal, --priority-high, --priority-urgent
}
```

## Related Code Files

### Files to Modify
| File | Action | Description |
|------|--------|-------------|
| `apps/web/src/app/globals.css` | Modify | Add all CSS variables |
| `apps/web/src/app/layout.tsx` | Modify | Add JetBrains Mono font |

## Implementation Steps

### Step 1: Update globals.css với CSS Variables

```css
@import "tailwindcss";

:root {
  /* Base Colors */
  --background: 222.2 84% 4.9%;
  --foreground: 210 40% 98%;

  /* Card */
  --card: 222.2 84% 4.9%;
  --card-foreground: 210 40% 98%;

  /* Popover */
  --popover: 222.2 84% 4.9%;
  --popover-foreground: 210 40% 98%;

  /* Primary */
  --primary: 222.2 47.4% 11.2%;
  --primary-foreground: 210 40% 98%;

  /* Secondary */
  --secondary: 217.2 32.6% 17.5%;
  --secondary-foreground: 210 40% 98%;

  /* Muted */
  --muted: 217.2 32.6% 17.5%;
  --muted-foreground: 215 20.2% 65.1%;

  /* Accent */
  --accent: 217.2 32.6% 17.5%;
  --accent-foreground: 210 40% 98%;

  /* Destructive */
  --destructive: 0 62.8% 30.6%;
  --destructive-foreground: 210 40% 98%;

  /* Border/Input/Ring */
  --border: 217.2 32.6% 17.5%;
  --input: 217.2 32.6% 17.5%;
  --ring: 212.7 26.8% 83.9%;

  /* Radius */
  --radius: 0.5rem;

  /* ===== GLASS EFFECT ===== */
  --glass-bg: 0 0% 100% / 0.1;
  --glass-bg-hover: 0 0% 100% / 0.15;
  --glass-border: 0 0% 100% / 0.2;
  --glass-blur: 10px;
  --glass-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37);

  /* Glass Dark (sidebar) */
  --glass-dark-bg: 0 0% 0% / 0.3;
  --glass-dark-border: 0 0% 100% / 0.1;

  /* ===== STATUS COLORS ===== */
  --status-todo: 221 83% 53%;
  --status-in-progress: 38 92% 50%;
  --status-review: 271 91% 65%;
  --status-revision: 0 84% 60%;
  --status-done: 142 76% 36%;

  /* ===== PRIORITY COLORS ===== */
  --priority-low: 220 9% 46%;
  --priority-normal: 221 83% 53%;
  --priority-high: 38 92% 50%;
  --priority-urgent: 0 84% 60%;

  /* ===== SEMANTIC COLORS ===== */
  --success: 142 76% 36%;
  --warning: 38 92% 50%;
  --error: 0 84% 60%;
  --info: 221 83% 53%;

  /* Chart Colors (optional) */
  --chart-1: 220 70% 50%;
  --chart-2: 160 60% 45%;
  --chart-3: 30 80% 55%;
  --chart-4: 280 65% 60%;
  --chart-5: 340 75% 55%;
}

/* Light mode override (if needed) */
@media (prefers-color-scheme: light) {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    /* ... other light overrides */
  }
}

@theme inline {
  --color-background: hsl(var(--background));
  --color-foreground: hsl(var(--foreground));
  --color-card: hsl(var(--card));
  --color-card-foreground: hsl(var(--card-foreground));
  --color-popover: hsl(var(--popover));
  --color-popover-foreground: hsl(var(--popover-foreground));
  --color-primary: hsl(var(--primary));
  --color-primary-foreground: hsl(var(--primary-foreground));
  --color-secondary: hsl(var(--secondary));
  --color-secondary-foreground: hsl(var(--secondary-foreground));
  --color-muted: hsl(var(--muted));
  --color-muted-foreground: hsl(var(--muted-foreground));
  --color-accent: hsl(var(--accent));
  --color-accent-foreground: hsl(var(--accent-foreground));
  --color-destructive: hsl(var(--destructive));
  --color-destructive-foreground: hsl(var(--destructive-foreground));
  --color-border: hsl(var(--border));
  --color-input: hsl(var(--input));
  --color-ring: hsl(var(--ring));

  /* Glass */
  --color-glass-bg: hsl(var(--glass-bg));
  --color-glass-border: hsl(var(--glass-border));

  /* Status */
  --color-status-todo: hsl(var(--status-todo));
  --color-status-in-progress: hsl(var(--status-in-progress));
  --color-status-review: hsl(var(--status-review));
  --color-status-revision: hsl(var(--status-revision));
  --color-status-done: hsl(var(--status-done));

  /* Priority */
  --color-priority-low: hsl(var(--priority-low));
  --color-priority-normal: hsl(var(--priority-normal));
  --color-priority-high: hsl(var(--priority-high));
  --color-priority-urgent: hsl(var(--priority-urgent));

  /* Fonts */
  --font-sans: var(--font-inter);
  --font-mono: var(--font-jetbrains-mono);

  /* Radius */
  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: var(--radius);
  --radius-lg: calc(var(--radius) + 4px);
  --radius-xl: calc(var(--radius) + 8px);
}

body {
  background: var(--color-background);
  color: var(--color-foreground);
}
```

### Step 2: Add JetBrains Mono Font

Update `apps/web/src/app/layout.tsx`:

```tsx
import type { Metadata } from "next";
import { Inter, JetBrains_Mono } from "next/font/google";
import "./globals.css";

const inter = Inter({
  variable: "--font-inter",
  subsets: ["latin"],
});

const jetbrainsMono = JetBrains_Mono({
  variable: "--font-jetbrains-mono",
  subsets: ["latin"],
});

export const metadata: Metadata = {
  title: "Synkao - POD Order Management",
  description: "Design task management for Print-on-Demand",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="vi" className="dark">
      <body className={`${inter.variable} ${jetbrainsMono.variable} antialiased`}>
        {children}
      </body>
    </html>
  );
}
```

### Step 3: Create Glass Utility Classes (Optional)

Add to globals.css after @theme block:

```css
/* Glass utility classes */
.glass {
  background: hsl(var(--glass-bg));
  backdrop-filter: blur(var(--glass-blur));
  -webkit-backdrop-filter: blur(var(--glass-blur));
  border: 1px solid hsl(var(--glass-border));
  box-shadow: var(--glass-shadow);
}

.glass:hover {
  background: hsl(var(--glass-bg-hover));
}

.glass-dark {
  background: hsl(var(--glass-dark-bg));
  backdrop-filter: blur(var(--glass-blur));
  -webkit-backdrop-filter: blur(var(--glass-blur));
  border: 1px solid hsl(var(--glass-dark-border));
}
```

## Todo List

- [ ] Update globals.css với full CSS variables
- [ ] Add JetBrains Mono font to layout.tsx
- [ ] Add `className="dark"` to html element
- [ ] Add glass utility classes
- [ ] Verify variables work with `pnpm dev`

## Success Criteria

- [ ] All CSS variables defined and accessible
- [ ] Fonts load without layout shift
- [ ] Glass effect visible on test elements
- [ ] Dark mode active by default

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Tailwind v4 @theme compatibility | High | Test each variable |
| Backdrop-filter browser support | Low | Fallback to solid bg |
| Font loading performance | Medium | Use next/font |

## Security Considerations
- N/A

## Next Steps
- Proceed to Phase 03: Test Base Components
