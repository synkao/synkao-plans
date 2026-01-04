# Phase 01: Types & Mock Data

## Context

- Parent: [plan.md](plan.md)
- Dependencies: None (foundation phase)
- Docs: [codebase-summary.md](../../docs/codebase-summary.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P1 (foundation) |
| Effort | 1h |
| Implementation Status | Pending |
| Review Status | Pending |

**Objective:** Add new types và mock data cho print specs, reference files, comments. Extend MockDesignVersion với thumbnail fields.

## Key Insights

1. Types file: `apps/web/src/mocks/types.ts` - centralized mock types
2. Mock data: `apps/web/src/mocks/data/` - separate files per entity
3. Pattern: UUID generator từ `factories/mock-factory.ts`
4. Images: picsum.photos for placeholder thumbnails

## Requirements

### New Types

```typescript
// MockReferenceFile - Tài liệu tham khảo từ admin/staff
interface MockReferenceFile {
  id: string;
  taskId: string;
  fileUrl: string;
  thumbnailUrl: string;
  fileName: string;
  uploadedById: string;
  createdAt: string;
}

// MockComment - Designer/reviewer conversation
interface MockComment {
  id: string;
  taskId: string;
  userId: string;
  content: string;
  designVersionId?: string; // optional version link
  createdAt: string;
}

// MockDesignVersion - Extended fields
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
  thumbnailUrl: string;
  fileName: string;
  fileSize: number;
  notes?: string;
  uploadedById?: string;
}

// PrintSpecs - flexible key-value
printSpecs: Record<string, string>;
```

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `apps/web/src/mocks/types.ts` | Modify | Add new interfaces |
| `apps/web/src/mocks/data/design-versions.data.ts` | Modify | Add thumbnail fields |
| `apps/web/src/mocks/data/reference-files.data.ts` | Create | New mock data |
| `apps/web/src/mocks/data/comments.data.ts` | Create | New mock data |
| `apps/web/src/mocks/data/index.ts` | Modify | Export new data |
| `apps/web/src/mocks/factories/mock-factory.ts` | Modify | Add UUID generators |

## Implementation Steps

### Step 1: Update types.ts

Add MockReferenceFile, MockComment interfaces. Extend MockDesignVersion và MockOrderItem.

### Step 2: Update mock-factory.ts

Add UUID generators:
```typescript
export const referenceFileUUID = (n: number) => `ref-${String(n).padStart(3, '0')}`;
export const commentUUID = (n: number) => `cmt-${String(n).padStart(3, '0')}`;
```

### Step 3: Update design-versions.data.ts

Add thumbnailUrl, fileName, fileSize fields sử dụng picsum.photos.

### Step 4: Create reference-files.data.ts

Generate 10-15 mock reference files cho random tasks.

### Step 5: Create comments.data.ts

Generate 20-30 mock comments với some linked to versions.

### Step 6: Update data/index.ts

Export new mock data files.

## Todo List

- [ ] Add MockReferenceFile interface
- [ ] Add MockComment interface
- [ ] Extend MockDesignVersion với thumbnail fields
- [ ] Add printSpecs to MockOrderItem
- [ ] Create reference-files.data.ts
- [ ] Create comments.data.ts
- [ ] Update design-versions.data.ts
- [ ] Update factories với new UUID generators
- [ ] Update index.ts exports

## Success Criteria

- [ ] TypeScript compiles without errors
- [ ] Mock data generates correctly
- [ ] Picsum thumbnails work
- [ ] Export paths valid

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Breaking existing types | High | Extend, không change existing fields |
| Mock data inconsistency | Low | Use same patterns as existing |

## Security Considerations

None - mock data only, no real user data.

## Next Steps

→ Phase 02: Print Specs Section
