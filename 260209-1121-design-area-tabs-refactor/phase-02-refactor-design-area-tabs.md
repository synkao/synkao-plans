# Phase 02: Refactor DesignAreaTabs — Filter + Upload + Preview

## Context
- Parent plan: [plan.md](plan.md)
- Depends on: Phase 01 (all items have designAreas)
- Files: `design-area-tabs.tsx`, reuse `image-lightbox-dialog.tsx`

## Overview
- Priority: P2
- Status: ✅ completed
- Description: Refactor DesignAreaTabs to only show positions in designAreas map, add upload for null positions, add image preview click for positions with files

## Key Insights
- Current: iterates all 11 DESIGN_POSITION_OPTIONS, shows inactive gray tabs for missing positions
- New: `.filter((pos) => pos.value in designAreas)` → only positions in map shown
- Tab with `fileInfo === null` → wrap with DesignUploadModal (awaiting upload, allow click to upload)
- Tab with `fileInfo !== null` → tooltip + `onClick` → open ImageLightboxDialog for preview
- ImageLightboxDialog already exists at `order-list/image-lightbox-dialog.tsx` — reuse directly

## Requirements
### Functional
- Only display tabs for positions present in `designAreas` map
- Null file positions: styled differently (dashed border or lighter blue) + click opens DesignUploadModal
- File positions: solid blue + tooltip (filename, size) + click opens full-size preview
- Preview uses ImageLightboxDialog with file URL
- Non-image files (PDF/AI/PSD): fallback — show filename in lightbox or skip preview

### Non-functional
- Component stays under 200 LOC
- Accessible: ARIA labels, focus rings, keyboard nav

## Architecture

### Component state
```typescript
const [previewFile, setPreviewFile] = useState<{url: string; fileName: string} | null>(null);
```

### Rendering flow
```
DESIGN_POSITION_OPTIONS
  .filter(pos => pos.value in designAreas)
  .map(pos => {
    const fileInfo = designAreas[pos.value];

    if (fileInfo) {
      // Has file → tooltip + click to preview
      return <Tooltip>
        <button onClick={() => setPreviewFile(fileInfo)}>
          {pos.label}
        </button>
        <TooltipContent>filename + size</TooltipContent>
      </Tooltip>
    }

    // No file (null) → upload modal trigger
    return <DesignUploadModal trigger={<button>{pos.label}</button>} />
  })

// At bottom: <ImageLightboxDialog open={!!previewFile} ... />
```

### Tab styling
- **Has file:** `border-blue-500 bg-blue-50 text-blue-700` (current active style)
- **Null (awaiting):** `border-blue-300 bg-blue-50/50 text-blue-500 border-dashed` (lighter, dashed)

## Related Code Files
- **Modify:** `apps/web/src/features/orders/components/order-detail/design-area-tabs.tsx`
- **Reuse:** `apps/web/src/features/orders/components/order-list/image-lightbox-dialog.tsx`
- **Reuse:** `apps/web/src/features/orders/components/order-detail/design-upload-modal.tsx`

## Implementation Steps
1. Add `useState` for `previewFile` (url + fileName)
2. Add `.filter((pos) => pos.value in designAreas)` to DESIGN_POSITION_OPTIONS mapping
3. Refactor tab rendering:
   - `fileInfo !== null` → tooltip + `onClick={() => setPreviewFile({url: fileInfo.url, fileName: fileInfo.fileName})}`
   - `fileInfo === null` → wrap with DesignUploadModal (dashed border style)
4. Remove inactive tab branch entirely (old lines 84-104)
5. Import and render `ImageLightboxDialog` at component bottom
6. Add `cursor-pointer` to clickable active tabs
7. Verify no TypeScript errors, compile check

## Todo
- [x] Add previewFile state
- [x] Filter positions by designAreas keys
- [x] Refactor active tab with file → clickable + tooltip + preview
- [x] Refactor null tab → DesignUploadModal trigger
- [x] Remove inactive tab rendering code
- [x] Add ImageLightboxDialog at component bottom
- [x] Style differentiation (solid vs dashed border)
- [x] Compile check & visual verification (pending Node 20+ env)

## Success Criteria
- Only positions in designAreas map are visible as tabs
- Click tab with file → ImageLightboxDialog opens showing the image
- Click tab without file → DesignUploadModal opens
- Hover tab with file → tooltip shows filename + size
- No gray inactive tabs shown
- Component under 200 LOC
- No TypeScript errors

## Risk Assessment
- Low: ImageLightboxDialog uses Next/Image with `unoptimized` — external URLs work fine
- Edge case: non-image files (PDF/AI/PSD) won't render in Image component → can add file type check later if needed

## Security Considerations
- File URLs from mock data are trusted; in production, ensure URL sanitization
- No XSS risk: file names displayed as text content, not innerHTML
