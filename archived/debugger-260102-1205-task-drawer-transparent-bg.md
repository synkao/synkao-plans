# [RESOLVED] Debug Report: Task Detail Drawer - Transparent Background Issue

**Date:** 2026-01-02
**Reporter:** Debugger Agent
**Severity:** Medium (UI/UX Impact)
**Resolved:** 2026-01-03
**Fix Commit:** `d6a68c5`

---

## Executive Summary

Task Detail Drawer hiển thị nền trong suốt, cho phép nhìn thấy content phía sau. Root cause: **SheetContent component thiếu padding và có thể có vấn đề với CSS variable `--background` trong Tailwind CSS v4**.

---

## Technical Analysis

### 1. Component Structure

**File:** `/home/tuntran/application/github.com/synkao/synkao/apps/web/src/features/design/components/drawer/task-detail-drawer.tsx`

```tsx
<Sheet open={open} onOpenChange={onOpenChange}>
  <SheetContent className="w-full sm:max-w-lg overflow-y-auto">
    {/* Content */}
  </SheetContent>
</Sheet>
```

**Sheet Component:** Wrapper từ shadcn/ui sử dụng `@radix-ui/react-dialog` v1.1.15

### 2. SheetContent Implementation

**File:** `/home/tuntran/application/github.com/synkao/synkao/apps/web/src/components/ui/sheet.tsx` (lines 47-82)

```tsx
function SheetContent({ className, children, side = "right", ...props }) {
  return (
    <SheetPortal>
      <SheetOverlay />
      <SheetPrimitive.Content
        className={cn(
          "bg-background data-[state=open]:animate-in data-[state=closed]:animate-out fixed z-50 flex flex-col gap-4 shadow-lg transition ease-in-out data-[state=closed]:duration-300 data-[state=open]:duration-500",
          side === "right" &&
            "data-[state=closed]:slide-out-to-right data-[state=open]:slide-in-from-right inset-y-0 right-0 h-full w-3/4 border-l sm:max-w-sm",
          // ... other sides
          className
        )}
      >
        {children}
        {/* Close button */}
      </SheetPrimitive.Content>
    </SheetPortal>
  )
}
```

**SheetOverlay Implementation:** (lines 31-45)

```tsx
function SheetOverlay({ className, ...props }) {
  return (
    <SheetPrimitive.Overlay
      className={cn(
        "data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 fixed inset-0 z-50 bg-black/50",
        className
      )}
      {...props}
    />
  )
}
```

### 3. CSS Variables

**File:** `/home/tuntran/application/github.com/synkao/synkao/apps/web/src/app/globals.css`

```css
:root {
  --background: oklch(0.984 0.003 247);  /* Light gray, NO alpha channel */
}

@theme inline {
  --color-background: hsl(var(--background));
}

body {
  background: var(--color-background);
  color: var(--color-foreground);
}
```

### 4. Critical Findings

#### A. Z-Index Conflict
- **SheetOverlay:** `z-50 bg-black/50` (semi-transparent backdrop)
- **SheetContent:** `z-50 bg-background` (drawer panel)

**ISSUE:** Cả overlay và content đều có **cùng z-index `z-50`**. Trong stacking context, nếu overlay render sau content, nó có thể che phủ content, tạo hiệu ứng trong suốt.

#### B. Padding/Spacing Issue
- SheetContent có `gap-4` nhưng **KHÔNG có padding**
- SheetHeader có `p-4` (line 88 sheet.tsx)
- Content section bên trong drawer có `mt-6 space-y-6` nhưng không có horizontal padding

#### C. Tailwind CSS v4 Compatibility
**Package versions:**
- `tailwindcss: ^4`
- `@tailwindcss/postcss: ^4`

Tailwind v4 xử lý CSS variables khác v3. Class `bg-background` có thể không được resolve đúng với `--background: oklch(...)`.

#### D. Background Color Transparency
`--background: oklch(0.984 0.003 247)` **không có alpha channel**, nhưng khi apply `bg-background` class, có thể bị:
- Tailwind v4 không parse đúng OKLCH format
- CSS variable fallback về transparent
- Override bởi browser default

### 5. Stacking Context Analysis

```
<SheetPortal>
  └── <SheetOverlay /> (z-50, bg-black/50, inset-0)
  └── <SheetContent /> (z-50, bg-background, right-0, h-full)
      └── {children}
```

**DOM Order:** Overlay → Content (trong cùng Portal)

Vì cùng z-index và cùng stacking context, thứ tự render quyết định visual stacking. Nếu Overlay render sau Content trong DOM, nó sẽ nằm trên Content → Content bị trong suốt.

**Nhưng:** Code cho thấy Overlay render **trước** Content, nên lý thuyết Content phải nằm trên Overlay. Vậy vấn đề có thể nằm ở:
- CSS `bg-background` không render đúng màu
- Padding thiếu khiến content vẫn hiển thị nhưng background không fill

---

## Evidence from Code

