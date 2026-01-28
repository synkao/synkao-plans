# Scout Report: Admin Spaces Feature Analysis & Terminology Mapping

**Report Date**: 2026-01-28 22:30  
**Scope**: Comprehensive analysis of "space" terminology usage in admin portal  
**Task**: Map all locations where "space" would need to change to "hub"

---

## Executive Summary

The admin spaces feature is currently a **placeholder implementation** with comprehensive terminology usage across routes, components, UI text, and documentation. Identified **13 primary files** with space-related nomenclature and **multiple layers of terminology** that require coordinated changes for a space→hub migration.

---

## 1. File & Directory Structure Changes

### 1.1 Route/Directory Paths (MUST CHANGE)

| Current Path | Proposed Path | Type | Priority |
|---|---|---|---|
| `apps/web/src/app/(admin)/admin/spaces/` | `apps/web/src/app/(admin)/admin/hubs/` | Directory | P0 |
| `apps/web/src/app/(admin)/admin/spaces/[id]/` | `apps/web/src/app/(admin)/admin/hubs/[id]/` | Directory | P0 |
| `apps/web/src/app/(admin)/admin/spaces/page.tsx` | `apps/web/src/app/(admin)/admin/hubs/page.tsx` | File | P0 |
| `apps/web/src/app/(admin)/admin/spaces/[id]/page.tsx` | `apps/web/src/app/(admin)/admin/hubs/[id]/page.tsx` | File | P0 |

---

## 2. Component & Page Changes (MUST CHANGE)

### 2.1 Page Components - Admin Spaces

**File**: `/Users/taquanglinh/Documents/synkao/apps/web/src/app/(admin)/admin/spaces/page.tsx`

Current exports:
- Function: `AdminSpacesPage()` → `AdminHubsPage()`

Current terminology in file:
- `<h1>Spaces</h1>` → `<h1>Hubs</h1>`
- Button text: `Add Space` → `Add Hub`
- Card title: `Spaces Management` → `Hubs Management`
- Description: `Manage workspaces and tenant configurations` (keep or update?)
- Copy: `Space management features will be available soon` → `Hub management features will be available soon`

**Line-by-line changes**:
```
Line 5:  export default function AdminSpacesPage() 
         → export default function AdminHubsPage()

Line 11: <h1 className="text-3xl font-bold tracking-tight">Spaces</h1>
         → <h1 className="text-3xl font-bold tracking-tight">Hubs</h1>

Line 18: <Plus className="mr-2 h-4 w-4" />
         Add Space
         → Add Hub

Line 27: <CardTitle className="flex items-center gap-2">
         Spaces Management
         → Hubs Management

Line 30: This page is under development. Space management features will be available soon.
         → This page is under development. Hub management features will be available soon.

Line 38: Space management functionality including CRUD operations,
         → Hub management functionality including CRUD operations,
```

---

### 2.2 Page Components - Admin Space Detail

**File**: `/Users/taquanglinh/Documents/synkao/apps/web/src/app/(admin)/admin/spaces/[id]/page.tsx`

Current exports:
- Interface: `SpaceDetailPageProps` → `HubDetailPageProps`
- Function: `SpaceDetailPage()` → `HubDetailPage()`

Current terminology in file:
```
Line 7:  interface SpaceDetailPageProps
         → interface HubDetailPageProps

Line 11: export default function SpaceDetailPage({ params }: SpaceDetailPageProps)
         → export default function HubDetailPage({ params }: HubDetailPageProps)

Line 19: <Link href="/admin/spaces">
         → <Link href="/admin/hubs">

Line 24: <h1 className="text-3xl font-bold tracking-tight">Space Details</h1>
         → <h1 className="text-3xl font-bold tracking-tight">Hub Details</h1>

Line 26: Viewing space ID: {id}
         → Viewing hub ID: {id}

Line 36: <CardTitle>Space Information</CardTitle>
         → <CardTitle>Hub Information</CardTitle>

Line 39: Detailed space configuration and settings
         → Detailed hub configuration and settings

Line 47: Space detail view with configuration, members, and analytics
         → Hub detail view with configuration, members, and analytics
```

---

## 3. Navigation Configuration Changes

### 3.1 Navigation Data File

