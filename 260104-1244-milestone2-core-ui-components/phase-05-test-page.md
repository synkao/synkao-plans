# Phase 05: UI Test Page

## Context

- **Parent Plan:** [plan.md](plan.md)
- **Dependencies:** Phase 01, 02, 04

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P1 |
| Effort | 30 min |
| Implementation | ⬜ Pending |
| Review | ⬜ Pending |

## Key Insights

- Existing `/design-system` page showcases base components
- Need dedicated test page for new components
- Interactive testing for Toast and ConfirmDialog

## Requirements

1. Create `/dev-tools/ui-test` page
2. Test all 4 badge types with all variants
3. Interactive toast buttons
4. ConfirmDialog demo

## Related Files

| File | Action |
|------|--------|
| `apps/web/src/app/dev-tools/ui-test/page.tsx` | Create |

## Implementation

**File:** `apps/web/src/app/dev-tools/ui-test/page.tsx`

```tsx
"use client";

import { useState } from "react";
import { toast } from "sonner";
import { Button } from "@/components/ui/button";
import {
  DesignStatusBadge,
  PriorityBadge,
  FulfillmentBadge,
  SourceBadge,
} from "@/components/ui/status-badge";
import { ConfirmDialog } from "@/components/ui/confirm-dialog";
import { ContentArea } from "@/components/layout";

export default function UITestPage() {
  const [showConfirm, setShowConfirm] = useState(false);
  const [showDestructive, setShowDestructive] = useState(false);

  return (
    <ContentArea className="space-y-12">
      <header>
        <h1 className="text-2xl font-bold">UI Components Test</h1>
        <p className="text-muted-foreground">
          Test page for Milestone 2 components
        </p>
      </header>

      {/* Toast Section */}
      <section className="space-y-4">
        <h2 className="text-lg font-semibold">🔔 Toast Notifications</h2>
        <div className="flex flex-wrap gap-2">
          <Button onClick={() => toast.success("Success message!")}>
            Success
          </Button>
          <Button
            variant="destructive"
            onClick={() => toast.error("Error message!")}
          >
            Error
          </Button>
          <Button variant="outline" onClick={() => toast.info("Info message!")}>
            Info
          </Button>
          <Button
            variant="secondary"
            onClick={() => toast.warning("Warning message!")}
          >
            Warning
          </Button>
          <Button
            variant="ghost"
            onClick={() =>
              toast("Default toast", {
                description: "With description text",
                action: { label: "Undo", onClick: () => console.log("Undo") },
              })
            }
          >
            With Action
          </Button>
        </div>
      </section>

      {/* Status Badges Section */}
      <section className="space-y-6">
        <h2 className="text-lg font-semibold">🏷️ Status Badges</h2>

        {/* Design Status */}
        <div className="glass rounded-xl p-4 space-y-2">
          <h3 className="font-medium text-sm">Design Status</h3>
          <div className="flex flex-wrap gap-2">
            <DesignStatusBadge status="TODO" />
            <DesignStatusBadge status="IN_PROGRESS" />
            <DesignStatusBadge status="REVIEW" />
            <DesignStatusBadge status="REVISION" />
            <DesignStatusBadge status="APPROVED" />
          </div>
        </div>

        {/* Priority */}
        <div className="glass rounded-xl p-4 space-y-2">
          <h3 className="font-medium text-sm">Priority</h3>
          <div className="flex flex-wrap gap-2">
            <PriorityBadge priority="LOW" />
            <PriorityBadge priority="NORMAL" />
            <PriorityBadge priority="HIGH" />
            <PriorityBadge priority="URGENT" />
          </div>
        </div>

        {/* Fulfillment */}
        <div className="glass rounded-xl p-4 space-y-2">
          <h3 className="font-medium text-sm">Fulfillment Status</h3>
          <div className="flex flex-wrap gap-2">
            <FulfillmentBadge status="PENDING" />
            <FulfillmentBadge status="PROCESSING" />
            <FulfillmentBadge status="SHIPPED" />
            <FulfillmentBadge status="DELIVERED" />
            <FulfillmentBadge status="FAILED" />
          </div>
        </div>

        {/* Source */}
        <div className="glass rounded-xl p-4 space-y-2">
          <h3 className="font-medium text-sm">Order Source</h3>
          <div className="flex flex-wrap gap-2">
            <SourceBadge source="NEW_ORDER" />
            <SourceBadge source="REDESIGN" />
            <SourceBadge source="COMPLAINT" />
          </div>
        </div>
      </section>

      {/* ConfirmDialog Section */}
      <section className="space-y-4">
        <h2 className="text-lg font-semibold">💬 Confirm Dialog</h2>
        <div className="flex flex-wrap gap-2">
          <Button variant="outline" onClick={() => setShowConfirm(true)}>
            Default Confirm
          </Button>
          <Button variant="destructive" onClick={() => setShowDestructive(true)}>
            Destructive Confirm
          </Button>
        </div>

        <ConfirmDialog
          open={showConfirm}
          onOpenChange={setShowConfirm}
          title="Confirm Action"
          description="Are you sure you want to proceed with this action?"
          onConfirm={() => toast.success("Confirmed!")}
        />

        <ConfirmDialog
          open={showDestructive}
          onOpenChange={setShowDestructive}
          title="Delete Item?"
          description="This action cannot be undone. The item will be permanently deleted."
          confirmText="Delete"
          variant="destructive"
          onConfirm={() => toast.success("Deleted!")}
        />
      </section>

      {/* ContentArea Demo */}
      <section className="space-y-4">
        <h2 className="text-lg font-semibold">📦 ContentArea</h2>
        <div className="border border-dashed border-border rounded-xl p-4">
          <p className="text-sm text-muted-foreground">
            This page uses ContentArea wrapper with default padding (p-6).
          </p>
        </div>
      </section>
    </ContentArea>
  );
}
```

## Todo

- [ ] Create ui-test page
- [ ] Test all toast variants
- [ ] Test all badge types
- [ ] Test both ConfirmDialog variants

## Success Criteria

- [ ] Page accessible at `/dev-tools/ui-test`
- [ ] All toast buttons work
- [ ] All badges render correctly
- [ ] ConfirmDialog opens/closes properly

## Next Steps

After all phases → Run final build verification
