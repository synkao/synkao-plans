# Phase 4: Dashboard + Settings Module

## Context Links
- **Parent Plan:** [plan.md](./plan.md)
- **Dependencies:** [Phase 1](./phase-01-foundation-layout.md), [Phase 2](./phase-02-design-module.md)
- **Design System:** [synkao-design-system.md](../../docs/design/synkao-design-system.md)
- **UI Spec:** Section 5 (Dashboard), Section 8 (Settings)

---

## Overview

| Property | Value |
|----------|-------|
| Priority | P2 |
| Status | pending |
| Effort | 1 day |
| Description | Build Dashboard with stats, pending tasks, recent orders. Build Settings pages for Team and Workspaces. |

---

## Key Insights

1. **Dashboard** - 4 stat cards + pending tasks list + recent orders + activity feed
2. **Stat Cards** - Glass effect, trend indicator (up/down)
3. **Team Management** - Table with user list, invite modal (static)
4. **Workspace Settings** - Grid of workspace cards, create/edit modal
5. **User Data** - 6 users in mockUsers (1 Admin, 2 Staff, 3 Designers)

---

## Requirements

### Functional
- F1: Dashboard with 4 stat cards (orders, pending design, ready, completed)
- F2: Pending tasks list (top 5 urgent/high priority)
- F3: Recent orders list (last 5)
- F4: Activity feed (mock events)
- F5: Team page with user table
- F6: Invite member modal (UI only)
- F7: Workspaces page with cards
- F8: Create/Edit workspace modal (UI only)

### Non-Functional
- NF1: Stat cards have glass effect
- NF2: Cards clickable → navigate to filtered list
- NF3: Modals don't actually save (demo)

---

## Architecture

### Component Tree - Dashboard
```
DashboardPage
├── PageHeader (title, date filter)
├── StatsGrid (4 cards)
│   ├── StatCard (Đơn mới)
│   ├── StatCard (Chờ thiết kế)
│   ├── StatCard (Sẵn sàng giao)
│   └── StatCard (Hoàn thành)
├── Grid (2 columns)
│   ├── Column 1
│   │   ├── PendingTasksCard
│   │   └── RecentOrdersCard
│   └── Column 2
│       └── ActivityFeedCard
```

### Component Tree - Team Settings
```
TeamPage
├── PageHeader (title, invite button)
├── FilterBar (search, role, status)
└── TeamTable
    └── UserRow[]
        └── Actions (edit, deactivate)

InviteMemberModal
├── EmailInput
├── RoleSelector
└── WorkspaceSelector (for Designer)
```

### Component Tree - Workspace Settings
```
WorkspacesPage
├── PageHeader (title, create button)
└── WorkspacesGrid
    ├── WorkspaceSettingsCard[]
    └── CreateWorkspaceCard (+ button)

WorkspaceModal
├── NameInput
├── IconSelector
├── ColorSelector
└── DescriptionInput
```

---

## Related Code Files

### Create - Pages
| File Path | Description |
|-----------|-------------|
| `apps/web/src/app/(main)/page.tsx` | Dashboard page - stat cards + lists |
| `apps/web/src/app/(main)/settings/team/page.tsx` | Team management page |
| `apps/web/src/app/(main)/settings/workspaces/page.tsx` | Workspace settings page |

### Create - Dashboard Components
| File Path | Description |
|-----------|-------------|
| `apps/web/src/features/dashboard/components/stat-card.tsx` | Glass effect KPI card |
| `apps/web/src/features/dashboard/components/stats-grid.tsx` | 4-card grid container |
| `apps/web/src/features/dashboard/components/trend-badge.tsx` | Trend indicator (up/down) |
| `apps/web/src/features/dashboard/components/pending-tasks-card.tsx` | Top 5 urgent tasks |
| `apps/web/src/features/dashboard/components/recent-orders-card.tsx` | Last 5 orders |
| `apps/web/src/features/dashboard/components/activity-feed.tsx` | Activity timeline |
| `apps/web/src/features/dashboard/components/index.ts` | Barrel exports |
| `apps/web/src/features/dashboard/lib/constants.ts` | Stat configs |
| `apps/web/src/features/dashboard/index.ts` | Feature barrel exports |

