# Code Review Report: Phase 03 - Design Areas Components

**Date:** 2026-02-05
**Reviewer:** code-reviewer
**Phase:** Phase 03 - Design Areas Components
**Plan:** `/Users/taquanglinh/Documents/synkao/plans/260205-2018-order-detail-ui-updates/phase-03-design-areas-components.md`

---

## Code Review Summary

### Scope
- **Files Reviewed:**
  - `apps/web/src/features/orders/lib/constants.ts` (Modified)
  - `apps/web/src/features/orders/components/order-detail/design-upload-modal.tsx` (NEW)
  - `apps/web/src/features/orders/components/order-detail/order-item-card.tsx` (Modified)
  - `apps/web/src/features/orders/components/order-detail/index.ts` (Modified)
- **Lines of Code Analyzed:** ~500 lines
- **Review Focus:** Recent changes for Phase 03 implementation
- **TypeScript Check:** PASSED - No compilation errors

---

## Overall Assessment

**Score: 7/10**

Implementation follows plan requirements closely. Code is readable, well-structured with proper TypeScript typing. Major concerns: missing error handling details, incomplete file validation, and simulated upload needs backend integration. Task button logic correctly implements conditional rendering based on design/task state.

---

## Critical Issues

### 1. File Upload Validation - Client-side Only
**Location:** `design-upload-modal.tsx:39-56`

**Issue:** File validation relies solely on FileUploader component prop validation. No explicit file type/size checks before upload attempt.

**Risk:** Malformed files or wrong MIME types could bypass validation if FileUploader has bugs.

**Recommendation:**
```typescript
const handleUpload = async () => {
  const file = files[0];
  if (!file) return;

  // Explicit validation before upload
  if (file.size > DESIGN_UPLOAD_CONFIG.maxSize) {
    toast.error(`File too large. Max size: ${formatFileSize(DESIGN_UPLOAD_CONFIG.maxSize)}`);
    return;
  }

  const allowedTypes = Object.keys(DESIGN_UPLOAD_CONFIG.accept);
  if (!allowedTypes.includes(file.type)) {
    toast.error('Invalid file type. Supported: PNG, JPG, PDF, AI, PSD');
    return;
  }

  setIsUploading(true);
  try {
    // TODO: Replace with actual API call when backend is ready
    await new Promise((resolve) => setTimeout(resolve, 1000));

    toast.success(`Design uploaded for ${itemName}`);
    onUploadComplete?.(file);
    setOpen(false);
    setFiles([]);
  } catch (error) {
    const message = error instanceof Error ? error.message : 'Failed to upload design';
    toast.error(message);
  } finally {
    setIsUploading(false);
  }
};
```

---

## High Priority Findings

### 1. Error Handling - Generic Catch Block
**Location:** `design-upload-modal.tsx:52-53`

**Issue:** Generic error handler with no error type checking or specific error messages.

**Impact:** Users won't know why upload failed (network, file type, size, server error).

