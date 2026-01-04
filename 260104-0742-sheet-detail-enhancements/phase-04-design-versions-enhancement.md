# Phase 04: Design Versions Enhancement

## Context

- Parent: [plan.md](plan.md)
- Dependencies: Phase 01 (types), Phase 03 (ImageLightbox)
- Docs: [codebase-summary.md](../../docs/codebase-summary.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P2 |
| Effort | 1h |
| Implementation Status | Pending |
| Review Status | Pending |

**Objective:** Enhance DesignVersionsSection với thumbnails, lightbox preview, và upload form UI.

## Key Insights

1. Current: Version list without images
2. Target: Thumbnail per version, click to lightbox
3. Upload: Form với notes field, demo only
4. Reuse: ImageLightbox từ Phase 03

## Requirements

### Enhanced DesignVersionsSection

- Thumbnail (100x75) bên trái mỗi version
- Click thumbnail → open lightbox
- Upload new version button
- Version notes display (if exists)

### Version Upload Form

- File dropzone
- Optional notes textarea
- Submit button (demo only)
- Show preview before "submit"

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `drawer/design-versions-section.tsx` | Modify | Add thumbnails, lightbox |
| `task-detail-drawer.tsx` | Unchanged | Already includes section |

## Implementation Steps

### Step 1: Update version card layout

Add thumbnail on left side, keep existing info on right.

```tsx
<div className="flex gap-3">
  {/* Thumbnail */}
  <div className="w-24 h-18 rounded overflow-hidden cursor-pointer">
    <img src={version.thumbnailUrl} alt={`Version ${version.version}`} />
  </div>

  {/* Version info (existing) */}
  <div className="flex-1">
    ...
  </div>
</div>
```

### Step 2: Add lightbox integration

Reuse ImageLightbox từ Phase 03.

### Step 3: Add upload button + form

Simple form với file input và notes textarea.

### Step 4: Display notes

Show version.notes if exists.

## Todo List

- [ ] Add thumbnail to version card
- [ ] Integrate ImageLightbox
- [ ] Add upload button
- [ ] Create upload form UI
- [ ] Display version notes
- [ ] Test lightbox navigation

## Success Criteria

- [ ] Thumbnails display correctly
- [ ] Lightbox opens on click
- [ ] Upload form appears
- [ ] Notes displayed
- [ ] Styling consistent

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Layout break | Medium | Test các screen sizes |
| Lightbox reuse | Low | Share component từ Phase 03 |

## Security Considerations

None - demo UI only.

## Next Steps

→ Phase 05: Comments Section