### Create - Settings/Team Components
| File Path | Description |
|-----------|-------------|
| `apps/web/src/features/settings/components/team/team-table.tsx` | User table với actions |
| `apps/web/src/features/settings/components/team/user-row.tsx` | Single user row |
| `apps/web/src/features/settings/components/team/role-badge.tsx` | Role badge với color |
| `apps/web/src/features/settings/components/team/status-badge.tsx` | User status badge |
| `apps/web/src/features/settings/components/team/invite-modal.tsx` | Invite member form |
| `apps/web/src/features/settings/components/team/index.ts` | Team barrel exports |

### Create - Settings/Workspaces Components
| File Path | Description |
|-----------|-------------|
| `apps/web/src/features/settings/components/workspaces/workspace-settings-card.tsx` | Workspace card với actions |
| `apps/web/src/features/settings/components/workspaces/workspace-modal.tsx` | Create/edit form |
| `apps/web/src/features/settings/components/workspaces/create-workspace-card.tsx` | Add new card |
| `apps/web/src/features/settings/components/workspaces/icon-picker.tsx` | Icon selector |
| `apps/web/src/features/settings/components/workspaces/color-picker.tsx` | Color selector |
| `apps/web/src/features/settings/components/workspaces/index.ts` | Workspaces barrel exports |

### Create - Settings Shared
| File Path | Description |
|-----------|-------------|
| `apps/web/src/features/settings/components/index.ts` | Settings barrel exports |
| `apps/web/src/features/settings/lib/constants.ts` | Role, status configs |
| `apps/web/src/features/settings/index.ts` | Feature barrel exports |

### Create - Mock Data
| File Path | Description |
|-----------|-------------|
| `apps/web/src/mocks/data/activity-events.data.ts` | Dashboard activity events |

### Modify
| File Path | Change Description |
|-----------|-------------------|
| `apps/web/src/lib/navigation.ts` | Add Settings routes with children |
| `apps/web/src/mocks/index.ts` | Export activity events |

### Delete
| File Path | Reason |
|-----------|--------|
| `apps/web/src/app/page.tsx` | Move to (main)/page.tsx |

---

## Implementation Steps

### 1. Create StatCard Component (45 min)
Glass effect card with:
- Icon
- Label
- Value (large number)
- Trend indicator (arrow + percentage)

```tsx
<Card className="bg-white/70 backdrop-blur-sm">
  <CardContent className="p-6">
    <div className="flex justify-between items-start">
      <div className="p-2 rounded-lg bg-blue-50">
        <Icon className="h-5 w-5 text-blue-500" />
      </div>
      <TrendBadge trend={trend} />
    </div>
    <p className="text-sm text-muted mt-4">{label}</p>
    <p className="text-3xl font-bold">{value}</p>
  </CardContent>
</Card>
```

### 2. Create StatsGrid (20 min)
```tsx
const stats = [
  { label: 'Đơn mới hôm nay', value: 12, trend: '+20%', icon: Package },
  { label: 'Chờ thiết kế', value: 8, trend: '-5%', icon: Palette },
  { label: 'Sẵn sàng giao', value: 5, trend: '0%', icon: Truck },
  { label: 'Hoàn thành tuần này', value: 45, trend: '+15%', icon: CheckCircle },
];

<div className="grid grid-cols-4 gap-4">
  {stats.map(s => <StatCard key={s.label} {...s} />)}
</div>
```

### 3. Create PendingTasksCard (30 min)
List of 5 urgent/high priority tasks:
```tsx
<Card>
  <CardHeader>
    <CardTitle>Design tasks cần xử lý</CardTitle>
  </CardHeader>
  <CardContent>
    <div className="space-y-3">
      {tasks.slice(0, 5).map(task => (
        <div key={task.id} className="flex items-center justify-between">
          <div>
            <p className="font-medium">{task.taskCode}</p>
            <p className="text-sm text-muted">{task.productName}</p>
          </div>
          <PriorityBadge priority={task.priority} />
        </div>
      ))}
    </div>
  </CardContent>
  <CardFooter>
    <Link href="/design/backlog">Xem tất cả →</Link>
  </CardFooter>
</Card>
```

