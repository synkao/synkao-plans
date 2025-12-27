# Phase 03: Test Base Components

## Context Links
- [Parent Plan](./plan.md)
- [Phase 02](./phase-02-configure-glass-theme.md)
- [Design System](../../docs/synkao-design-system.md)

## Overview
- **Priority:** P0
- **Status:** Pending
- **Effort:** 45 min
- **Description:** Add Button, Card, Input components và test trên trang demo

## Key Insights

### Components cần test
1. **Button** - Core interaction element
2. **Card** - Glass effect showcase
3. **Input** - Form element với variants

### shadcn CLI Commands
```bash
pnpm dlx shadcn@latest add button
pnpm dlx shadcn@latest add card
pnpm dlx shadcn@latest add input
```

## Requirements

### Functional
- Button với variants: default, destructive, outline, secondary, ghost, link
- Card với glass effect
- Input với states: default, error, disabled

### Non-functional
- Components render correctly
- Glass effect visible
- Hover/focus states work

## Architecture

```
apps/web/src/
├── components/
│   └── ui/
│       ├── button.tsx    # shadcn button
│       ├── card.tsx      # shadcn card
│       └── input.tsx     # shadcn input
└── app/
    └── page.tsx          # Demo page
```

## Related Code Files

### Files to Create
| File | Action | Description |
|------|--------|-------------|
| `apps/web/src/components/ui/button.tsx` | Create | shadcn Button |
| `apps/web/src/components/ui/card.tsx` | Create | shadcn Card |
| `apps/web/src/components/ui/input.tsx` | Create | shadcn Input |

### Files to Modify
| File | Action | Description |
|------|--------|-------------|
| `apps/web/src/app/page.tsx` | Modify | Add demo components |

## Implementation Steps

### Step 1: Add shadcn Components
```bash
cd apps/web
pnpm dlx shadcn@latest add button card input
```

### Step 2: Customize Card for Glass Effect

Update `apps/web/src/components/ui/card.tsx`:

```tsx
// Add glass variant to Card
const Card = React.forwardRef<
  HTMLDivElement,
  React.HTMLAttributes<HTMLDivElement> & { variant?: "default" | "glass" }
>(({ className, variant = "default", ...props }, ref) => (
  <div
    ref={ref}
    className={cn(
      "rounded-xl border shadow",
      variant === "glass"
        ? "glass" // Uses utility class from globals.css
        : "bg-card text-card-foreground",
      className
    )}
    {...props}
  />
))
```

### Step 3: Create Demo Page

Update `apps/web/src/app/page.tsx`:

```tsx
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";

export default function Home() {
  return (
    <main className="min-h-screen bg-gradient-to-br from-slate-900 via-purple-900 to-slate-900 p-8">
      <div className="max-w-4xl mx-auto space-y-8">
        <h1 className="text-3xl font-bold text-white">
          Synkao Design System Demo
        </h1>

        {/* Button Variants */}
        <section className="space-y-4">
          <h2 className="text-xl font-semibold text-white">Buttons</h2>
          <div className="flex flex-wrap gap-4">
            <Button>Default</Button>
            <Button variant="secondary">Secondary</Button>
            <Button variant="destructive">Destructive</Button>
            <Button variant="outline">Outline</Button>
            <Button variant="ghost">Ghost</Button>
            <Button variant="link">Link</Button>
          </div>
        </section>

        {/* Glass Card */}
        <section className="space-y-4">
          <h2 className="text-xl font-semibold text-white">Glass Card</h2>
          <Card className="glass">
            <CardHeader>
              <CardTitle className="text-white">Glass Card</CardTitle>
            </CardHeader>
            <CardContent className="text-white/80">
              This card demonstrates the glass morphism effect.
            </CardContent>
          </Card>
        </section>

        {/* Input Variants */}
        <section className="space-y-4">
          <h2 className="text-xl font-semibold text-white">Inputs</h2>
          <div className="space-y-2 max-w-sm">
            <Input placeholder="Default input" />
            <Input placeholder="Disabled input" disabled />
          </div>
        </section>

        {/* Status Colors */}
        <section className="space-y-4">
          <h2 className="text-xl font-semibold text-white">Status Colors</h2>
          <div className="flex flex-wrap gap-2">
            <span className="px-3 py-1 rounded-full text-sm bg-[hsl(var(--status-todo))]/20 text-[hsl(var(--status-todo))] border border-[hsl(var(--status-todo))]/30">
              TODO
            </span>
            <span className="px-3 py-1 rounded-full text-sm bg-[hsl(var(--status-in-progress))]/20 text-[hsl(var(--status-in-progress))] border border-[hsl(var(--status-in-progress))]/30">
              IN PROGRESS
            </span>
            <span className="px-3 py-1 rounded-full text-sm bg-[hsl(var(--status-review))]/20 text-[hsl(var(--status-review))] border border-[hsl(var(--status-review))]/30">
              REVIEW
            </span>
            <span className="px-3 py-1 rounded-full text-sm bg-[hsl(var(--status-revision))]/20 text-[hsl(var(--status-revision))] border border-[hsl(var(--status-revision))]/30">
              REVISION
            </span>
            <span className="px-3 py-1 rounded-full text-sm bg-[hsl(var(--status-done))]/20 text-[hsl(var(--status-done))] border border-[hsl(var(--status-done))]/30">
              DONE
            </span>
          </div>
        </section>

        {/* Priority Colors */}
        <section className="space-y-4">
          <h2 className="text-xl font-semibold text-white">Priority Colors</h2>
          <div className="flex flex-wrap gap-2">
            <span className="px-3 py-1 rounded-full text-sm bg-[hsl(var(--priority-low))]/20 text-[hsl(var(--priority-low))]">
              LOW
            </span>
            <span className="px-3 py-1 rounded-full text-sm bg-[hsl(var(--priority-normal))]/20 text-[hsl(var(--priority-normal))]">
              NORMAL
            </span>
            <span className="px-3 py-1 rounded-full text-sm bg-[hsl(var(--priority-high))]/20 text-[hsl(var(--priority-high))]">
              HIGH
            </span>
            <span className="px-3 py-1 rounded-full text-sm bg-[hsl(var(--priority-urgent))]/20 text-[hsl(var(--priority-urgent))]">
              URGENT
            </span>
          </div>
        </section>
      </div>
    </main>
  );
}
```

### Step 4: Run Dev Server and Verify
```bash
cd apps/web
pnpm dev
```

Open http://localhost:3000 and verify:
- All buttons render with correct variants
- Glass card shows blur effect
- Inputs work correctly
- Status/Priority badges show correct colors

## Todo List

- [ ] Add Button component via shadcn CLI
- [ ] Add Card component via shadcn CLI
- [ ] Add Input component via shadcn CLI
- [ ] Update page.tsx với demo content
- [ ] Run dev server và verify visually
- [ ] Run `pnpm build` to check for errors

## Success Criteria

- [ ] Button variants render correctly
- [ ] Card với glass effect visible
- [ ] Input states work (default, disabled)
- [ ] Status/Priority colors match design system
- [ ] `pnpm build` passes without errors
- [ ] No console errors in browser

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Component styling conflicts | Medium | Check shadcn output |
| CSS variable resolution | Medium | Test in browser devtools |

## Security Considerations
- N/A (UI components only)

## Next Steps
- Issue #2 completed
- Ready for Issue #3: Setup Mock Service Worker (MSW)
