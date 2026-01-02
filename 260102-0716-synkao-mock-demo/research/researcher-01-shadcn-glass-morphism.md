# Research: shadcn/ui + Glass Morphism Implementation
**Date:** 2026-01-02 | **Project:** SynKao Mock Demo | **Status:** Complete

---

## 1. shadcn/ui Component Library

### Available Components (70+)
**Navigation & Layout:**
- `sidebar` - Composable, themeable sidebar with collapsible states
- `navigation-menu` - Multi-level navigation support
- `breadcrumb` - Path navigation

**Data Display:**
- `data-table` - TanStack Table integration (powerful datagrids)
- `table` - Basic table component
- `pagination` - Paginated data display

**Feedback & Modals:**
- `sheet` / `drawer` - Sliding panels (equivalent)
- `dialog` - Modal windows
- `alert` / `alert-dialog` - Alert boxes

**Forms & Input:**
- `form` - Form builder with validation
- `input`, `textarea`, `select`, `checkbox`, `radio-group`
- `date-picker` - Calendar input

### Installation Commands
```bash
# Core components needed for mock demo
npx shadcn-ui@latest add sidebar
npx shadcn-ui@latest add data-table
npx shadcn-ui@latest add sheet
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add card
npx shadcn-ui@latest add button
npx shadcn-ui@latest add input
npx shadcn-ui@latest add dropdown-menu
```

### Kanban Note
**Kanban is NOT a core component** in shadcn/ui. Options:
1. Use community packages (react-beautiful-dnd, dnd-kit)
2. Build custom with Card + Drag-drop utilities
3. Check shadcn Registry Directory for third-party implementations

### Customization Patterns
- All components use Radix UI primitives underneath
- Customization via Tailwind utility classes
- CSS variables for theme colors
- Copy-to-clipboard approach (not npm package) enables full customization

---

## 2. Glass Morphism CSS Implementation

### Tailwind CSS 4 Approach
**Core Utilities:**
```html
<!-- Basic glass effect -->
<div class="backdrop-blur-sm backdrop-brightness-95 bg-white/40 border border-white/20">
  Content
</div>
```

**Tailwind Classes Available:**
- `backdrop-blur-{xs|sm|md|lg|xl|2xl}` - Blur intensity
- `backdrop-brightness-*` - Brightness adjustment
- `backdrop-contrast-*` - Contrast control
- `backdrop-grayscale` - Grayscale effect
- `backdrop-saturate-*` - Color saturation
- `backdrop-opacity-*` - Opacity override

### CSS Spec (from Brainstorm)
```css
background: rgba(255, 255, 255, 0.6);
backdrop-filter: blur(12px);
border: 1px solid rgba(226, 232, 240, 0.8);
```

**Tailwind Equivalent:**
```html
<div class="bg-white/60 backdrop-blur-md border border-slate-200/80">
</div>
```

### Best Practices
1. **Use RGBA for transparency** - `bg-white/60` = 60% opacity
2. **Combine with borders** - Subtle borders enhance glass effect
3. **Stack filters cautiously** - Multiple filters reduce performance
4. **Test on background images** - Glass morphism effect depends on what's behind
5. **Provide solid fallback** - Older browsers don't support backdrop-filter

### Browser Compatibility
| Browser | Support | Notes |
|---------|---------|-------|
| Chrome/Edge | ✅ 76+ | Full support |
| Firefox | ✅ 103+ | Full support |
| Safari | ✅ 9+ | Good support |
| Mobile Safari | ✅ 9+ | Good support |

**Fallback Strategy:**
```css
@supports (backdrop-filter: blur(1px)) {
  /* Glass morphism implementation */
}

/* Fallback: solid background */
background-color: rgba(255, 255, 255, 0.9);
```

---

## 3. Sidebar Navigation Patterns

### shadcn/ui Sidebar Structure
```tsx
<SidebarProvider>
  <Sidebar>
    <SidebarHeader />
    <SidebarContent>
      <SidebarGroup>
        <SidebarGroupLabel>Menu</SidebarGroupLabel>
        <SidebarGroupContent>
          <SidebarMenu>
            <SidebarMenuItem>
              <SidebarMenuButton asChild isActive>
                <a href="#">Active Item</a>
              </SidebarMenuButton>
            </SidebarMenuItem>
          </SidebarMenu>
        </SidebarGroupContent>
      </SidebarGroup>
    </SidebarContent>
    <SidebarFooter />
  </Sidebar>

  <main>{children}</main>
</SidebarProvider>
```

### Configuration Options
**Sidebar Props:**
- `side` - `"left"` | `"right"`
- `variant` - `"sidebar"` | `"floating"` | `"inset"`
- `collapsible` - `"offcanvas"` | `"icon"` | `"none"`

### Active State Handling
1. **Direct Method:** Use `isActive` prop
   ```tsx
   <SidebarMenuButton isActive={pathname === '/dashboard'}>
   ```

2. **With Route Groups:** Use Next.js App Router route groups
   ```
   (dashboard)/layout.tsx → Sidebar with dashboard routes marked active
   ```

3. **Keyboard Shortcut:** Default `cmd+b` / `ctrl+b` to toggle

### State Management
- Use `useSidebar()` hook for programmatic control
- Properties: `state`, `open`, `setOpen()`, `toggleSidebar()`
- Integrates with localStorage for persistence

---

## 4. Implementation Recommendations

### Component Installation Order
1. **Foundation:** button, card, input, label
2. **Navigation:** sidebar, dropdown-menu, breadcrumb
3. **Data:** data-table, pagination
4. **Modals:** dialog, sheet, alert-dialog
5. **Forms:** form, select, checkbox

### Glass Morphism Integration with Sidebar
```tsx
<Sidebar className="border-0 bg-white/40 backdrop-blur-md">
  {/* Sidebar content */}
</Sidebar>
```

### For Kanban Board
**Recommendation:** Use `dnd-kit` (modern, performant)
```bash
npm install @dnd-kit/sortable @dnd-kit/utilities @dnd-kit/core
```

Then build columns with shadcn `card` components + drag handlers.

---

## 5. Unresolved Questions
- Specific Kanban library preference for mock demo (dnd-kit vs react-beautiful-dnd)?
- Should glass morphism apply to sidebar, modals, or both?
- Data table pagination style preferences?
- Dark mode support needed for glass morphism effects?