### 4. Create RecentOrdersCard (30 min)
List of 5 recent orders

### 5. Create ActivityFeed (45 min)
Timeline of mock events:
```typescript
const mockEvents = [
  { time: '10 phút trước', text: 'Task #123 được duyệt bởi Staff A' },
  { time: '30 phút trước', text: 'Đơn #ORD-005 import từ WooCommerce' },
  { time: '1 giờ trước', text: 'Designer B upload v2 cho task #456' },
];
```

### 6. Create Dashboard Page (30 min)
```tsx
export default function DashboardPage() {
  const urgentTasks = mockTasks
    .filter(t => t.priority === 'URGENT' || t.priority === 'HIGH')
    .slice(0, 5);
  const recentOrders = mockOrders.slice(0, 5);

  return (
    <>
      <PageHeader title="Dashboard" />
      <StatsGrid />
      <div className="grid grid-cols-3 gap-6 mt-6">
        <div className="col-span-2 space-y-6">
          <PendingTasksCard tasks={urgentTasks} />
          <RecentOrdersCard orders={recentOrders} />
        </div>
        <ActivityFeed />
      </div>
    </>
  );
}
```

### 7. Create Settings Constants (15 min)
```typescript
export const USER_ROLES = {
  ADMIN: { label: 'Admin', color: 'violet' },
  STAFF: { label: 'Staff', color: 'blue' },
  DESIGNER: { label: 'Designer', color: 'amber' },
};

export const USER_STATUS = {
  ACTIVE: { label: 'Active', color: 'emerald' },
  INVITED: { label: 'Invited', color: 'amber' },
  INACTIVE: { label: 'Inactive', color: 'gray' },
};
```

### 8. Create TeamTable (45 min)
```tsx
<Table>
  <TableHeader>
    <TableRow>
      <TableHead>Avatar</TableHead>
      <TableHead>Tên</TableHead>
      <TableHead>Email</TableHead>
      <TableHead>Vai trò</TableHead>
      <TableHead>Trạng thái</TableHead>
      <TableHead></TableHead>
    </TableRow>
  </TableHeader>
  <TableBody>
    {mockUsers.map(user => <UserRow key={user.id} user={user} />)}
  </TableBody>
</Table>
```

### 9. Create InviteModal (45 min)
```tsx
<Dialog>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Mời thành viên mới</DialogTitle>
    </DialogHeader>
    <div className="space-y-4">
      <Input placeholder="Email" />
      <RadioGroup>
        <RadioGroupItem value="STAFF" label="Staff" />
        <RadioGroupItem value="DESIGNER" label="Designer" />
      </RadioGroup>
      {isDesigner && <WorkspaceCheckboxes />}
    </div>
    <DialogFooter>
      <Button>Gửi lời mời</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

### 10. Create Team Page (20 min)
```tsx
export default function TeamPage() {
  return (
    <>
      <PageHeader title="Quản lý Team" actions={<InviteButton />} />
      <TeamTable users={mockUsers} />
    </>
  );
}
```

### 11. Create WorkspaceSettingsCard (30 min)
```tsx
<Card className="p-4">
  <div className="flex items-center gap-3">
    <div className="p-3 rounded-lg bg-blue-50">
      <Icon className="h-6 w-6" />
    </div>
    <div>
      <h3 className="font-medium">{workspace.name}</h3>
      <p className="text-sm text-muted">{taskCount} tasks • {memberCount} designers</p>
    </div>
  </div>
  <div className="flex gap-2 mt-4">
    <Button variant="outline" size="sm">Edit</Button>
    <Button variant="destructive" size="sm">Delete</Button>
  </div>
