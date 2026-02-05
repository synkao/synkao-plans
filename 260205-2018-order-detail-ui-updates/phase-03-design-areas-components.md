# Phase 03: Design Areas Components

## Context
- [Codebase Analysis](./reports/codebase-analysis.md)
- [Phase 01](./phase-01-header-and-status-dropdown.md) and [Phase 02](./phase-02-sidebar-cards-updates.md) should be completed first

## Overview
- **Priority:** P1
- **Status:** ✓ COMPLETED (All fixes verified)
- **Effort:** 2h
- **Code Review:** [Initial Report](./reports/code-reviewer-260205-2300-phase03-design-areas.md)
- **Re-Review:** [Final Report](./reports/code-reviewer-260205-2308-phase03-rereviewed.md)
- **Initial Score:** 7/10 → **Final Score: 9/10** ⬆️
- **Completed:** 2026-02-05 23:09
- **Review Date:** 2026-02-05

## Key Insights
- OrderItemCard has existing action buttons for design workflow
- Need new upload modal for empty design tabs
- Task modal needs position dropdown (11 options)
- Task button logic is conditional on design/task state

## Requirements

### Functional

#### 1. Design Upload Modal (NEW)
- Trigger: Click on empty design tab area
- Features:
  - Drag & drop zone
  - File preview (thumbnail)
  - Filename + size display
  - Supported formats: PNG, JPG, PDF, AI, PSD
  - Max file size: 50MB
  - Upload button (disabled until file selected)

#### 2. Task Modal Position Dropdown
- Replace filename dropdown with position/angle dropdown
- 11 position options:
  - Front, Back, Left Sleeve, Right Sleeve
  - Pocket, Collar, Full Body
  - Top, Bottom, Left, Right
- 1 file per row layout

#### 3. Task Button Logic (NEW)
| Design State | Task State | Button Display |
|-------------|-----------|----------------|
| No design | - | "+ Create Task" (blue, enabled) |
| Has design | No task | "+ Create Task" (gray, DISABLED) |
| Has design | Has task | "* {Status}" badge |

## Architecture

### Upload Modal Structure
```
DesignUploadModal
├── DialogTrigger (empty design area)
├── DialogContent
│   ├── DialogHeader: "Upload Design"
│   ├── FileUploader (drag & drop)
│   │   ├── Drop zone
│   │   └── File preview (when selected)
│   └── DialogFooter
│       ├── Cancel button
│       └── Upload button (disabled until file)
```

### Position Options Constant
```typescript
export const DESIGN_POSITION_OPTIONS = [
  { value: 'FRONT', label: 'Front' },
  { value: 'BACK', label: 'Back' },
  { value: 'LEFT_SLEEVE', label: 'Left Sleeve' },
  { value: 'RIGHT_SLEEVE', label: 'Right Sleeve' },
  { value: 'POCKET', label: 'Pocket' },
  { value: 'COLLAR', label: 'Collar' },
  { value: 'FULL_BODY', label: 'Full Body' },
  { value: 'TOP', label: 'Top' },
  { value: 'BOTTOM', label: 'Bottom' },
  { value: 'LEFT', label: 'Left' },
  { value: 'RIGHT', label: 'Right' },
] as const;
```

## Related Code Files

### Create
| File | Purpose |
|------|---------|
| `design-upload-modal.tsx` | File upload dialog for designs |

### Modify
| File | Changes |
|------|---------|
| `order-item-card.tsx` | Task button logic, integrate upload modal |
| `constants.ts` | Add DESIGN_POSITION_OPTIONS |

## Implementation Steps

### Step 1: Add Position Options (constants.ts)
```typescript
// Add to: features/orders/lib/constants.ts

export const DESIGN_POSITION_OPTIONS = [
  { value: 'FRONT', label: 'Front' },
  { value: 'BACK', label: 'Back' },
  { value: 'LEFT_SLEEVE', label: 'Left Sleeve' },
  { value: 'RIGHT_SLEEVE', label: 'Right Sleeve' },
  { value: 'POCKET', label: 'Pocket' },
  { value: 'COLLAR', label: 'Collar' },
  { value: 'FULL_BODY', label: 'Full Body' },
  { value: 'TOP', label: 'Top' },
  { value: 'BOTTOM', label: 'Bottom' },
  { value: 'LEFT', label: 'Left' },
  { value: 'RIGHT', label: 'Right' },
] as const;

export type DesignPositionType = typeof DESIGN_POSITION_OPTIONS[number]['value'];

// File upload config
export const DESIGN_UPLOAD_CONFIG = {
  maxSize: 50 * 1024 * 1024, // 50MB
  accept: {
    'image/png': ['.png'],
    'image/jpeg': ['.jpg', '.jpeg'],
    'application/pdf': ['.pdf'],
    'application/postscript': ['.ai'],
    'image/vnd.adobe.photoshop': ['.psd'],
  },
} as const;
```