**File**: `/Users/taquanglinh/Documents/synkao/apps/web/src/lib/navigation.ts`

Admin navigation array definitions:

**Lines 72-74**:
```ts
{
  title: "Spaces",        → title: "Hubs",
  href: "/admin/spaces",  → href: "/admin/hubs",
  icon: Building2,
},
```

**Line 86**:
```ts
"/admin/spaces": "Spaces",  → "/admin/hubs": "Hubs",
```

**Impact**: Updates admin navigation menu + breadcrumb labels

---

## 4. Admin UI Components (SHOULD REVIEW)

### 4.1 Stats Cards Component

**File**: `/Users/taquanglinh/Documents/synkao/apps/web/src/features/admin/components/stats-cards.tsx`

**Lines 49-53**: Mock data definition
```ts
{
  title: 'Active Spaces',          → title: 'Active Hubs',
  value: '48',
  description: 'from last month',
  icon: <Building2 className="h-4 w-4 text-muted-foreground" />,
  trend: { value: 8.2, isPositive: true },
},
```

**Impact**: Admin dashboard statistics label

---

## 5. Related Components & Navigation Support

### 5.1 Admin Sidebar Navigation Component

**File**: `/Users/taquanglinh/Documents/synkao/apps/web/src/components/layout/admin-sidebar-nav.tsx`

**Status**: No direct space terminology (uses generic adminNavigation array)  
**Change Required**: NO (it dynamically renders from navigation.ts)

### 5.2 Admin Header Component

**File**: `/Users/taquanglinh/Documents/synkao/apps/web/src/components/layout/admin-header.tsx`

**Status**: No direct space terminology  
**Change Required**: NO (generic component)

---

## 6. Plan/Documentation References

### 6.1 Implementation Plan Files

**File**: `/Users/taquanglinh/Documents/synkao/plans/260128-2130-admin-portal/plan.md`

Lines that reference spaces:
- Line 16: "placeholder pages for space and user management"
- Line 34: Space route structure documentation
- Line 36: "Spaces" → "Hubs"

**File**: `/Users/taquanglinh/Documents/synkao/plans/260128-2130-admin-portal/phase-04-placeholder-pages.md`

**Multiple references** (comprehensive planning document):
- Lines 20, 32-33: Space management references
- Lines 45-49: Architecture diagram with "spaces/" directory
- Lines 58-62: File list to create
- Lines 68-172: Code implementation templates with extensive Space references
- Lines 315-335: Todo list and success criteria

**Detailed change list**:
```
Line 20: /admin/spaces, /admin/spaces/[id]
Line 32: Shows "Spaces Management" header
Line 33: /admin/spaces, /admin/spaces/[id]
Line 36: "Space Details" with ID from params
Line 45-49: Directory structure diagram
Line 75-103: Code template with function names, headings, descriptions
Line 108-172: Space detail page template
Line 328-329: Success criteria with /admin/spaces route tests
```

---

## 7. Mock Data & Types (REVIEW FOR CONTEXT)

### 7.1 Files with "workspace" terminology (different from "space")

These files use **"workspace"** not "space" - they are related but separate concepts:

| File | Type | Purpose | Action |
|---|---|---|---|
| `apps/web/src/mocks/data/workspaces.data.ts` | Mock Data | Mock workspace entities | No change (workspace ≠ space) |
| `apps/web/src/mocks/handlers/workspaces.ts` | API Handler | Mock workspace endpoints | No change |
| `apps/web/src/lib/navigation.ts` (other refs) | Navigation | Workspace route navigation | No change |

**Note**: The codebase distinguishes between "Workspace" (design module feature) and "Space" (admin portal feature). These are separate concepts.

---

## 8. Cascade Effects & Related Changes

### 8.1 All Files That Need Updating

```
Priority P0 - CRITICAL (rename/restructure):
✓ apps/web/src/app/(admin)/admin/spaces/page.tsx
✓ apps/web/src/app/(admin)/admin/spaces/[id]/page.tsx
✓ Directory: apps/web/src/app/(admin)/admin/spaces/

Priority P1 - HIGH (content updates):
✓ apps/web/src/lib/navigation.ts
✓ apps/web/src/features/admin/components/stats-cards.tsx
✓ plans/260128-2130-admin-portal/plan.md
✓ plans/260128-2130-admin-portal/phase-04-placeholder-pages.md

Priority P2 - MEDIUM (documentation/comments):
✓ Any code comments referencing space terminology
✓ Component documentation strings
```

