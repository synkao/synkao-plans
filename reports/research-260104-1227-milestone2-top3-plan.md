# Implementation Plan: Milestone 2 Top 3 Issues

**Date:** 2026-01-04
**Issues:** #6 Layout, #8 Status Badges, #11 Overlays
**Total Effort:** ~4.5h

---

## Table of Contents

1. [Issue #6: Layout Components](#issue-6-layout-components)
2. [Issue #8: Status Badges](#issue-8-status-badges)
3. [Issue #11: Overlay Components](#issue-11-overlay-components)

---

## Issue #6: Layout Components

**Status:** ~80% done
**Remaining Effort:** ~1h

### Current State

```
✅ glass-sidebar.tsx   - SidebarProvider + GlassSidebar
✅ page-header.tsx     - Breadcrumb + title + actions
✅ sidebar-nav.tsx     - Navigation menu
✅ app-header.tsx      - Top header
❌ app-shell.tsx       - Missing wrapper component
❌ content-area.tsx    - Missing content wrapper
```

### What's Already Working

`(main)/layout.tsx` already uses the pattern:
```tsx
<SidebarProvider>
  <GlassSidebar />
  <SidebarInset>
    <AppHeader />
    <main className="flex-1 p-6">{children}</main>
  </SidebarInset>
</SidebarProvider>
```

### Tasks

#### Task 6.1: Create ContentArea component (30 min)

**File:** `apps/web/src/components/layout/content-area.tsx`

```tsx
interface ContentAreaProps {
  children: React.ReactNode;
  className?: string;
  noPadding?: boolean;
}

export function ContentArea({ children, className, noPadding }: ContentAreaProps) {
  return (
    <div className={cn(
      "flex-1",
      !noPadding && "p-6",
      className
    )}>
      {children}
    </div>
  );
}
```

#### Task 6.2: Create AppShell wrapper (optional) (30 min)

**File:** `apps/web/src/components/layout/app-shell.tsx`

Wrapper to encapsulate full layout logic. **YAGNI consideration:** Current layout.tsx already handles this. Only create if reuse needed.

```tsx
interface AppShellProps {
  children: React.ReactNode;
  header?: React.ReactNode;
}

export function AppShell({ children, header }: AppShellProps) {
  return (
    <SidebarProvider>
      <GlassSidebar />
      <SidebarInset>
        {header ?? <AppHeader />}
        <ContentArea>{children}</ContentArea>
      </SidebarInset>
    </SidebarProvider>
  );
}
```

### Acceptance Criteria

- [ ] ContentArea wraps page content with consistent padding
- [ ] Layout exports updated in `components/layout/index.ts`
- [ ] Build passes

---

## Issue #8: Status Badges

**Status:** 0% done
**Effort:** 2h

### Requirements

Domain-specific badge variants for:

| Type | Values |
|------|--------|
| Design Status | TODO, IN_PROGRESS, REVIEW, REVISION, APPROVED |
| Priority | LOW, NORMAL, HIGH, URGENT |
| Fulfillment | PENDING, PROCESSING, SHIPPED, DELIVERED, FAILED |
| Source | NEW_ORDER, REDESIGN, COMPLAINT |

### Tasks

#### Task 8.1: Create status-badge.tsx (1.5h)

**File:** `apps/web/src/components/ui/status-badge.tsx`

```tsx
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

// Design Status Badge
const designStatusVariants = cva(
  "inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium",
  {
    variants: {
      status: {
        TODO: "bg-slate-100 text-slate-700",
        IN_PROGRESS: "bg-blue-100 text-blue-700",
        REVIEW: "bg-purple-100 text-purple-700",
        REVISION: "bg-amber-100 text-amber-700",
        APPROVED: "bg-green-100 text-green-700",
      },
    },
  }
);

export function DesignStatusBadge({
  status,
  className
}: { status: keyof typeof designStatusVariants.variants.status } & { className?: string }) {
  return (
    <span className={cn(designStatusVariants({ status }), className)}>
      {status.replace("_", " ")}
    </span>
  );
}

// Priority Badge
const priorityVariants = cva(
  "inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium",
  {
    variants: {
      priority: {
        LOW: "bg-gray-100 text-gray-600",
        NORMAL: "bg-blue-100 text-blue-700",
        HIGH: "bg-amber-100 text-amber-700",
        URGENT: "bg-red-100 text-red-700",
      },
    },
  }
);

export function PriorityBadge({
  priority,
  className
}: { priority: keyof typeof priorityVariants.variants.priority } & { className?: string }) {
  return (
    <span className={cn(priorityVariants({ priority }), className)}>
      {priority}
    </span>
  );
}

// Fulfillment Badge
const fulfillmentVariants = cva(
  "inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium",
  {
    variants: {
      status: {
        PENDING: "bg-slate-100 text-slate-600",
        PROCESSING: "bg-blue-100 text-blue-700",
        SHIPPED: "bg-indigo-100 text-indigo-700",
        DELIVERED: "bg-green-100 text-green-700",
        FAILED: "bg-red-100 text-red-700",
      },
    },
  }
);

export function FulfillmentBadge({
  status,
  className
}: { status: keyof typeof fulfillmentVariants.variants.status } & { className?: string }) {
  return (
    <span className={cn(fulfillmentVariants({ status }), className)}>
      {status}
    </span>
  );
}

// Source Badge
const sourceVariants = cva(
  "inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium border",
  {
    variants: {
      source: {
        NEW_ORDER: "bg-emerald-50 text-emerald-700 border-emerald-200",
        REDESIGN: "bg-sky-50 text-sky-700 border-sky-200",
        COMPLAINT: "bg-rose-50 text-rose-700 border-rose-200",
      },
    },
  }
);

export function SourceBadge({
  source,
  className
}: { source: keyof typeof sourceVariants.variants.source } & { className?: string }) {
  return (
    <span className={cn(sourceVariants({ source }), className)}>
      {source.replace("_", " ")}
    </span>
  );
}
```

#### Task 8.2: Create TypeScript types (30 min)

**File:** `apps/web/src/types/status.ts`

```tsx
export type DesignStatus = "TODO" | "IN_PROGRESS" | "REVIEW" | "REVISION" | "APPROVED";
export type Priority = "LOW" | "NORMAL" | "HIGH" | "URGENT";
export type FulfillmentStatus = "PENDING" | "PROCESSING" | "SHIPPED" | "DELIVERED" | "FAILED";
export type OrderSource = "NEW_ORDER" | "REDESIGN" | "COMPLAINT";
```

### Acceptance Criteria

- [ ] All 4 badge types created with correct colors
- [ ] TypeScript types exported
- [ ] Badges use cva pattern consistent with existing badge.tsx
- [ ] Build passes

---

## Issue #11: Overlay Components

**Status:** ~50% done
**Remaining Effort:** ~1.5h

### Current State

```
✅ dialog.tsx   - Modal with sizes
✅ sheet.tsx    - Drawer (left/right)
✅ tooltip.tsx  - Tooltip
❌ sonner.tsx   - Toast notifications (MISSING)
❌ confirm-dialog.tsx - Confirmation dialog (MISSING)
```

### Tasks

#### Task 11.1: Add Sonner for Toast (45 min)

**Step 1:** Install sonner
```bash
pnpm add sonner --filter web
```

**Step 2:** Create toast wrapper

**File:** `apps/web/src/components/ui/sonner.tsx`

```tsx
"use client"

import { Toaster as Sonner } from "sonner"

type ToasterProps = React.ComponentProps<typeof Sonner>

function Toaster({ ...props }: ToasterProps) {
  return (
    <Sonner
      theme="light"
      className="toaster group"
      toastOptions={{
        classNames: {
          toast:
            "group toast group-[.toaster]:bg-background group-[.toaster]:text-foreground group-[.toaster]:border-border group-[.toaster]:shadow-lg",
          description: "group-[.toast]:text-muted-foreground",
          actionButton:
            "group-[.toast]:bg-primary group-[.toast]:text-primary-foreground",
          cancelButton:
            "group-[.toast]:bg-muted group-[.toast]:text-muted-foreground",
        },
      }}
      {...props}
    />
  )
}

export { Toaster }
```

**Step 3:** Add to root layout

```tsx
// apps/web/src/app/layout.tsx
import { Toaster } from "@/components/ui/sonner"

// In body:
<Toaster />
```

**Usage:**
```tsx
import { toast } from "sonner"

toast.success("Order created!")
toast.error("Failed to save")
toast.info("Processing...")
toast.warning("Low stock")
```

#### Task 11.2: Create ConfirmDialog (45 min)

**File:** `apps/web/src/components/ui/confirm-dialog.tsx`

```tsx
"use client"

import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
} from "@/components/ui/alert-dialog"

interface ConfirmDialogProps {
  open: boolean
  onOpenChange: (open: boolean) => void
  title: string
  description?: string
  confirmText?: string
  cancelText?: string
  onConfirm: () => void
  variant?: "default" | "destructive"
}

export function ConfirmDialog({
  open,
  onOpenChange,
  title,
  description,
  confirmText = "Confirm",
  cancelText = "Cancel",
  onConfirm,
  variant = "default",
}: ConfirmDialogProps) {
  return (
    <AlertDialog open={open} onOpenChange={onOpenChange}>
      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle>{title}</AlertDialogTitle>
          {description && (
            <AlertDialogDescription>{description}</AlertDialogDescription>
          )}
        </AlertDialogHeader>
        <AlertDialogFooter>
          <AlertDialogCancel>{cancelText}</AlertDialogCancel>
          <AlertDialogAction
            onClick={onConfirm}
            className={variant === "destructive" ? "bg-destructive text-white hover:bg-destructive/90" : ""}
          >
            {confirmText}
          </AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  )
}
```

**Note:** Requires `alert-dialog` from shadcn:
```bash
pnpm dlx shadcn@latest add alert-dialog --cwd apps/web
```

### Acceptance Criteria

- [ ] Sonner toast working with 4 variants (success, error, info, warning)
- [ ] ConfirmDialog with destructive variant for delete actions
- [ ] Toast added to root layout
- [ ] Build passes

---

## File Structure Summary

```
apps/web/src/
├── components/
│   ├── layout/
│   │   ├── index.ts              # Export all
│   │   ├── content-area.tsx      # NEW #6
│   │   └── app-shell.tsx         # OPTIONAL #6
│   └── ui/
│       ├── status-badge.tsx      # NEW #8
│       ├── sonner.tsx            # NEW #11
│       ├── confirm-dialog.tsx    # NEW #11
│       └── alert-dialog.tsx      # ADD via shadcn #11
├── types/
│   └── status.ts                 # NEW #8
└── app/
    └── layout.tsx                # MODIFY: add Toaster
```

---

## Implementation Order

```
1. #11 Sonner     (45 min) ← Add toast first, useful for feedback
2. #8 Badges      (2h)     ← Domain components for Kanban
3. #6 Layout      (1h)     ← Finish layout wrappers
4. #11 Confirm    (45 min) ← Last, less critical
```

---

## Commands to Run

```bash
# Install dependencies
pnpm add sonner --filter web
pnpm dlx shadcn@latest add alert-dialog --cwd apps/web

# Verify build
pnpm build --filter web
pnpm lint --filter web
```

---

## Success Metrics

- [ ] All 4 badge types render correctly
- [ ] Toast notifications appear on actions
- [ ] ConfirmDialog blocks destructive actions
- [ ] Layout components exported properly
- [ ] No TypeScript errors
- [ ] Build passes

---

## Unresolved Questions

1. Glass Card (#7) - Should it extend existing card.tsx or create new glass-card.tsx?
2. Storybook - Need stories for each component?
