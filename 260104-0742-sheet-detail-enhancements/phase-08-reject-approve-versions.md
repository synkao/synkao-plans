# Phase 08: Reject/Approve Design Versions

## Context

- Parent: [plan.md](plan.md)
- Dependencies: Phase 04 (DesignVersionsSection)
- Docs: [codebase-summary.md](../../docs/codebase-summary.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P2 |
| Effort | 1h |
| Implementation Status | Done |
| Review Status | Done |

**Objective:** Thêm Approve/Reject buttons cho SUBMITTED design versions với rejection reason dialog.

## Key Insights

1. Chỉ versions với status `SUBMITTED` mới cần action buttons
2. Reject cần dialog input reason (bắt buộc)
3. Approve có thể instant action hoặc simple confirm
4. Actions là demo only (mock, không persist)

## Requirements

### Approve Button

- Hiển thị cho versions status = SUBMITTED
- Click → instant change to APPROVED (local state)
- Optional: simple confirm toast

### Reject Button

- Hiển thị cho versions status = SUBMITTED
- Click → mở Dialog
- Dialog có textarea nhập rejection reason (required)
- Submit → change to REJECTED + save reason

### UI Layout

```
Version Card (SUBMITTED)
├── Thumbnail
├── Version info
└── Actions row ← NEW
    ├── [✓ Approve] (green)
    └── [✗ Reject]  (red outline)
```

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `drawer/design-versions-section.tsx` | Modify | Add action buttons |
| `drawer/reject-version-dialog.tsx` | Create | Dialog component |
| `drawer/index.ts` | Modify | Export new component |

## Implementation Steps

### Step 1: Create RejectVersionDialog

```typescript
// reject-version-dialog.tsx
interface RejectVersionDialogProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  onReject: (reason: string) => void;
  versionNumber: number;
}

export function RejectVersionDialog({ ... }) {
  const [reason, setReason] = useState('');

  const handleSubmit = () => {
    if (!reason.trim()) return;
    onReject(reason);
    onOpenChange(false);
  };

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Reject Version {versionNumber}</DialogTitle>
        </DialogHeader>
        <Textarea
          placeholder="Enter rejection reason..."
          value={reason}
          onChange={(e) => setReason(e.target.value)}
        />
        <DialogFooter>
          <Button variant="outline" onClick={() => onOpenChange(false)}>
            Cancel
          </Button>
          <Button
            variant="destructive"
            onClick={handleSubmit}
            disabled={!reason.trim()}
          >
            Reject
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}
```

### Step 2: Add local state to DesignVersionsSection

```typescript
// design-versions-section.tsx
const [localVersions, setLocalVersions] = useState(versions);
const [rejectDialogOpen, setRejectDialogOpen] = useState(false);
const [selectedVersion, setSelectedVersion] = useState<MockDesignVersion | null>(null);

const handleApprove = (versionId: string) => {
  setLocalVersions(prev =>
    prev.map(v => v.id === versionId ? { ...v, status: 'APPROVED' } : v)
  );
  toast.success('Version approved!');
};

const handleReject = (reason: string) => {
  if (!selectedVersion) return;
  setLocalVersions(prev =>
    prev.map(v => v.id === selectedVersion.id
      ? { ...v, status: 'REJECTED', rejectionReason: reason }
      : v
    )
  );
  setSelectedVersion(null);
  toast.success('Version rejected');
};
```

### Step 3: Add action buttons to version cards

```tsx
{version.status === 'SUBMITTED' && (
  <div className="flex gap-2 mt-2">
    <Button
      size="sm"
      variant="ghost"
      className="h-7 text-green-600 hover:bg-green-50"
      onClick={() => handleApprove(version.id)}
    >
      <CheckCircle2 className="h-3 w-3 mr-1" />
      Approve
    </Button>
    <Button
      size="sm"
      variant="ghost"
      className="h-7 text-red-600 hover:bg-red-50"
      onClick={() => {
        setSelectedVersion(version);
        setRejectDialogOpen(true);
      }}
    >
      <XCircle className="h-3 w-3 mr-1" />
      Reject
    </Button>
  </div>
)}
```

## Todo List

- [x] Create RejectVersionDialog component
- [x] Add local state management to DesignVersionsSection
- [x] Add Approve/Reject buttons for SUBMITTED versions
- [x] Export new component from index.ts
- [x] Add toast notifications (optional - skipped, not critical)
- [x] Verify TypeScript compiles
- [x] Test UI interactions

## Success Criteria

- [x] SUBMITTED versions show Approve/Reject buttons
- [x] Approve changes status to APPROVED instantly
- [x] Reject opens dialog for reason input
- [x] Reason is required (button disabled if empty)
- [x] After reject, version shows REJECTED + reason
- [x] TypeScript compiles without errors

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| State sync issues | Medium | Use local state, sync on drawer close |
| Toast not available | Low | Use console.log fallback or skip |

## Security Considerations

None - demo only with mock data.
