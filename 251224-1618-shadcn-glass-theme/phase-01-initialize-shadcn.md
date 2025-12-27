# Phase 01: Initialize shadcn/ui

## Context Links
- [Parent Plan](./plan.md)
- [Design System](../../docs/synkao-design-system.md)
- [shadcn/ui Docs](https://ui.shadcn.com/docs/installation/next)

## Overview
- **Priority:** P0
- **Status:** Pending
- **Effort:** 45 min
- **Description:** Initialize shadcn/ui trong Next.js 16 project với Tailwind CSS 4

## Key Insights

### Tailwind CSS 4 Compatibility
- shadcn/ui đang transition sang Tailwind v4
- Cần dùng `@shadcn/ui@canary` hoặc `@latest` (đã support v4)
- Tailwind v4 dùng `@theme inline` thay vì tailwind.config.js

### Configuration Choices
- **Style:** New York (cleaner, modern look)
- **Base color:** Slate
- **CSS Variables:** Yes (required cho theming)
- **React Server Components:** Yes (Next.js 16 default)

## Requirements

### Functional
- shadcn CLI initialized
- components.json configured
- lib/utils.ts với cn() helper (đã có)
- Base CSS variables setup

### Non-functional
- Compatible với Tailwind CSS 4
- TypeScript strict mode

## Architecture

```
apps/web/
├── src/
│   ├── components/
│   │   └── ui/           # shadcn components sẽ nằm đây
│   ├── lib/
│   │   └── utils.ts      # cn() helper (đã có)
│   └── app/
│       └── globals.css   # CSS variables
├── components.json       # shadcn config
└── package.json
```

## Related Code Files

### Files to Modify
| File | Action | Description |
|------|--------|-------------|
| `apps/web/package.json` | Modify | Add dependencies |
| `apps/web/src/app/globals.css` | Modify | Update CSS variables |

### Files to Create
| File | Action | Description |
|------|--------|-------------|
| `apps/web/components.json` | Create | shadcn configuration |
| `apps/web/src/components/ui/` | Create | Components directory |

## Implementation Steps

### Step 1: Install Dependencies
```bash
cd apps/web
pnpm add class-variance-authority lucide-react
pnpm add -D @types/node
```

### Step 2: Initialize shadcn/ui
```bash
cd apps/web
pnpm dlx shadcn@latest init
```

**Interactive prompts:**
- Style: `new-york`
- Base color: `slate`
- CSS variables: `yes`
- Tailwind CSS config: Skip (Tailwind v4 uses CSS)
- Components alias: `@/components`
- Utilities alias: `@/lib/utils`
- React Server Components: `yes`

### Step 3: Verify components.json
Expected output:
```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "new-york",
  "rsc": true,
  "tsx": true,
  "tailwind": {
    "config": "",
    "css": "src/app/globals.css",
    "baseColor": "slate",
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils"
  }
}
```

### Step 4: Verify tsconfig.json Paths
Ensure paths configured:
```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

## Todo List

- [ ] Install class-variance-authority, lucide-react
- [ ] Run shadcn init với đúng options
- [ ] Verify components.json created
- [ ] Verify tsconfig paths
- [ ] Create components/ui/ directory

## Success Criteria

- [ ] `pnpm build` passes
- [ ] components.json exists với correct config
- [ ] No TypeScript errors
- [ ] lib/utils.ts exports cn()

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Tailwind v4 incompatibility | High | Use shadcn@latest, check docs |
| Path alias issues | Medium | Verify tsconfig.json |

## Security Considerations
- N/A (setup phase only)

## Next Steps
- Proceed to Phase 02: Configure Glass Theme
