# Root Cause Analysis: Horizontal Scrollbar Issue

**Date:** 2026-01-04
**Reporter:** debugger subagent
**Status:** Investigation Complete

---

## Executive Summary

Horizontal scrollbar xuất hiện ở bottom window do **Kanban board overflow** không được contain properly. Kanban board có tổng width ~1600px (5 columns × 320px) vượt quá viewport width, gây overflow tại body level thay vì main content container.

**Business Impact:** UX degradation - user phải scroll toàn bộ page thay vì chỉ scroll kanban area.

**Root Cause:** Layout containment strategy không đúng - overflow-x-auto được apply ở wrong level.

---

## Technical Analysis

### Layout Hierarchy

```
<html>
  <body> ← HORIZONTAL SCROLLBAR XUẤT HIỆN Ở ĐÂY
    <SidebarProvider> ← w-full flex min-h-svh
      <Sidebar> ← fixed, width: 16rem
      <SidebarInset> ← w-full flex flex-1
        <AppHeader> ← sticky top-0
        <main> ← flex-1 p-6
          <PageContent>
            <KanbanBoard> ← overflow-x-auto (ĐÚNG)
              <div flex gap-4> ← container
                <KanbanColumn w-80> ← 320px × 5 = 1600px
                <KanbanColumn w-80>
                <KanbanColumn w-80>
                <KanbanColumn w-80>
                <KanbanColumn w-80>
```

### Files Affected

| File | Line | Issue | Severity |
|------|------|-------|----------|
| `apps/web/src/components/ui/sidebar.tsx` | 142 | `SidebarProvider` có `w-full` | Medium |
| `apps/web/src/components/ui/sidebar.tsx` | 312 | `SidebarInset` có `w-full` | Medium |
| `apps/web/src/features/design/components/kanban/kanban-board.tsx` | 110 | `overflow-x-auto` ở inner div | High |
| `apps/web/src/features/design/components/kanban/kanban-column.tsx` | 30 | `w-80 flex-shrink-0` × 5 columns | Critical |

### Detailed Breakdown

#### 1. SidebarProvider (sidebar.tsx:142)

```tsx
<div
  data-slot="sidebar-wrapper"
  className="group/sidebar-wrapper has-data-[variant=inset]:bg-sidebar flex min-h-svh w-full"
  // ^^^^^^ w-full - sets 100vw width
>
```

**Problem:** `w-full` forces wrapper to take 100% viewport width, nhưng không prevent children overflow.

#### 2. SidebarInset (sidebar.tsx:312)

```tsx
<main
  data-slot="sidebar-inset"
  className="bg-background relative flex w-full flex-1 flex-col"
  // ^^^^^^ w-full - inherits parent width
>
```

**Problem:** `w-full` + `flex-1` không có `overflow` constraint, children có thể expand freely.

#### 3. KanbanBoard Container (kanban-board.tsx:110)

```tsx
<div className="flex h-full gap-4 overflow-x-auto pb-4">
  {/* 5 columns × 320px = 1600px */}
</div>
```

**Problem:** `overflow-x-auto` ở đây là ĐÚNG, nhưng parent containers phía trên không có width containment → overflow bubbles up to body.

#### 4. KanbanColumn (kanban-column.tsx:30)

```tsx
<div className="flex h-full w-80 flex-shrink-0 flex-col ...">
  {/* Each column: 320px fixed width */}
```

**Problem:**
- `w-80` = 320px fixed width
- `flex-shrink-0` = cannot shrink
- 5 columns + gaps = **~1600px total**
- Exceeds typical viewport (1366px, 1440px, 1920px on most screens)

---

## Evidence

### CSS Computed Values

**Body:**
```css
overflow-x: visible; /* DEFAULT - gây scrollbar */
width: 100vw;
```

**SidebarProvider wrapper:**
```css
display: flex;
width: 100%; /* = 100vw */
min-height: 100svh;
```

**SidebarInset (main content):**
```css
display: flex;
flex: 1 1 0%;
width: 100%; /* = parent width */
overflow: visible; /* NO CONSTRAINT */
```

**Kanban container:**
```css
display: flex;
gap: 1rem;
overflow-x: auto; /* Có scroll, nhưng parent không contain */
```