**Fix:** See recommendation above (Critical Issue #1).

---

### 2. Unused Props - itemId Not Used
**Location:** `design-upload-modal.tsx:29-34`

**Issue:** `itemId` prop passed but never used in component. Likely needed for API call.

**Impact:** LOW - Will be needed when backend integration happens.

**Recommendation:** Add comment explaining future use or pass to API call.

```typescript
// itemId will be used for API endpoint: POST /api/orders/items/${itemId}/design
```

---

### 3. Task Button Accessibility - Missing aria-label for Status Badge
**Location:** `order-item-card.tsx:116-129`

**Issue:** Status badge (Case 3) has no aria-label or screen reader text.

**Impact:** Screen reader users won't know task status.

**Recommendation:**
```typescript
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
      role="status"
      aria-label={`Task status: ${statusConfig.label}`}
    >
      <span className="h-1.5 w-1.5 rounded-full bg-current" aria-hidden="true" />
      {statusConfig.label}
    </span>
  );
}
```

---

## Medium Priority Improvements

### 1. DESIGN_UPLOAD_CONFIG - Hard to Maintain MIME Types
**Location:** `constants.ts:373-382`

**Issue:** MIME type for AI files (`application/postscript`) may not work consistently across browsers. PSD MIME type (`image/vnd.adobe.photoshop`) rarely recognized.

**Recommendation:**
```typescript
export const DESIGN_UPLOAD_CONFIG = {
  maxSize: 50 * 1024 * 1024, // 50MB
  accept: {
    'image/png': ['.png'],
    'image/jpeg': ['.jpg', '.jpeg'],
    'application/pdf': ['.pdf'],
    'application/postscript': ['.ai'],
    'application/illustrator': ['.ai'], // Alternative MIME
    'image/vnd.adobe.photoshop': ['.psd'],
    'image/x-photoshop': ['.psd'], // Alternative MIME
    'application/photoshop': ['.psd'], // Alternative MIME
  },
  // Alternative: Use file extension validation as fallback
  allowedExtensions: ['.png', '.jpg', '.jpeg', '.pdf', '.ai', '.psd'],
} as const;
```

---

### 2. File Preview - No Preview for PDF/AI/PSD
**Location:** `design-upload-modal.tsx:74-86`

**Issue:** FileUploader `showPreview={true}` likely only works for images. PDF/AI/PSD won't show preview.

**Impact:** Poor UX for non-image files.

**Recommendation:** Add file type indicator or custom preview logic for documents.

---

### 3. Task Button - Disabled State Unclear
**Location:** `order-item-card.tsx:100-112`

**Issue:** Gray disabled button says "Create Task" but doesn't explain why it's disabled.

**Recommendation:**
```typescript
// Case 2: Has design, no task -> Create Task (gray, disabled with tooltip)
if (hasDesign && !hasTask) {
  return (
    <Tooltip>
      <TooltipTrigger asChild>
        <span>
          <Button
            variant="outline"
            size="sm"
            disabled
            className="text-gray-400 cursor-not-allowed"
            aria-label={`Create task for ${item.productName} (disabled - design already uploaded)`}
          >
            <Plus className="mr-1 h-3 w-3" />
            Create Task
          </Button>
        </span>
      </TooltipTrigger>
      <TooltipContent>
        Cannot create task after design is uploaded
      </TooltipContent>
    </Tooltip>
  );
}
```

---

### 4. Empty Design Upload Trigger - Not Keyboard Accessible
**Location:** `order-item-card.tsx:233-249`

**Issue:** Using `<button>` element correctly, but missing keyboard navigation feedback.

**Recommendation:** Add focus styles:
```typescript
<button
  className="mt-2 w-full border-2 border-dashed border-gray-300 rounded-lg p-4 text-center hover:border-gray-400 hover:bg-gray-50 focus:border-blue-500 focus:ring-2 focus:ring-blue-200 transition-colors"
  aria-label={`Upload design for ${item.productName}`}
>
```

---

## Low Priority Suggestions

### 1. Constants - Add JSDoc Comments
**Location:** `constants.ts:349-382`

**Suggestion:** Add JSDoc comments for better IDE autocomplete and documentation.

```typescript
/**
 * Design position options for task modal dropdown
 * 11 position options for design placement on items
 * @example
 * <Select options={DESIGN_POSITION_OPTIONS} />
 */
export const DESIGN_POSITION_OPTIONS = [
  // ...
];
```

---

### 2. File Size Formatting - Duplicate Logic
**Location:** `order-item-card.tsx:36-40` and `file-uploader.tsx:49-55`

**Issue:** Two implementations of `formatFileSize` function.

**Recommendation:** Extract to shared utility:
```typescript
// lib/utils.ts
export function formatFileSize(bytes: number): string {
  if (bytes === 0) return '0 Bytes';
  const k = 1024;
  const sizes = ['Bytes', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return `${(bytes / Math.pow(k, i)).toFixed(1)} ${sizes[i]}`;
}
```

---

### 3. Modal Dialog - Add Description for Screen Readers
**Location:** `design-upload-modal.tsx:69-72`

**Suggestion:**
```typescript
<DialogHeader>
  <DialogTitle>Upload Design</DialogTitle>
  <DialogDescription>
    Upload a design file for {itemName}. Supported formats: PNG, JPG, PDF, AI, PSD (max 50MB)
  </DialogDescription>
</DialogHeader>
```

---

## Positive Observations

1. **Type Safety:** All props properly typed with TypeScript interfaces
2. **Accessibility:** Good use of aria-labels on buttons (except status badge)
3. **Clean Separation:** Upload modal is separate component, reusable
4. **Conditional Logic:** renderTaskButton() clearly implements 3-state logic
5. **Constants Usage:** Good use of DESIGN_UPLOAD_CONFIG and TASK_STATUS_CONFIG
6. **Error Boundary:** Toast notifications for user feedback
7. **Loading State:** isUploading state prevents double uploads
8. **File Cleanup:** Dialog closes, files reset on cancel/success

---

## Security Audit

### Passed Checks
- ✅ No hardcoded credentials
- ✅ No sensitive data in logs
- ✅ File size limit enforced (50MB)
- ✅ File type validation via accept prop
- ✅ No eval() or dangerous code execution
- ✅ No XSS vulnerabilities (React escapes by default)

### Warnings
- ⚠️ File MIME type validation client-side only (need server-side validation)
- ⚠️ TODO: Backend API not implemented (simulated upload)
- ⚠️ AI/PSD MIME types may not work consistently

---

## Performance Analysis

### Passed Checks
- ✅ No unnecessary re-renders (useState properly scoped)
- ✅ No memory leaks (cleanup in handleOpenChange)
- ✅ Images optimized with Next.js Image component
- ✅ No blocking operations

### Observations
- File preview rendering handled by FileUploader component (assumed optimized)
- Modal only renders when triggered (lazy loading via Dialog)

---

## Plan Completeness Verification

### Todo List Status (from plan.md)
- ✅ Add DESIGN_POSITION_OPTIONS to constants.ts
- ✅ Add DESIGN_UPLOAD_CONFIG to constants.ts
- ✅ Create design-upload-modal.tsx component
- ✅ Update OrderItemCardProps with taskStatus prop
- ✅ Implement renderTaskButton logic in order-item-card.tsx
- ✅ Add empty design upload trigger area
- ✅ Update index.ts exports
- ❌ Run TypeScript compiler to verify (DONE - passed, but should document)

### Success Criteria Status
- ✅ Upload modal opens when clicking empty design area
- ✅ Modal shows drag & drop with file preview
- ✅ Upload button disabled until file selected
- ✅ Task button shows blue/enabled when no design
- ✅ Task button shows gray/disabled when has design but no task
- ✅ Task button shows status badge when has task
- ✅ No TypeScript errors

**All tasks completed. All success criteria met.**

---

## Recommended Actions (Priority Order)

1. **CRITICAL:** Add explicit file validation in handleUpload (type + size checks)
2. **HIGH:** Improve error handling with specific error messages
3. **HIGH:** Add aria-label to status badge for accessibility
4. **MEDIUM:** Add alternative MIME types for AI/PSD files
5. **MEDIUM:** Add tooltip explaining disabled "Create Task" button
6. **MEDIUM:** Add focus styles to upload trigger button
7. **LOW:** Extract formatFileSize to shared utility
8. **LOW:** Add JSDoc comments to constants
9. **LOW:** Add DialogDescription to upload modal

---

## Metrics

- **Type Coverage:** 100% (all exports typed)
- **Test Coverage:** Not available (no tests written yet)
- **Linting Issues:** ESLint not configured
- **Compilation Errors:** 0
- **TODO Comments:** 1 (line 45 - backend integration)

---

## Unresolved Questions

1. What happens when backend API is implemented? Need API endpoint spec.
2. Should file preview support PDF/AI/PSD files or just show filename?
3. Should there be a progress bar for large file uploads (50MB)?
4. What error codes will backend return for file upload failures?
5. Should uploaded files be stored temporarily before task creation?
6. Is there a maximum number of designs per order item?

---

## Next Steps

1. Address critical file validation issue before merging
2. Fix accessibility issues (aria-label, focus styles)
3. Document backend API contract in plan file
4. Consider writing integration tests for upload flow
5. Update plan.md with "completed" status for Phase 03