### Task Detail Drawer Usage
**File:** `/home/tuntran/application/github.com/synkao/synkao/apps/web/src/app/(main)/design/backlog/page.tsx` (lines 89-93)

```tsx
<TaskDetailDrawer
  task={selectedTask}
  open={drawerOpen}
  onOpenChange={setDrawerOpen}
/>
```

### Sheet Component từ shadcn/ui
- Base: `@radix-ui/react-dialog`
- Custom styles: Tailwind classes
- No custom CSS overrides detected

---

## Root Cause Hypotheses

### Primary Hypothesis: CSS Variable Resolution Issue
**Probability:** HIGH

Tailwind CSS v4 có thể không resolve `bg-background` đúng với OKLCH color format. Class `bg-background` fallback về `transparent` hoặc inherit.

**Evidence:**
- `--background: oklch(0.984 0.003 247)` sử dụng OKLCH (modern CSS)
- Tailwind v4 mới release, có thể chưa fully support OKLCH trong utility classes
- Không có alpha channel nhưng vẫn transparent → variable không được parse

### Secondary Hypothesis: Padding/Content Layout
**Probability:** MEDIUM

SheetContent thiếu padding horizontal, khiến background color không render đầy đủ.

**Evidence:**
- SheetHeader có `p-4`
- Content section trong drawer không có padding class từ parent
- Override `className="w-full sm:max-w-lg overflow-y-auto"` không có padding

### Tertiary Hypothesis: Z-Index Stacking Bug
**Probability:** LOW

Mặc dù cùng z-50, DOM order cho thấy Content phải nằm trên Overlay.

**Evidence:**
- DOM structure: Overlay render trước Content
- Portal ensure both mount to document.body
- Radix UI Dialog manage stacking correctly

---

## Files Need Fixing

### Priority 1: CSS Variables & Background
1. **`/home/tuntran/application/github.com/synkao/synkao/apps/web/src/app/globals.css`**
   - Check OKLCH color support
   - Consider fallback HSL format
   - Test `--background` variable resolution

### Priority 2: Sheet Component
2. **`/home/tuntran/application/github.com/synkao/synkao/apps/web/src/components/ui/sheet.tsx`**
   - Fix z-index: SheetContent should be `z-[51]` hoặc higher
   - Add padding to SheetContent: `p-6` or similar
   - Ensure `bg-background` applies correctly

### Priority 3: Task Drawer
3. **`/home/tuntran/application/github.com/synkao/synkao/apps/web/src/features/design/components/drawer/task-detail-drawer.tsx`**
   - Add explicit background: `bg-white` or `bg-background`
   - Add padding if needed: `p-6`
   - Consider custom className override

---

## CSS Properties to Verify

### Background
- `background-color`: Should be `oklch(0.984 0.003 247)` or white
- `background`: Should NOT be `transparent`

### Backdrop/Overlay
- `backdrop-filter`: Not used (Overlay use solid `bg-black/50`)
- Overlay opacity: `0.5` (from `/50`)

### Z-Index
- SheetOverlay: `z-50` → `z-49` (below content)
- SheetContent: `z-50` → `z-50` (above overlay)
- OR SheetContent: `z-50` → `z-[51]` (ensure above)

### Opacity
- SheetContent: Should NOT have `opacity < 1`
- Check computed styles for inherited opacity

---

## Testing Recommendations

### 1. Inspect Computed Styles (DevTools)
```javascript
// Check SheetContent background
const sheetContent = document.querySelector('[data-slot="sheet-content"]');
const computed = window.getComputedStyle(sheetContent);
console.log('Background:', computed.backgroundColor);
console.log('Z-Index:', computed.zIndex);
console.log('Opacity:', computed.opacity);
```

### 2. Test CSS Variable
```css
/* Add to globals.css temporarily */
.test-bg-background {
  background: var(--color-background);
  background-color: oklch(0.984 0.003 247);
}
```

### 3. Override Test
```tsx
// In task-detail-drawer.tsx
<SheetContent className="w-full sm:max-w-lg overflow-y-auto bg-white p-6">
```

---

## Next Steps

1. **Verify Tailwind v4 OKLCH support** - Check official docs
2. **Inspect browser DevTools** - Computed styles of SheetContent when drawer open
3. **Test with solid color** - Replace `bg-background` → `bg-white` to confirm
4. **Fix z-index** - Separate overlay and content stacking
5. **Add padding** - Ensure content has proper spacing

---

## Unresolved Questions

1. **Tailwind CSS v4 OKLCH parsing:** Does Tailwind v4 support OKLCH format in utility classes like `bg-background`?
2. **Browser compatibility:** Does target browser support OKLCH color space?
3. **Radix UI Dialog z-index behavior:** Does Radix manage z-index internally that conflict with manual z-50?
4. **Portal mounting order:** Is there race condition in Portal render that cause Overlay mount after Content?
5. **CSS cascade:** Are there other stylesheets overriding `bg-background`?

---

**END REPORT**
