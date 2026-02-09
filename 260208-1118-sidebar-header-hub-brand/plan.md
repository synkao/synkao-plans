# Sidebar Header: Replace App Name with Hub Name

**Issue:** [#79](https://github.com/synkao/synkao/issues/79)
**Branch:** `main`
**Complexity:** Low (~30min)

## Overview

Replace fixed "Synkao" branding in sidebar header with dynamic hub name + sublabel. Merge Logo and HubSwitcher into a single unified component.

## Phase 01 — Implementation

**Status:** Pending

### Changes

| File | Action | Description |
|------|--------|-------------|
| `hub-switcher.tsx` | **Modify** | Add logo-style icon, hub name as primary text, sublabel row |
| `glass-sidebar.tsx` | **Modify** | Remove `<Logo>` import/usage, keep only HubSwitcher |
| `logo.tsx` | **Keep** | No changes — still used by admin-sidebar |

### Design Spec

```
┌──────────────────────────┐
│  [Avatar]  Hub Name   ▼  │  ← Hub emoji/icon + name + dropdown chevron
│            Workspace     │  ← Sublabel based on hub type
└──────────────────────────┘
```

**Collapsed state:** Show only hub avatar icon (emoji or colored square)

### Hub Type → Sublabel Map
- `sale` → "Sale"
- `design` → "Design"
- `freelance` → "Freelance"

### Implementation Steps
- [x] Read current HubSwitcher, Logo, GlassSidebar
- [ ] Redesign HubSwitcher: 2-row layout (name + sublabel), larger touch target
- [ ] Update GlassSidebar: remove Logo, adjust padding
- [ ] Verify collapsed state works
- [ ] Build check

### Success Criteria
- Hub name displays as primary heading in sidebar header
- Sublabel shows hub type below name
- Dropdown still opens HubSwitcherDrawer
- Collapsed state shows only avatar icon
- No compile errors
