# Brainstorm Report: Sheet Detail Enhancements

**Date:** 2026-01-03
**Context:** Phase 02 - Design Module
**Related Plan:** [phase-02-design-module.md](../260102-0716-synkao-mock-demo/phase-02-design-module.md)

---

## Problem Statement

Sheet detail (TaskDetailDrawer) cần được enhance với:
1. Bổ sung các fields còn thiếu
2. Thêm thumbnail cho design versions
3. Thêm form và upload area
4. Thêm comments cho designer/reviewer trao đổi

---

## Final Scope

### Enhanced Sheet Structure

```
TaskDetailDrawer (Sheet)
├── Header
│   ├── Task code, Product name, Customer
│   └── Quick Status Actions buttons ← NEW
│
├── TaskInfoSection (enhanced)
│   ├── Status, Priority, Source badges
│   ├── Assignee, Deadline (with warning), Workspace
│   ├── Order link (text clickable) ← NEW
│   └── Order Item link (text clickable) ← NEW
│
├── PrintSpecsSection ← NEW
│   ├── Key-value display (Record<string, string>)
│   └── Edit Modal form
│
├── ReferenceFilesSection ← NEW
│   ├── 3-column grid thumbnails
│   ├── Lightbox preview (shadcn Dialog, no zoom)
│   └── Upload area (demo UI only, not functional)
│
├── DesignNotesSection (existing)
│   └── Read-only from order item
│
├── DesignVersionsSection (enhanced)
│   ├── Thumbnails for each version ← NEW
│   ├── Lightbox preview on click ← NEW
│   └── Upload form (demo UI, optional notes) ← NEW
│
├── CommentsSection ← NEW
│   ├── Comment list (designer/reviewer conversation)
│   ├── Optional link to specific version
│   └── Simple text input form
│
└── TimelineSection (existing)
```

---

## New/Enhanced Types

### MockDesignVersion (Enhanced)

```typescript
interface MockDesignVersion {
  // Existing
  id: string;
  taskId: string;
  version: number;
  fileUrl: string;
  status: 'DRAFT' | 'SUBMITTED' | 'APPROVED' | 'REJECTED';
  rejectionReason?: string;
  createdAt: string;

  // NEW
  thumbnailUrl: string;      // Preview 150x150
  fileName: string;          // Original filename
  fileSize: number;          // Bytes
  notes?: string;            // Optional notes when submitting
  uploadedById?: string;     // Who uploaded
}
```

### MockReferenceFile (NEW)

```typescript
interface MockReferenceFile {
  id: string;
  taskId: string;
  fileUrl: string;
  thumbnailUrl: string;
  fileName: string;
  uploadedById: string;      // Admin/Staff only
  createdAt: string;
}
```

### MockComment (NEW)

```typescript
interface MockComment {
  id: string;
  taskId: string;
  userId: string;            // Comment author
  content: string;
  designVersionId?: string;  // Optional: comment on specific version
  createdAt: string;
}
```

### Print Specs (in MockOrderItem or MockTask)

```typescript
// Flexible key-value, imported from sales order, can be manually edited
printSpecs: Record<string, string>;

// Example:
{
  "Print Size": "A3",
  "DPI": "300",
  "Color Mode": "CMYK",
  "Material": "Cotton 250gsm",
  "Bleed": "3mm"
}
```

---

## UI Decisions

| Feature | Decision | Notes |
|---------|----------|-------|
| Print Specs UI | Inline display + Edit Modal | Key-value list, click Edit opens form modal |
| Reference Files Layout | 3-column grid | Responsive, thumbnails với hover |
| Lightbox Component | shadcn Dialog | No zoom (MVP), fullsize image display |
| Lightbox Actions | Navigation arrows + Download button | Arrow keys support |
| Upload Component | react-dropzone | Demo UI only, không functional |
| Upload Permission (Ref) | Admin/Staff only | Designer không upload reference |
| Version Notes | Optional field | Có thể add notes khi submit version |
| Comments | With optional version link | Dropdown select version to link |
| Comment Form | Simple text input | Input + Send button |
| Quick Status Actions | Below header | Buttons thay vì dropdown |
| Order/Item Links | Text clickable | Order code và item code clickable |

---

## Files to Create

### Components (NEW)

```
apps/web/src/features/design/components/drawer/
├── print-specs-section.tsx       # Key-value specs display + edit modal
├── reference-files-section.tsx   # 3-col grid + upload UI
├── comments-section.tsx          # Comment list + input form
├── image-lightbox.tsx            # shadcn Dialog lightbox
├── file-upload-zone.tsx          # react-dropzone wrapper (demo)
├── quick-status-actions.tsx      # Status change buttons
└── edit-specs-modal.tsx          # Modal form for specs editing
```

### Mock Data (NEW)

```
apps/web/src/mocks/data/
├── reference-files.data.ts       # Mock reference files
├── comments.data.ts              # Mock comments
└── (update) design-versions.data.ts  # Add thumbnailUrl, fileName, fileSize
```

---

## Files to Modify

### Types

```
apps/web/src/mocks/types.ts
- Add MockReferenceFile interface
- Add MockComment interface
- Update MockDesignVersion (thumbnailUrl, fileName, fileSize, notes)
- Add printSpecs to MockOrderItem or MockTask
```

### Components

```
apps/web/src/features/design/components/drawer/
├── task-detail-drawer.tsx        # Add new sections, reorder
├── task-info-section.tsx         # Add order/item links, deadline warning
└── design-versions-section.tsx   # Add thumbnails, lightbox trigger
```

---

## Libraries Required

```bash
pnpm add react-dropzone
```

---

## Implementation Notes

### Demo-Only Features
- Upload UI hiển thị nhưng không functional
- File selection shows preview nhưng không actual upload
- Mock data generates picsum thumbnails

### Quick Status Flow
```
TODO → [Start Work] → IN_PROGRESS → [Submit] → REVIEW → [Approve/Reject] → DONE/REVISION
```

### Deadline Warning Rules
- Due date < 2 days: Yellow warning badge
- Overdue: Red warning badge với "Overdue" text

### Comments Display Order
- Newest first hoặc oldest first (TBD during implementation)
- Grouped by version nếu có version link

---

## Success Criteria

- [ ] Print specs section hiển thị key-value từ order
- [ ] Edit modal cho specs với add/remove fields
- [ ] Reference files grid 3 columns với thumbnails
- [ ] Lightbox opens on thumbnail click
- [ ] Design versions có thumbnail preview
- [ ] Upload zone UI (demo only)
- [ ] Comments section với list và input form
- [ ] Optional version link trong comments
- [ ] Quick status action buttons below header
- [ ] Order/item text links clickable
- [ ] Deadline warning badges

---

## Next Steps

1. Update phase-02 plan với new scope
2. Implement in order:
   - Types & mock data first
   - Print specs section
   - Reference files section
   - Enhanced design versions (thumbnails + lightbox)
   - Comments section
   - Quick status actions
   - Upload UI (demo)
