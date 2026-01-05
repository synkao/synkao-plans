# Brainstorm: Đơn giản hóa Priority → Urgent Flag

**Date:** 2026-01-05
**Status:** Đã thống nhất

---

## 1. Problem Statement

Hệ thống hiện có 4 mức priority: `LOW | NORMAL | HIGH | URGENT`

**Vấn đề:**
- User ít khi dùng đủ 4 mức → cognitive overhead không cần thiết
- MVP scope cần đơn giản hóa
- Chỉ cần biết "có cần làm gấp không"

---

## 2. Giải pháp đề xuất

### ✅ Recommended: Boolean `isUrgent`

```typescript
// Thay thế
type Priority = "LOW" | "NORMAL" | "HIGH" | "URGENT";

// Bằng
isUrgent: boolean; // true = urgent, false = normal
```

**Lý do chọn:**
- **KISS**: Đơn giản nhất, không cần enum
- **YAGNI**: Không cần 4 mức khi chỉ dùng 1
- **UI/UX**: Chỉ render icon/badge khi `isUrgent === true`, ẩn hoàn toàn khi false
- **DX**: Toggle đơn giản, không cần dropdown/select

---

## 3. Các file cần sửa

| File | Thay đổi |
|------|----------|
| `src/types/status.ts` | Xóa `Priority` type |
| `src/lib/constants.ts` | Xóa `PRIORITY_COLORS`, thêm `URGENT_COLOR` |
| `src/features/design/lib/constants.ts` | Đơn giản hóa config |
| `src/features/design/components/priority-badge.tsx` | Đổi thành `UrgentBadge` hoặc xóa |
| `src/mocks/types.ts` | Đổi `priority: PriorityType` → `isUrgent: boolean` |
| `src/mocks/data/tasks.data.ts` | Update mock data |
| `src/mocks/handlers/workspaces.ts` | Update bulk action endpoint |

---

## 4. UI Implementation

```tsx
// Chỉ render khi urgent
{task.isUrgent && (
  <span className="text-red-500 text-xs font-medium">
    🔥 Urgent
  </span>
)}

// Hoặc icon đơn giản
{task.isUrgent && <AlertCircle className="h-4 w-4 text-red-500" />}
```

---

## 5. Migration

Nếu có data cũ (Low/Normal/High/Urgent):
```typescript
// Mapping: chỉ URGENT → true, còn lại → false
isUrgent = oldPriority === "URGENT"
```

---

## 6. Trade-offs

| Pros | Cons |
|------|------|
| Đơn giản, ít code | Không rollback dễ nếu cần 4 mức |
| UI gọn gàng | - |
| Toggle nhanh | - |
| Phù hợp MVP | - |

---

## 7. Next Steps

1. Update types & constants
2. Update mock data structure
3. Đổi PriorityBadge → UrgentBadge (hoặc inline component)
4. Update bulk action handlers
5. Test UI

---

## Kết luận

**Đề xuất:** Thay `Priority` enum bằng `isUrgent: boolean`
**Impact:** Low - chủ yếu rename/simplify
**Effort:** ~1-2 giờ implementation