### Step 2: Create DesignUploadModal
```typescript
// Create: features/orders/components/order-detail/design-upload-modal.tsx

'use client';

import { useState } from 'react';
import { toast } from 'sonner';
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogFooter,
  DialogTrigger,
} from '@/components/ui/dialog';
import { Button } from '@/components/ui/button';
import { FileUploader } from '@/components/ui/file-uploader';
import { DESIGN_UPLOAD_CONFIG } from '../../lib/constants';

export interface DesignUploadModalProps {
  itemId: string;
  itemName: string;
  trigger: React.ReactNode;
  onUploadComplete?: (file: File) => void;
}

export function DesignUploadModal({
  itemId,
  itemName,
  trigger,
  onUploadComplete,
}: DesignUploadModalProps) {
  const [open, setOpen] = useState(false);
  const [files, setFiles] = useState<File[]>([]);
  const [isUploading, setIsUploading] = useState(false);

  const handleUpload = async () => {
    if (files.length === 0) return;

    setIsUploading(true);
    try {
      // Simulate upload (replace with actual API call)
      await new Promise((resolve) => setTimeout(resolve, 1000));

      toast.success(`Design uploaded for ${itemName}`);
      onUploadComplete?.(files[0]);
      setOpen(false);
      setFiles([]);
    } catch (error) {
      toast.error('Failed to upload design');
    } finally {
      setIsUploading(false);
    }
  };

  const handleOpenChange = (newOpen: boolean) => {
    setOpen(newOpen);
    if (!newOpen) {
      setFiles([]);
    }
  };

  return (
    <Dialog open={open} onOpenChange={handleOpenChange}>
      <DialogTrigger asChild>{trigger}</DialogTrigger>
      <DialogContent className="sm:max-w-md">
        <DialogHeader>
          <DialogTitle>Upload Design</DialogTitle>
        </DialogHeader>

        <div className="py-4">
          <FileUploader
            value={files}
            onValueChange={setFiles}
            accept={DESIGN_UPLOAD_CONFIG.accept}
            maxFiles={1}
            maxSize={DESIGN_UPLOAD_CONFIG.maxSize}
            showPreview={true}
          />
          <p className="mt-2 text-xs text-gray-500">
            Supported: PNG, JPG, PDF, AI, PSD (Max 50MB)
          </p>
        </div>

        <DialogFooter>
          <Button variant="outline" onClick={() => setOpen(false)}>
            Cancel
          </Button>
          <Button
            onClick={handleUpload}
            disabled={files.length === 0 || isUploading}
          >
            {isUploading ? 'Uploading...' : 'Upload'}
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}
```

### Step 3: Update OrderItemCard Task Button Logic
```typescript
// Modify: features/orders/components/order-detail/order-item-card.tsx

// Add imports
import { DesignUploadModal } from './design-upload-modal';
import { TASK_STATUS_CONFIG } from '../../lib/constants';
import type { DesignTaskStatusType } from '../../types';

// Add props for task state
export interface OrderItemCardProps {
  item: MockOrderItem;
  designerName?: string;
  taskStatus?: DesignTaskStatusType | null; // NEW: task status
  onViewDesign?: (item: MockOrderItem) => void;
  onDownloadDesign?: (item: MockOrderItem) => void;
  onCreateTask?: (item: MockOrderItem) => void;
  onUploadDesign?: (item: MockOrderItem) => void;
  onReplaceDesign?: (item: MockOrderItem) => void;
}

// In the component, update the task button logic:

// Replace existing task button section with:
const renderTaskButton = () => {
  const hasDesign = item.designStatus && item.designStatus !== 'NOT_REQUIRED';
  const hasTask = !!taskStatus;

  // Case 1: No design -> Create Task (blue, enabled)
  if (!hasDesign) {
    return (
      <Button
        variant="default"
        size="sm"
        onClick={() => onCreateTask?.(item)}
        aria-label={`Create task for ${item.productName}`}
      >
        <Plus className="mr-1 h-3 w-3" />
        Create Task
      </Button>
    );
  }

  // Case 2: Has design, no task -> Create Task (gray, disabled)
  if (hasDesign && !hasTask) {
    return (
      <Button
        variant="outline"
        size="sm"
        disabled
        className="text-gray-400 cursor-not-allowed"
        aria-label={`Create task for ${item.productName} (disabled)`}
      >
        <Plus className="mr-1 h-3 w-3" />
        Create Task
      </Button>
    );
  }

  // Case 3: Has task -> Show status badge
  if (hasTask && taskStatus) {
    const statusConfig = TASK_STATUS_CONFIG[taskStatus];
    return (
      <span
        className={cn(
          'inline-flex items-center gap-1 px-2 py-1 rounded-full text-xs font-medium',
          statusConfig.bgClass,
          statusConfig.textClass
        )}
      >
        <span className="h-1.5 w-1.5 rounded-full bg-current" />
        {statusConfig.label}
      </span>
    );
  }

  return null;
};

// In the Actions section, replace the existing Create Task button with:
{renderTaskButton()}
```

