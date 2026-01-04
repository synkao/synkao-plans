# Phase 03: Reference Files Section

## Context

- Parent: [plan.md](plan.md)
- Dependencies: Phase 01 (types), react-dropzone
- Docs: [codebase-summary.md](../../docs/codebase-summary.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P2 |
| Effort | 1.5h |
| Implementation Status | Pending |
| Review Status | Pending |

**Objective:** Create ReferenceFilesSection với 3-column grid thumbnails, lightbox preview, và upload UI (demo only).

## Key Insights

1. Grid: 3 columns responsive
2. Lightbox: shadcn Dialog fullsize image
3. Upload: react-dropzone wrapper, demo only
4. Permission: Upload visible for admin/staff only (check role)

## Requirements

### ReferenceFilesSection Component

- 3-column grid của thumbnails
- Click thumbnail → open lightbox
- Show file name on hover
- Upload button/zone at end

### ImageLightbox Component (Shared)

- shadcn Dialog với fullsize image
- Navigation arrows (prev/next)
- Download button
- Keyboard support (arrow keys, Esc)
- No zoom (MVP)

### FileUploadZone Component

- react-dropzone integration
- Drag-drop visual feedback
- Accept image types only
- Demo: show preview, không actual upload
- Permission check: only admin/staff

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `drawer/reference-files-section.tsx` | Create | Grid + upload |
| `drawer/image-lightbox.tsx` | Create | Fullsize preview |
| `drawer/file-upload-zone.tsx` | Create | Dropzone wrapper |
| `task-detail-drawer.tsx` | Modify | Add section |

## Implementation Steps

### Step 1: Install react-dropzone

```bash
pnpm add react-dropzone
```

### Step 2: Create image-lightbox.tsx

Reusable lightbox cho cả reference files và design versions.

```tsx
interface ImageLightboxProps {
  images: { url: string; title?: string }[];
  currentIndex: number;
  open: boolean;
  onOpenChange: (open: boolean) => void;
  onNavigate: (index: number) => void;
}
```

### Step 3: Create file-upload-zone.tsx

```tsx
interface FileUploadZoneProps {
  onFilesSelected: (files: File[]) => void;
  disabled?: boolean;
  accept?: string[];
}
```

### Step 4: Create reference-files-section.tsx

Combine grid, lightbox trigger, upload zone.

### Step 5: Integrate into drawer

Add after PrintSpecsSection.

## Todo List

- [ ] Install react-dropzone
- [ ] Create image-lightbox.tsx
- [ ] Create file-upload-zone.tsx
- [ ] Create reference-files-section.tsx
- [ ] Implement 3-column grid
- [ ] Add lightbox integration
- [ ] Style upload zone
- [ ] Add to drawer
- [ ] Test keyboard navigation

## Success Criteria

- [ ] Grid displays thumbnails correctly
- [ ] Lightbox opens on click
- [ ] Navigation arrows work
- [ ] Upload zone styled
- [ ] Drag feedback visible
- [ ] Permission check works

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Large images slow | Medium | Use thumbnailUrl |
| react-dropzone version | Low | Use stable version |

## Security Considerations

- File upload demo only, không actual upload
- Role check cho upload visibility

## Next Steps

→ Phase 04: Design Versions Enhancement