**Calculations:**
- Sidebar width: 256px (16rem)
- Main content width: `calc(100vw - 256px)`
- Kanban needs: 1600px
- Overflow: `1600px > calc(100vw - 256px)` → **TRUE on most screens**

---

## Root Cause

**Primary Issue:** Layout containment strategy sai

1. `SidebarInset` (main content area) không có `overflow` constraint
2. `w-full` propagates width nhưng không enforce containment
3. Kanban board's `overflow-x-auto` bị override bởi parent's `overflow: visible`
4. Horizontal overflow bubbles up đến body level

**Contributing Factors:**
- Fixed width columns (`w-80`, `flex-shrink-0`)
- No `max-width` trên parent containers
- No `overflow-x: hidden` trên SidebarInset

---

## Recommended Fixes

### Option 1: Add overflow containment to SidebarInset (RECOMMENDED)

**File:** `apps/web/src/components/ui/sidebar.tsx`
**Line:** 312

```tsx
<main
  data-slot="sidebar-inset"
  className={cn(
    "bg-background relative flex w-full flex-1 flex-col overflow-x-hidden",
    // ^^^^^^^^^^^^^^^^^^^^^^^ ADD THIS
    "md:peer-data-[variant=inset]:m-2 ...",
    className
  )}
  {...props}
/>
```

**Impact:**
- ✅ Scrollbar moves inside main content area
- ✅ Body không có horizontal scroll
- ✅ Minimal code change
- ✅ Không break existing layouts

**Tradeoff:** Kanban columns có thể bị cut off edges nếu viewport quá nhỏ (đã có `overflow-x-auto` ở kanban board level để handle)

---

### Option 2: Remove body-level horizontal scroll globally

**File:** `apps/web/src/app/globals.css`
**Line:** 75 (add after existing body styles)

```css
body {
  background: var(--color-background);
  color: var(--color-foreground);
  overflow-x: hidden; /* ADD THIS */
}
```

**Impact:**
- ✅ Prevents horizontal scroll toàn app
- ❌ Global change, có thể ảnh hưởng other pages
- ❌ Không fix root cause, chỉ hide symptom

**Tradeoff:** Có thể hide legitimate overflow issues ở other components.

---

### Option 3: Make kanban responsive (COMPLEX, long-term)

**File:** `apps/web/src/features/design/components/kanban/kanban-column.tsx`
**Line:** 30

```tsx
<div className="flex h-full w-80 min-w-[280px] max-w-[320px] flex-shrink-1 flex-col ...">
  {/* Allow columns to shrink on small screens */}
```

**+ Update kanban-board.tsx line 110:**
```tsx
<div className="flex h-full gap-4 overflow-x-auto pb-4 md:overflow-x-hidden">
  {/* Only scroll on mobile */}
```

**Impact:**
- ✅ Responsive design
- ✅ No scrollbar on desktop (if viewport allows)
- ✅ Better mobile experience
- ❌ Requires more testing
- ❌ May affect kanban UX

---

## Comparison Matrix

| Solution | Complexity | Impact | Side Effects | Recommended |
|----------|-----------|--------|--------------|-------------|
| Option 1: SidebarInset overflow | Low | Medium | Minimal | ⭐⭐⭐⭐⭐ |
| Option 2: Body overflow hidden | Low | High | Potential | ⭐⭐⭐ |
| Option 3: Responsive columns | High | Low-Med | Testing needed | ⭐⭐ (future) |

---

## Implementation Priority

1. **Quick Fix (NOW):** Option 1 - Add `overflow-x-hidden` to SidebarInset
2. **Validate:** Test trên các screen sizes (1366px, 1440px, 1920px)
3. **Future:** Consider Option 3 for full responsive experience

---

## Testing Checklist

- [ ] Desktop (1920px) - no horizontal scrollbar
- [ ] Laptop (1440px) - kanban scrollable inside main area
- [ ] Small laptop (1366px) - same behavior
- [ ] Tablet landscape (1024px) - sidebar collapsed, content scrollable
- [ ] Mobile (375px) - sidebar offcanvas, kanban horizontal scroll

---

## Unresolved Questions

1. Có pages khác ngoài kanban cũng có wide content cần horizontal scroll không?
2. BacklogTable (có `overflow-x-auto`) có bị affect bởi fix này không?
3. User có prefer toàn page scroll hay chỉ content area scroll?