---

## 9. Terminology Mapping Reference

### 9.1 Quick Search & Replace Guide

For consistent terminology updates:

| Find | Replace | Scope |
|---|---|---|
| `admin/spaces` | `admin/hubs` | Routes, href attributes, links |
| `AdminSpacesPage` | `AdminHubsPage` | Component function names |
| `SpaceDetailPage` | `HubDetailPage` | Component function names |
| `SpaceDetailPageProps` | `HubDetailPageProps` | Interface names |
| `"Spaces"` | `"Hubs"` | UI labels, titles |
| `"Space "` | `"Hub "` | Descriptive text (Space Details, Space Management, Space Information) |
| `"Active Spaces"` | `"Active Hubs"` | Stats card titles |

---

## 10. Implementation Checklist

### Phase 1: File Structure
- [ ] Rename directory: `apps/web/src/app/(admin)/admin/spaces/` → `.../hubs/`
- [ ] Move `spaces/page.tsx` → `hubs/page.tsx`
- [ ] Move `spaces/[id]/page.tsx` → `hubs/[id]/page.tsx`

### Phase 2: Component Code
- [ ] Update function names: `AdminSpacesPage` → `AdminHubsPage`
- [ ] Update interface names: `SpaceDetailPageProps` → `HubDetailPageProps`
- [ ] Update all UI text and labels in components

### Phase 3: Navigation & Routing
- [ ] Update `lib/navigation.ts` - adminNavigation array
- [ ] Update `lib/navigation.ts` - adminRouteLabels object
- [ ] Update href attributes from `/admin/spaces` to `/admin/hubs`

### Phase 4: Dashboard Stats
- [ ] Update stats card title in `features/admin/components/stats-cards.tsx`

### Phase 5: Documentation
- [ ] Update plan.md with "hub" terminology
- [ ] Update phase-04-placeholder-pages.md
  - Update code examples
  - Update directory diagrams
  - Update success criteria

### Phase 6: Testing
- [ ] Test route navigation /admin/hubs works
- [ ] Test /admin/hubs/[id] dynamic routes
- [ ] Verify breadcrumbs display "Hubs"
- [ ] Test "Back to App" from hub pages
- [ ] Build and verify no TypeScript errors

---

## 11. Risk Assessment

| Risk | Severity | Mitigation |
|---|---|---|
| Route path changes break navigation | HIGH | Update all href attributes, test navigation thoroughly |
| Component imports fail | MEDIUM | Verify TypeScript compilation after renames |
| Users/docs reference old paths | LOW | Update any user-facing documentation |
| Dead links in plan documents | LOW | Automated link check in plan files |

---

## 12. Dependencies & Notes

### Import Impact
- ✅ All imports are local to admin feature - no cross-feature impacts
- ✅ Navigation config is centralized - single file to update
- ✅ Routes use filesystem-based routing - directory rename automatically works

### Workspace vs Space Distinction
- **Workspace**: Existing feature in Design module (design/workspace)
- **Space**: New admin portal feature (admin/spaces → admin/hubs)
- **Action**: Keep workspaces as-is, only update spaces terminology

### Mock Data
- Mock data for spaces/hubs is currently hardcoded in stats card
- Future API integration will need space/hub model definitions
- No database schema changes needed at placeholder phase

---

## Summary Statistics

| Category | Count |
|---|---|
| Primary files requiring changes | 6 |
| Directories to rename | 1 |
| Component functions to rename | 2 |
| Interfaces to rename | 1 |
| Navigation entries to update | 2 |
| UI text labels to update | 8+ |
| Documentation files to update | 2 |
| Estimated effort | 30-45 minutes |

---

## Unresolved Questions

1. **Database Schema**: Will "Space" become "Hub" in the database schema? Currently unknown - placeholder phase only
2. **API Endpoints**: Future API endpoints should they be `/api/hubs` or `/api/admin/hubs`?
3. **Icon Choice**: Is Building2 icon still appropriate for "Hubs"? Consider alternatives like Hub icon if available
4. **Workspace Relationship**: How do Hubs relate to Workspaces? Need clarification for future features