</Card>
```

### 12. Create WorkspaceModal (30 min)
Name, icon picker, color picker, description

### 13. Create Workspaces Page (20 min)
```tsx
export default function WorkspacesPage() {
  return (
    <>
      <PageHeader title="Workspaces" actions={<CreateButton />} />
      <div className="grid grid-cols-3 gap-4">
        {mockWorkspaces.map(w => (
          <WorkspaceSettingsCard key={w.id} workspace={w} />
        ))}
        <CreateWorkspaceCard />
      </div>
    </>
  );
}
```

---

## Todo List

### Dashboard Feature
- [ ] Create `apps/web/src/features/dashboard/lib/constants.ts`
- [ ] Create `apps/web/src/features/dashboard/components/stat-card.tsx`
- [ ] Create `apps/web/src/features/dashboard/components/stats-grid.tsx`
- [ ] Create `apps/web/src/features/dashboard/components/trend-badge.tsx`
- [ ] Create `apps/web/src/features/dashboard/components/pending-tasks-card.tsx`
- [ ] Create `apps/web/src/features/dashboard/components/recent-orders-card.tsx`
- [ ] Create `apps/web/src/features/dashboard/components/activity-feed.tsx`
- [ ] Create `apps/web/src/features/dashboard/components/index.ts`
- [ ] Create `apps/web/src/features/dashboard/index.ts`

### Settings/Team Feature
- [ ] Create `apps/web/src/features/settings/lib/constants.ts`
- [ ] Create `apps/web/src/features/settings/components/team/team-table.tsx`
- [ ] Create `apps/web/src/features/settings/components/team/user-row.tsx`
- [ ] Create `apps/web/src/features/settings/components/team/role-badge.tsx`
- [ ] Create `apps/web/src/features/settings/components/team/status-badge.tsx`
- [ ] Create `apps/web/src/features/settings/components/team/invite-modal.tsx`
- [ ] Create `apps/web/src/features/settings/components/team/index.ts`

### Settings/Workspaces Feature
- [ ] Create `apps/web/src/features/settings/components/workspaces/workspace-settings-card.tsx`
- [ ] Create `apps/web/src/features/settings/components/workspaces/workspace-modal.tsx`
- [ ] Create `apps/web/src/features/settings/components/workspaces/create-workspace-card.tsx`
- [ ] Create `apps/web/src/features/settings/components/workspaces/icon-picker.tsx`
- [ ] Create `apps/web/src/features/settings/components/workspaces/color-picker.tsx`
- [ ] Create `apps/web/src/features/settings/components/workspaces/index.ts`

### Barrel Exports
- [ ] Create `apps/web/src/features/settings/components/index.ts`
- [ ] Create `apps/web/src/features/settings/index.ts`

### Mock Data
- [ ] Create `apps/web/src/mocks/data/activity-events.data.ts`

### Pages
- [ ] Create `apps/web/src/app/(main)/page.tsx` (Dashboard)
- [ ] Create `apps/web/src/app/(main)/settings/team/page.tsx`
- [ ] Create `apps/web/src/app/(main)/settings/workspaces/page.tsx`
- [ ] Delete old `apps/web/src/app/page.tsx`

### Integration
- [ ] Update `apps/web/src/lib/navigation.ts`
- [ ] Update `apps/web/src/mocks/index.ts`

### Testing
- [ ] Test dashboard stat cards với glass effect
- [ ] Test pending tasks card shows top 5
- [ ] Test recent orders card shows last 5
- [ ] Test activity feed renders
- [ ] Test team table shows 6 users
- [ ] Test invite modal opens/closes
- [ ] Test workspace cards render
- [ ] Test workspace modal opens/closes

---

## Success Criteria

- [ ] Dashboard shows 4 stat cards with glass effect
- [ ] Stats use mock data counts
- [ ] Pending tasks shows top 5
- [ ] Recent orders shows last 5
- [ ] Activity feed shows mock events
- [ ] Team table shows 6 users
- [ ] Invite modal opens
- [ ] Workspaces shows 2 cards
- [ ] Create/Edit modal opens

---

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Activity data missing | Low | Create simple mock array |
| Modal forms incomplete | Low | Static demo, no validation needed |
| Stats calculation | Low | Hardcode values from mock counts |

---

## Next Steps

After completion, proceed to [Phase 5: Auth + Polish + Deploy](./phase-05-auth-polish-deploy.md)
