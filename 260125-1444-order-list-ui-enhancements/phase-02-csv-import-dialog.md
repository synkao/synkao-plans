---
title: Phase 2 - CSV Import Dialog
status: completed
priority: high
effort: 3
tags: [csv, import, dialog, ui]
completed: 2026-01-25T15:01:00Z
---

# Phase 2: CSV Import Dialog

## Context
- [orders-page-header.tsx](../../apps/web/src/app/(main)/orders/orders-page-header.tsx) - Import button (Link to /orders/import)
- [file-uploader.tsx](../../apps/web/src/components/ui/file-uploader.tsx) - FileUploader with react-dropzone
- [dialog.tsx](../../apps/web/src/components/ui/dialog.tsx) - Dialog component

## Overview
- **Priority:** P2
- **Status:** ✅ Completed
- **Scope:** Import button opens dialog with CSV drag-drop (UI only)

## Requirements

### Functional
- Click Import button → Opens dialog (not navigate to page)
- Dialog contains FileUploader for CSV
- Show selected file info
- Import button (placeholder, no API)

### Non-functional
- Accept only .csv files
- Max 1 file
- Max 10MB file size
- No API integration in this phase

## Architecture

```
OrdersPageHeader
  ├── Import Button (onClick → open dialog)
  └── ImportOrdersDialog (controlled)
       └── FileUploader (CSV only)
```

## Related Files

### Create
| File | Purpose |
|------|---------|
| `import-orders-dialog.tsx` | Dialog with FileUploader |

### Modify
| File | Changes |
|------|---------|
| `orders-page-header.tsx` | Replace Link with dialog trigger |

## Implementation

### Step 1: Create `import-orders-dialog.tsx`

```tsx
// apps/web/src/features/orders/components/order-list/import-orders-dialog.tsx
'use client';

import { useState } from 'react';
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogDescription,
  DialogFooter,
} from '@/components/ui/dialog';
import { Button } from '@/components/ui/button';
import { FileUploader } from '@/components/ui/file-uploader';
import { Upload } from 'lucide-react';

interface ImportOrdersDialogProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  onImport?: (files: File[]) => void;
}

export function ImportOrdersDialog({
  open,
  onOpenChange,
  onImport,
}: ImportOrdersDialogProps) {
  const [files, setFiles] = useState<File[]>([]);

  const handleImport = () => {
    if (files.length > 0) {
      onImport?.(files);
      // Reset and close after import
      setFiles([]);
      onOpenChange(false);
    }
  };

  const handleClose = () => {
    setFiles([]);
    onOpenChange(false);
  };

  return (
    <Dialog open={open} onOpenChange={handleClose}>
      <DialogContent className="sm:max-w-lg">
        <DialogHeader>
          <DialogTitle className="flex items-center gap-2">
            <Upload className="h-5 w-5" />
            Import Orders
          </DialogTitle>
          <DialogDescription>
            Upload a CSV file to import orders. Max file size: 10MB.
          </DialogDescription>
        </DialogHeader>

        <div className="py-4">
          <FileUploader
            value={files}
            onValueChange={setFiles}
            accept={{ 'text/csv': ['.csv'] }}
            maxFiles={1}
            maxSize={10 * 1024 * 1024}
            showPreview={false}
          />
        </div>

        <DialogFooter>
          <Button variant="outline" onClick={handleClose}>
            Cancel
          </Button>
          <Button onClick={handleImport} disabled={files.length === 0}>
            Import
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}
```

### Step 2: Update `orders-page-header.tsx`

```tsx
// apps/web/src/app/(main)/orders/orders-page-header.tsx
'use client';

import { useState } from 'react';
import { PageHeader } from '@/components/layout';
import { Button } from '@/components/ui/button';
import { Plus, Upload } from 'lucide-react';
import Link from 'next/link';
import { ImportOrdersDialog } from '@/features/orders/components/order-list';

export function OrdersPageHeader() {
  const [importDialogOpen, setImportDialogOpen] = useState(false);

  const handleImport = (files: File[]) => {
    // TODO: Implement API integration
    console.log('Files to import:', files);
  };

  return (
    <>
      <PageHeader
        title="Orders"
        description="Manage your print-on-demand orders"
        breadcrumbs={[{ label: 'Dashboard', href: '/' }, { label: 'Orders' }]}
        actions={
          <div className="flex gap-2">
            <Button variant="outline" onClick={() => setImportDialogOpen(true)}>
              <Upload className="mr-2 h-4 w-4" />
              Import
            </Button>
            <Button>
              <Plus className="mr-2 h-4 w-4" />
              New Order
            </Button>
          </div>
        }
      />
      <ImportOrdersDialog
        open={importDialogOpen}
        onOpenChange={setImportDialogOpen}
        onImport={handleImport}
      />
    </>
  );
}
```

### Step 3: Update exports

```tsx
// features/orders/components/order-list/index.ts
export { ImportOrdersDialog } from './import-orders-dialog';
```

## Todo List

- [x] Create `import-orders-dialog.tsx`
- [x] Update `orders-page-header.tsx` - remove Link, add dialog state
- [x] Update `index.ts` exports
- [x] Run TypeScript compilation check

## Success Criteria

- [x] Import button opens dialog (not navigates)
- [x] FileUploader accepts only .csv
- [x] Selected file displays correctly
- [x] Cancel resets and closes
- [x] Import button disabled until file selected
- [x] No TypeScript errors

## Code Review Notes

**Review Date:** 2026-01-25
**Score:** 8.5/10

**Issues Found:**
- Add file extension validation in `handleImport` (can bypass MIME type by renaming)
- Remove `console.log` before production
- Backend must validate file content

**Status:** ✅ Approved with required fixes before merge
