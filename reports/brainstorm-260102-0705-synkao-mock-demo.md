# Brainstorm Report: SynKao Mock Demo

**Date:** 2025-01-02
**Type:** UI/UX Validation Demo
**Status:** Ready for Implementation

---

## 1. Problem Statement

Cần tạo mock demo cho SynKao - hệ thống quản lý đơn hàng Print-on-Demand để:
- Validate UI/UX với stakeholders trước khi code production
- Demo full app skeleton với tất cả screens từ UI spec
- Deploy lên cloud để stakeholders truy cập và review

## 2. Requirements Summary

| Aspect | Decision |
|--------|----------|
| Mục đích | Validate UI/UX với stakeholders |
| Scope | Full app skeleton (12 pages) |
| Interactivity | Static screens, click navigation |
| Approach | Next.js Static + shadcn/ui |
| Responsive | Desktop only (1280px+) |
| Deployment | Docker + Coolify (existing setup) |
| Visual | Glass morphism effect |
| Mock Data | English names, 10-20 items/list |
| Timeline | 1 tuần+ |

## 3. Evaluated Approaches

### Option A: Next.js Static Pages ✅ (Selected)
**Pros:**
- Match production tech stack (Next.js + shadcn/ui + Tailwind)
- Components reusable cho production
- Navigation hoạt động thật
- Deploy qua Docker + Coolify (đã setup sẵn)
- Type-safe với TypeScript

**Cons:**
- Setup ban đầu cần thời gian
- Cần kiến thức React/Next.js

### Option B: Figma Prototype ❌
**Pros:**
- Nhanh tạo screens
- Dễ share
- Có click-through prototype

**Cons:**
- Không code-based, phải làm lại
- Không match production stack
- Khó mở rộng

### Option C: HTML/Tailwind Static ❌
**Pros:**
- Nhẹ nhất
- Nhanh nhất setup

**Cons:**
- Không component-based
- Không reusable
- Khó maintain

## 4. Final Solution

### 4.1 Architecture

```
apps/web/
├── src/
│   ├── app/
│   │   ├── (auth)/                    # Auth group (no sidebar)
│   │   │   ├── login/page.tsx
│   │   │   └── forgot-password/page.tsx
│   │   │
│   │   ├── (main)/                    # Main app (with sidebar)
│   │   │   ├── layout.tsx             # GlassSidebar + AppHeader
│   │   │   ├── dashboard/page.tsx
│   │   │   ├── orders/
│   │   │   │   ├── page.tsx           # Orders list
│   │   │   │   ├── [id]/page.tsx      # Order detail
│   │   │   │   └── import/page.tsx    # Import wizard
│   │   │   ├── design/
│   │   │   │   ├── backlog/page.tsx
│   │   │   │   └── workspace/
│   │   │   │       ├── page.tsx       # Workspace list
│   │   │   │       └── [id]/page.tsx  # Kanban board
│   │   │   └── settings/
│   │   │       ├── team/page.tsx
│   │   │       └── workspaces/page.tsx
│   │   │
│   │   └── page.tsx                   # Redirect to dashboard
│   │
│   ├── components/
│   │   ├── layout/
│   │   │   ├── glass-sidebar.tsx
│   │   │   ├── app-header.tsx
│   │   │   └── page-header.tsx
│   │   ├── dashboard/
│   │   │   └── stat-card.tsx
│   │   ├── orders/
│   │   │   ├── orders-table.tsx
│   │   │   └── order-item-row.tsx
│   │   ├── design/
│   │   │   ├── kanban-board.tsx
│   │   │   ├── kanban-column.tsx
│   │   │   ├── task-card.tsx
│   │   │   └── task-drawer.tsx
│   │   └── ui/                        # shadcn components
│   │
│   └── lib/
│       └── mock-data/
│           ├── orders.ts
│           ├── tasks.ts
│           └── users.ts
```

### 4.2 Pages to Build (Priority Order)

**Phase 1: Foundation + Design Module (Priority)**
1. Layout components (GlassSidebar, AppHeader)
2. `/design/workspace/:id` - Kanban Board (core feature)
3. Task Detail Drawer
4. `/design/backlog` - Task backlog

**Phase 2: Orders Module**
5. `/orders` - Orders list với expandable rows
6. `/orders/:id` - Order detail + timeline
7. `/orders/import` - 3-step wizard

**Phase 3: Dashboard + Settings**
8. `/dashboard` - Stat cards + pending tasks
9. `/settings/team` - Team management
10. `/settings/workspaces` - Workspace config

