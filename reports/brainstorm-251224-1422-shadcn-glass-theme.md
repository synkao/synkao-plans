# Brainstorm Report: shadcn/ui + Glass Theme

**Issue:** [#2 - Cấu hình shadcn/ui + custom theme (Glass)](https://github.com/synkao/synkao/issues/2)
**Date:** 2024-12-24

---

## Problem Statement

Setup shadcn/ui với custom Glass Morphism theme theo Design System cho Synkao POD Order Management.

## Current State

| Component | Status |
|-----------|--------|
| Next.js 16.1.1 + React 19.2.3 | ✅ Done |
| Tailwind CSS v4 với `@theme inline` | ✅ Done |
| Inter font | ✅ Done |
| shadcn/ui | ❌ Not installed |
| Glass effect CSS | ❌ Not defined |
| Design System colors | ❌ Not applied |

---

## Evaluated Approaches

### Option A: CLI Init + Manual Override
- Run `pnpm dlx shadcn@latest init`
- Override CSS after init
- **Rejected**: Risk overwriting existing config, extra merge steps

### Option B: Manual Setup ✅ Selected
- Create `components.json` manually
- Extend `globals.css` with Design System
- Install minimal dependencies
- **Selected**: Full control, YAGNI-compliant, no conflicts

### Option C: Hybrid (CLI + Backup)
- Backup → CLI → Merge
- **Rejected**: Unnecessary complexity

---

## Final Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Approach | Manual Setup | Control, YAGNI |
| Style | **New York** | Default deprecated, rounded fits glass |
| Dark mode | **No** | Not required |
| Mono font | **JetBrains Mono self-hosted** | Offline, performance |

---

## Implementation Plan

### Step 1: Create `components.json`
```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "new-york",
  "tailwind": {
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "hooks": "@/hooks"
  }
}
```

### Step 2: Install Dependencies
```bash
pnpm add lucide-react class-variance-authority
pnpm add -D tw-animate-css
```

### Step 3: Extend globals.css
- Design System colors (primary, semantic, status)
- Glass effect variables
- Remove dark mode media query

### Step 4: Add JetBrains Mono Font
- Download from Google Fonts
- Place in `public/fonts/`
- Configure in layout.tsx

### Step 5: Test with Button Component
```bash
pnpm dlx shadcn@latest add button
```

---

## Risks & Mitigations

| Risk | Mitigation |
|------|-----------|
| HSL vs OKLCH | Use HSL in `:root`, map via `@theme inline` |
| Glass backdrop-filter | 95%+ browser support, acceptable |
| tw-animate-css | Official shadcn/ui recommendation |

---

## Success Criteria

- [ ] `components.json` created with new-york style
- [ ] Design System colors in globals.css
- [ ] Glass effect CSS variables defined
- [ ] JetBrains Mono self-hosted working
- [ ] Button component renders correctly
- [ ] No TypeScript/build errors

---

## References

- [shadcn/ui Tailwind v4](https://ui.shadcn.com/docs/tailwind-v4)
- [shadcn/ui Next.js installation](https://ui.shadcn.com/docs/installation/next)
- [Default vs New York](https://www.shadcndesign.com/blog/difference-between-default-and-new-york-style-in-shadcn-ui)
- [Design System](../docs/synkao-design-system.md)
