# Phase 1: UI Constants

## Context
- Parent: [plan.md](plan.md)
- Dependencies: None (standalone)

## Overview
| Field | Value |
|-------|-------|
| Date | 2026-01-02 |
| Priority | P2 |
| Effort | 30m |
| Status | pending |

Update central UI constants file that defines labels for status, priority, columns.

## Target File
`apps/web/src/features/design/lib/constants.ts`

## Changes Required

### KANBAN_COLUMNS
```typescript
// Before → After
{ id: 'TODO', name: 'Cần làm' }      → { id: 'TODO', name: 'To Do' }
{ id: 'IN_PROGRESS', name: 'Đang làm' } → { id: 'IN_PROGRESS', name: 'In Progress' }
{ id: 'REVIEW', name: 'Review' }     → (no change)
{ id: 'REVISION', name: 'Cần sửa' }  → { id: 'REVISION', name: 'Revision' }
{ id: 'DONE', name: 'Hoàn thành' }   → { id: 'DONE', name: 'Done' }
```

### PRIORITY_CONFIG
```typescript
URGENT: { label: 'Gấp' }       → { label: 'Urgent' }
HIGH: { label: 'Cao' }         → { label: 'High' }
NORMAL: { label: 'Bình thường' } → { label: 'Normal' }
LOW: { label: 'Thấp' }         → { label: 'Low' }
```

### STATUS_CONFIG
Same as KANBAN_COLUMNS labels.

### SOURCE_CONFIG
```typescript
NEW: { label: 'Mới' }          → { label: 'New' }
REDO: { label: 'Làm lại' }     → { label: 'Redo' }
COMPLAINT: { label: 'Khiếu nại' } → { label: 'Complaint' }
```

## Implementation Steps
- [ ] Read constants.ts
- [ ] Replace KANBAN_COLUMNS names
- [ ] Replace PRIORITY_CONFIG labels
- [ ] Replace STATUS_CONFIG labels
- [ ] Replace SOURCE_CONFIG labels

## Success Criteria
- [ ] All labels in English
- [ ] No Vietnamese characters in file
- [ ] TypeScript compiles without errors