### Step 4: Add Upload Trigger to Empty Design Area
```typescript
// In order-item-card.tsx, add upload modal for empty design state:

// After the thumbnail section, add clickable empty design area:
{!hasDesignFile && hasDesign && (
  <DesignUploadModal
    itemId={item.id}
    itemName={item.productName}
    trigger={
      <button
        className="mt-2 w-full border-2 border-dashed border-gray-300 rounded-lg p-4 text-center hover:border-gray-400 hover:bg-gray-50 transition-colors"
        aria-label={`Upload design for ${item.productName}`}
      >
        <Upload className="h-6 w-6 mx-auto text-gray-400 mb-1" />
        <span className="text-sm text-gray-500">Click to upload design</span>
      </button>
    }
    onUploadComplete={(file) => {
      toast.success(`Design ${file.name} uploaded`);
    }}
  />
)}
```

### Step 5: Update Index Exports
```typescript
// Modify: features/orders/components/order-detail/index.ts

export { DesignUploadModal } from './design-upload-modal';
export type { DesignUploadModalProps } from './design-upload-modal';
```

## Todo List
- [x] Add DESIGN_POSITION_OPTIONS to constants.ts
- [x] Add DESIGN_UPLOAD_CONFIG to constants.ts
- [x] Create design-upload-modal.tsx component
- [x] Update OrderItemCardProps with taskStatus prop
- [x] Implement renderTaskButton logic in order-item-card.tsx
- [x] Add empty design upload trigger area
- [x] Update index.ts exports
- [x] Run TypeScript compiler to verify

## Code Review Findings

### Initial Review (2026-02-05)
**Score: 7/10** - Implementation complete with recommendations

Found 7 issues requiring fixes:
1. Missing explicit file validation (CRITICAL)
2. Weak error handling (HIGH)
3. Missing aria-label on status badge (HIGH)
4. Limited MIME type support (WARNING)
5. Disabled button needs tooltip (SUGGESTION)
6. Missing DialogDescription (SUGGESTION)
7. Focus styles missing (LOW)

See [initial review report](./reports/code-reviewer-260205-2300-phase03-design-areas.md) for details.

### Re-Review After Fixes (2026-02-05)
**Score: 9/10** ✓ All fixes verified and approved

All 7 issues resolved:
✓ File validation (type + size) implemented with fallback checking
✓ Error handling improved with specific messages
✓ Aria-label and role="status" added to badge
✓ Expanded MIME types with browser fallbacks (AI/PSD)
✓ Tooltip added to disabled Create Task button
✓ DialogDescription added for accessibility
✓ Keyboard navigation improved

See [re-review report](./reports/code-reviewer-260205-2308-phase03-rereviewed.md) for detailed verification.

## Success Criteria
- [x] Upload modal opens when clicking empty design area
- [x] Modal shows drag & drop with file preview
- [x] Upload button disabled until file selected
- [x] Task button shows blue/enabled when no design
- [x] Task button shows gray/disabled when has design but no task
- [x] Task button shows status badge when has task
- [x] No TypeScript errors
- [x] File validation (type + size) in place
- [x] Proper error handling with user feedback
- [x] Accessibility compliance (ARIA, semantic HTML)
- [x] Browser compatibility for AI/PSD files

**All success criteria met ✓**
**Re-review verification complete ✓**

## Risk Assessment
| Risk | Impact | Mitigation |
|------|--------|------------|
| FileUploader accept format | Medium | Test with all file types |
| Task status prop missing | Low | Default to null, render Create Task |
| Large file handling | Medium | Show loading state, error handling |

## Security Considerations
- File type validation via accept prop
- File size limit enforced (50MB)
- No actual file upload (UI-only, toast simulation)

## Next Steps
After all phases complete:
1. Run full TypeScript check: `pnpm tsc --noEmit`
2. Run dev server and test all changes
3. Create PR for review
