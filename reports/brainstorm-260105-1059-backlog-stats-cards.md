# Brainstorm: Design Backlog Stats Cards

**Date:** 2026-01-05
**Page:** `/design/backlog`
**User:** Manager/Team Lead

---

## Problem Statement

Trang Design Backlog hiện có 2 stats cards (Total Tasks, Urgent). User cần thêm insight về **workload** và **nguồn task** để hỗ trợ quyết định phân công.

---

## Requirements

- User chính: Manager/Team Lead
- Insight cần: Workload + Source breakdown
- Giữ gọn gàng, không quá nhiều cards

---

## Options Evaluated

| Option | Cards | Pros | Cons |
|--------|-------|------|------|
| A - Giữ nguyên | 2 | Đơn giản | Thiếu insight |
| B - Workload focus | 4 | Balance urgency + workload | ✓ Selected |
| C - Source breakdown | 4 | Biết nguồn gốc | Thiếu urgency |
| D - Hybrid | 5 | Đầy đủ | Nhiều cards |
| E - Minimal | 3 | Gọn | Mất granularity |

---

## Final Decision: Option B (Modified)

### 4 Stats Cards

| # | Card | Data Source | Icon | Description |
|---|------|-------------|------|-------------|
| 1 | **Total Tasks** | `tasks.length` | FileStack (blue) | Tổng số tasks trong backlog |
| 2 | **Urgent** | `filter(t => t.isUrgent)` | Flame (red) | Tasks cần xử lý gấp |
| 3 | **Redesign** | `filter(t => t.source === 'REDESIGN')` | RefreshCw (orange) | Tasks yêu cầu thiết kế lại |
| 4 | **Avg. Wait** | `avg(now - createdAt)` | Clock (gray) | Thời gian chờ TB (giờ) |

### Display Format

- **Avg. Wait:** Format giờ, e.g., `36h`, `12h`, `48h`
- **Layout:** Grid 4 columns on desktop, 2x2 on mobile

---

## Implementation Considerations

### Code Changes

File: `apps/web/src/features/design/components/backlog/backlog-stats-bar.tsx`

**Calculations needed:**
```typescript
// Redesign count
const redesignCount = tasks.filter(t => t.source === 'REDESIGN').length;

// Avg wait in hours
const avgWaitHours = tasks.length > 0
  ? Math.round(tasks.reduce((sum, t) => {
      const waitMs = Date.now() - new Date(t.createdAt).getTime();
      return sum + waitMs;
    }, 0) / tasks.length / (1000 * 60 * 60))
  : 0;
```

### Risks

| Risk | Mitigation |
|------|------------|
| `createdAt` không có timezone consistency | Ensure UTC storage |
| Avg. Wait = 0 nếu không có tasks | Hiển thị "-" thay vì 0 |

---

## Success Metrics

- Manager có thể identify workload bottleneck nhanh hơn
- Redesign tasks được prioritize đúng
- Avg. Wait giúp track service level

---

## Next Steps

1. Update `backlog-stats-bar.tsx` với 4 cards
2. Add icon imports (RefreshCw, Clock từ lucide-react)
3. Test responsive layout (4 cols → 2x2)

---

## Unresolved Questions

*None*
