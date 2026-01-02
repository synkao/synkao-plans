# Phase 2: Mock Data

## Context
- Parent: [plan.md](plan.md)
- Dependencies: Phase 1 (constants should be updated first)

## Overview
| Field | Value |
|-------|-------|
| Date | 2026-01-02 |
| Priority | P2 |
| Effort | 45m |
| Status | pending |

Update all mock data files with English text.

## Target Files

### 1. users.data.ts
Path: `apps/web/src/mocks/data/users.data.ts`

```typescript
// Name mapping
'Trần Văn Admin'  → 'John Smith'
'Nguyễn Thị Hoa'  → 'Jane Doe'
'Lê Văn Dũng'     → 'Mike Johnson'
'Phạm Minh Tuấn'  → 'Tom Wilson'
'Hoàng Thị Lan'   → 'Emily Brown'
'Vũ Đức Anh'      → 'David Lee'
```

### 2. workspaces.data.ts
Path: `apps/web/src/mocks/data/workspaces.data.ts`

```typescript
'Workspace Chính' → 'Main Workspace'
```

### 3. timeline-events.data.ts
Path: `apps/web/src/mocks/data/timeline-events.data.ts`

```typescript
'Task được tạo từ đơn hàng' → 'Task created from order'
'Đã giao cho designer'      → 'Assigned to designer'
'Design v${n} đã được duyệt' → 'Design v${n} approved'
'Cần chỉnh sửa'             → 'Needs revision'
'Chuyển sang ${status}'      → 'Changed to ${status}'
```

### 4. design-versions.data.ts
Path: `apps/web/src/mocks/data/design-versions.data.ts`

```typescript
// rejectionReasons array
'Logo quá nhỏ, cần phóng to 150%'     → 'Logo too small, need to enlarge 150%'
'Màu sắc không đúng brand guideline'  → 'Colors don\'t match brand guidelines'
'Font chữ khó đọc trên nền tối'       → 'Font hard to read on dark background'
'Cần thêm shadow cho text'            → 'Need to add text shadow'
'Độ phân giải thấp, cần file 300dpi'  → 'Low resolution, need 300dpi file'
```

## Implementation Steps
- [ ] Update users.data.ts with English names
- [ ] Update workspaces.data.ts
- [ ] Update timeline-events.data.ts descriptions
- [ ] Update design-versions.data.ts rejection reasons

## Success Criteria
- [ ] All mock data in English
- [ ] No Vietnamese characters in mock files
- [ ] Data structure unchanged
