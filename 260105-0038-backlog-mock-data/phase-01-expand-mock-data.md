# Phase 01: Expand Mock Data

## Context
- Parent: [plan.md](./plan.md)
- Priority: P2
- Effort: 30m
- Status: Pending

## Overview

Mở rộng mock data để có ~40 tasks trong backlog.

## Requirements

- [ ] Thêm 5 orders mới (15→20)
- [ ] Order items tự động tăng (45→60)
- [ ] Điều chỉnh task distribution: TODO=40
- [ ] Đảm bảo variety: priorities, sources, designers

## Related Code Files

### Modify
| File | Changes |
|------|---------|
| `mocks/data/orders.data.ts` | Thêm 5 orders |
| `mocks/data/tasks.data.ts` | Thay đổi statusDistribution |

## Implementation Steps

### Step 1: Update orders.data.ts
Thêm 5 orders mới với customer names khác nhau.

### Step 2: Update tasks.data.ts
```ts
// NEW distribution: 40 TODO + 20 others = 60 total
const statusDistribution: DesignTaskStatusType[] = [
  ...Array(40).fill(DesignTaskStatus.TODO),
  ...Array(8).fill(DesignTaskStatus.IN_PROGRESS),
  ...Array(6).fill(DesignTaskStatus.REVIEW),
  ...Array(3).fill(DesignTaskStatus.REVISION),
  ...Array(3).fill(DesignTaskStatus.APPROVED),
];
```

### Step 3: Verify
- Build pass
- Backlog page hiển thị 40 tasks
- Pagination hoạt động đúng

## Todo List

- [ ] Thêm 5 orders vào orders.data.ts
- [ ] Update statusDistribution trong tasks.data.ts
- [ ] Verify build và UI

## Success Criteria

- [ ] Backlog có ~40 tasks
- [ ] Pagination perPage=20 hiển thị 2 pages
- [ ] Build không lỗi
