# Brainstorm: Convert Vietnamese Text to English

## Problem Statement
Synkao app targets global market, current codebase contains Vietnamese text in:
- Mock data (user names, descriptions)
- UI constants (labels, status names)
- Component hardcoded strings

## Scope
- **In scope**: Code + mock data conversion VN → EN
- **Out of scope**: CLAUDE.md, README, docs/ (keep VN for internal communication)

## Affected Files (16 files)

### Mock Data (5 files)
| File | VN Content | EN Translation |
|------|-----------|----------------|
| `users.data.ts` | Tên VN | English names (John Smith, Jane Doe...) |
| `workspaces.data.ts` | "Workspace Chính" | "Main Workspace" |
| `timeline-events.data.ts` | Descriptions | Translate all descriptions |
| `design-versions.data.ts` | rejectionReasons | Translate rejection reasons |
| `user-menu.tsx` | mockUser | English user |

### UI Constants (`constants.ts`)
| Config | VN → EN |
|--------|---------|
| KANBAN_COLUMNS | Cần làm → To Do, Đang làm → In Progress, Cần sửa → Revision, Hoàn thành → Done |
| PRIORITY_CONFIG | Gấp → Urgent, Cao → High, Bình thường → Normal, Thấp → Low |
| STATUS_CONFIG | Same as KANBAN_COLUMNS |
| SOURCE_CONFIG | Mới → New, Làm lại → Redo, Khiếu nại → Complaint |

### Components (~8 files)
| File | VN → EN |
|------|---------|
| `task-info-section.tsx` | "Thông tin task" → "Task Info", "Chưa giao" → "Unassigned" |
| Other components | Need audit |

## Translation Mapping

### User Names
```
Trần Văn Admin → John Smith (Admin)
Nguyễn Thị Hoa → Jane Doe (Staff)
Lê Văn Dũng → Mike Johnson (Staff)
Phạm Minh Tuấn → Tom Wilson (Designer)
Hoàng Thị Lan → Emily Brown (Designer)
Vũ Đức Anh → David Lee (Designer)
Nguyễn Văn A → Alex Chen (Admin - user-menu)
```

### Timeline Event Descriptions
```
Task được tạo từ đơn hàng → Task created from order
Đã giao cho designer → Assigned to designer
Design v{n} đã được duyệt → Design v{n} approved
Cần chỉnh sửa → Needs revision
Chuyển sang {status} → Changed to {status}
```

### Rejection Reasons
```
Logo quá nhỏ, cần phóng to 150% → Logo too small, need to enlarge 150%
Màu sắc không đúng brand guideline → Colors don't match brand guidelines
Font chữ khó đọc trên nền tối → Font hard to read on dark background
Cần thêm shadow cho text → Need to add text shadow
Độ phân giải thấp, cần file 300dpi → Low resolution, need 300dpi file
```

## Implementation Approach
Simple find-and-replace approach:
1. Start with constants.ts (central config)
2. Update mock data files
3. Update component files
4. Verify no VN text remains using grep

## Risks & Considerations
- No i18n needed now (single language)
- Can add i18n later if multi-language required
- Email domains can stay @synkao.com

## Decision
✅ Proceed with direct replacement (no i18n overhead)
✅ Use English names for mock users
✅ Translate all UI labels and descriptions

## Next Steps
Create detailed implementation plan using `/plan`