**Phase 4: Auth + Polish**
11. `/login` - Glass card login
12. `/forgot-password` - Email reset
13. Common patterns (loading, empty states, toasts)

### 4.3 Component Library

| Category | Components |
|----------|------------|
| Layout | GlassSidebar, AppHeader, PageHeader |
| Dashboard | StatCard, PendingTasksList, RecentOrdersList |
| Orders | OrdersTable, OrderItemRow, ImportWizard, OrderTimeline |
| Design | KanbanBoard, KanbanColumn, TaskCard, TaskDrawer, BacklogTable |
| Settings | TeamTable, WorkspaceCard, InviteModal |
| UI (shadcn) | Button, Input, Select, Badge, Table, Dialog, Drawer, Toast |

### 4.4 Visual Specifications

**Glass Morphism Sidebar:**
```css
background: rgba(255, 255, 255, 0.6);
backdrop-filter: blur(12px);
border: 1px solid rgba(226, 232, 240, 0.8);
```

**Color Palette:**
- Primary: `#3B82F6` (Blue 500)
- Success: `#10B981` (Green 500)
- Warning: `#F59E0B` (Amber 500)
- Error: `#EF4444` (Red 500)
- Background: `#F8FAFC` (Slate 50)

**Priority Badges:**
- URGENT: Red `#EF4444`
- HIGH: Amber `#F59E0B`
- NORMAL: Blue `#3B82F6`
- LOW: Gray `#6B7280`

### 4.5 Mock Data Structure

```typescript
// 10-20 orders với English names
const orders = [
  {
    id: "ORD-2024-001",
    customer: { name: "John Smith", email: "john@example.com" },
    items: [
      { name: "Custom T-Shirt", sku: "TSHIRT-001", quantity: 2, designStatus: "APPROVED" },
      { name: "Personalized Mug", sku: "MUG-001", quantity: 1, designStatus: "IN_PROGRESS" }
    ],
    status: "IN_DESIGN",
    createdAt: "2024-12-20T14:30:00Z"
  },
  // ... 15-20 more orders
];

// 20+ design tasks
const tasks = [
  {
    id: "TASK-001",
    orderNumber: "ORD-2024-001",
    itemName: "Custom T-Shirt",
    customer: "John Smith",
    priority: "URGENT",
    status: "TODO",
    assignee: { name: "Designer Mike", avatar: "..." },
    createdAt: "2024-12-20T14:30:00Z"
  },
  // ... more tasks
];
```

## 5. Implementation Considerations

### 5.1 Technical Decisions
- **Routing:** Next.js App Router với route groups
- **Styling:** Tailwind CSS + shadcn/ui components
- **State:** Không cần global state (static demo)
- **Navigation:** Link components, no client-side routing logic
- **Mock Data:** TypeScript constants, import trực tiếp

### 5.2 Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Kanban drag-drop không hoạt động | Static cards only, visual demo |
| Form submissions không có response | Hardcode success states |
| Stakeholder muốn interactive features | Clarify scope trước: static demo only |

### 5.3 Out of Scope
- Real API integration
- Authentication/Authorization
- Form validation logic
- Drag-and-drop functionality
- Real-time updates
- Mobile responsive

## 6. Success Metrics

- [ ] Tất cả 12 pages render đúng theo wireframe
- [ ] Navigation hoạt động giữa các pages
- [ ] Glass morphism sidebar đúng spec
- [ ] Mock data hiển thị realistic
- [ ] Deploy thành công qua Docker + Coolify
- [ ] Stakeholders có thể truy cập và review

## 7. Effort Estimation

| Phase | Tasks | Days |
|-------|-------|------|
| Foundation | Project setup, Layout components | 1.5 |
| Design Module | Kanban, Backlog, Task Drawer | 2 |
| Orders Module | List, Detail, Import wizard | 2 |
| Dashboard | Stats, pending tasks | 0.5 |
| Settings | Team, Workspaces | 0.5 |
| Auth | Login, Forgot Password | 0.5 |
| Polish + Deploy | Common patterns, Docker build | 1 |
| **Total** | | **~8 ngày** |

## 8. Next Steps

1. Tạo implementation plan chi tiết với file-level tasks
2. Setup shadcn/ui trong project
3. Build layout components trước
4. Implement Design Module (priority)
5. Continue với Orders, Dashboard, Settings
6. Build Docker image và deploy qua Coolify

---

*Report generated: 2025-01-02*
