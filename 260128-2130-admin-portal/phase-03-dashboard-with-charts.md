---
phase: 3
title: "Dashboard with Charts"
status: pending
effort: 2h
depends_on: [1, 2]
---

# Phase 3: Dashboard with Charts

## Context Links

- [Phase 1](./phase-01-setup-navigation-and-guards.md) - Recharts installation
- [Phase 2](./phase-02-layout-and-header-components.md) - Admin layout
- [Card component](../../apps/web/src/components/ui/card.tsx)
- [GlassCard](../../apps/web/src/components/ui/glass-card.tsx)

## Overview

- **Priority**: Medium
- **Status**: Pending
- **Description**: Implement admin dashboard with stats cards (Total Users, Active Spaces, Revenue, Growth %) and Recharts visualizations (Revenue trend Area chart, User growth Bar chart)

## Key Insights

- Recharts is a React wrapper for D3, easy to use
- Use GlassCard for consistent styling with main app
- Mock data for now (no backend integration)
- Responsive grid: 1 col mobile, 2 col tablet, 4 col desktop for stats

## Requirements

### Functional
- 4 stats cards: Total Users, Active Spaces, Revenue, Growth %
- Revenue chart: Area/Line chart showing monthly trend
- Users chart: Bar chart showing monthly user growth
- Cards show icon, title, value, and change indicator

### Non-functional
- Mobile-first responsive design
- Charts resize properly on different screens
- Use project color tokens for chart colors

## Architecture

```
(admin)/page.tsx (Dashboard)
├── StatsCards (grid of 4)
│   ├── Users card (Users icon, count, +X% change)
│   ├── Spaces card (Boxes icon, count, +X% change)
│   ├── Revenue card (DollarSign icon, $amount, +X% change)
│   └── Growth card (TrendingUp icon, percentage, arrow)
└── Charts (2-column grid)
    ├── RevenueChart (Area chart)
    └── UsersChart (Bar chart)

features/admin/components/
├── stats-cards.tsx
└── charts/
    ├── revenue-chart.tsx
    └── users-chart.tsx
```

## Related Code Files

### Files to Create
- `apps/web/src/features/admin/components/stats-cards.tsx`
- `apps/web/src/features/admin/components/charts/revenue-chart.tsx`
- `apps/web/src/features/admin/components/charts/users-chart.tsx`
- `apps/web/src/features/admin/components/charts/index.ts`
- `apps/web/src/features/admin/components/index.ts`
- `apps/web/src/features/admin/index.ts`

### Files to Modify
- `apps/web/src/app/(admin)/page.tsx` - Replace placeholder with dashboard

## Implementation Steps

### Step 1: Create Stats Cards Component

Create `apps/web/src/features/admin/components/stats-cards.tsx`:

```tsx
import { Users, Boxes, DollarSign, TrendingUp, TrendingDown } from 'lucide-react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { cn } from '@/lib/utils';

interface StatCardProps {
  title: string;
  value: string;
  change: number;
  icon: React.ReactNode;
}

function StatCard({ title, value, change, icon }: StatCardProps) {
  const isPositive = change >= 0;

  return (
    <Card>
      <CardHeader className="flex flex-row items-center justify-between pb-2">
        <CardTitle className="text-sm font-medium text-muted-foreground">
          {title}
        </CardTitle>
        <div className="text-muted-foreground">{icon}</div>
      </CardHeader>
      <CardContent>
        <div className="text-2xl font-bold">{value}</div>
        <div className={cn(
          'flex items-center gap-1 text-xs mt-1',
          isPositive ? 'text-green-600' : 'text-red-600'
        )}>
          {isPositive ? (
            <TrendingUp className="h-3 w-3" />
          ) : (
            <TrendingDown className="h-3 w-3" />
          )}
          <span>{isPositive ? '+' : ''}{change}% from last month</span>
        </div>
      </CardContent>
    </Card>
  );
}

// Mock data
const statsData = [
  {
    title: 'Total Users',
    value: '2,847',
    change: 12.5,
    icon: <Users className="h-4 w-4" />,
  },
  {
    title: 'Active Spaces',
    value: '156',
    change: 8.2,
    icon: <Boxes className="h-4 w-4" />,
  },
  {
    title: 'Revenue',
    value: '$45,231',
    change: 20.1,
    icon: <DollarSign className="h-4 w-4" />,
  },
  {
    title: 'Growth Rate',
    value: '24.5%',
    change: -2.4,
    icon: <TrendingUp className="h-4 w-4" />,
  },
];

export function StatsCards() {
  return (
    <div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-4">
      {statsData.map((stat) => (
        <StatCard key={stat.title} {...stat} />
      ))}
    </div>
  );
}
```

### Step 2: Create Revenue Chart Component

Create `apps/web/src/features/admin/components/charts/revenue-chart.tsx`:

```tsx
'use client';

import {
  AreaChart,
  Area,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
} from 'recharts';
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from '@/components/ui/card';

// Mock data - last 12 months
const revenueData = [
  { month: 'Jan', revenue: 12400 },
  { month: 'Feb', revenue: 14800 },
  { month: 'Mar', revenue: 13200 },
  { month: 'Apr', revenue: 16500 },
  { month: 'May', revenue: 18900 },
  { month: 'Jun', revenue: 21200 },
  { month: 'Jul', revenue: 19800 },
  { month: 'Aug', revenue: 24100 },
  { month: 'Sep', revenue: 27500 },
  { month: 'Oct', revenue: 31200 },
  { month: 'Nov', revenue: 35800 },
  { month: 'Dec', revenue: 45231 },
];

export function RevenueChart() {
  return (
    <Card className="col-span-1">
      <CardHeader>
        <CardTitle>Revenue Trend</CardTitle>
        <CardDescription>Monthly revenue over the past year</CardDescription>
      </CardHeader>
      <CardContent>
        <div className="h-[300px]">
          <ResponsiveContainer width="100%" height="100%">
            <AreaChart
              data={revenueData}
              margin={{ top: 10, right: 10, left: 0, bottom: 0 }}
            >
              <defs>
                <linearGradient id="colorRevenue" x1="0" y1="0" x2="0" y2="1">
                  <stop offset="5%" stopColor="hsl(var(--primary))" stopOpacity={0.3} />
                  <stop offset="95%" stopColor="hsl(var(--primary))" stopOpacity={0} />
                </linearGradient>
              </defs>
              <CartesianGrid strokeDasharray="3 3" className="stroke-muted" />
              <XAxis
                dataKey="month"
                tick={{ fontSize: 12 }}
                tickLine={false}
                axisLine={false}
                className="text-muted-foreground"
              />
              <YAxis
                tick={{ fontSize: 12 }}
                tickLine={false}
                axisLine={false}
                tickFormatter={(value) => `$${(value / 1000).toFixed(0)}k`}
                className="text-muted-foreground"
              />
              <Tooltip
                content={({ active, payload }) => {
                  if (active && payload && payload.length) {
                    return (
                      <div className="rounded-lg border bg-background p-2 shadow-md">
                        <p className="text-sm font-medium">
                          ${payload[0].value?.toLocaleString()}
                        </p>
                      </div>
                    );
                  }
                  return null;
                }}
              />
              <Area
                type="monotone"
                dataKey="revenue"
                stroke="hsl(var(--primary))"
                strokeWidth={2}
                fill="url(#colorRevenue)"
              />
            </AreaChart>
          </ResponsiveContainer>
        </div>
      </CardContent>
    </Card>
  );
}
```

### Step 3: Create Users Chart Component

Create `apps/web/src/features/admin/components/charts/users-chart.tsx`:

```tsx
'use client';

import {
  BarChart,
  Bar,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
} from 'recharts';
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from '@/components/ui/card';

// Mock data - last 12 months
const usersData = [
  { month: 'Jan', users: 120, newUsers: 45 },
  { month: 'Feb', users: 180, newUsers: 62 },
  { month: 'Mar', users: 210, newUsers: 38 },
  { month: 'Apr', users: 285, newUsers: 78 },
  { month: 'May', users: 340, newUsers: 55 },
  { month: 'Jun', users: 420, newUsers: 82 },
  { month: 'Jul', users: 490, newUsers: 71 },
  { month: 'Aug', users: 580, newUsers: 95 },
  { month: 'Sep', users: 720, newUsers: 145 },
  { month: 'Oct', users: 890, newUsers: 175 },
  { month: 'Nov', users: 1150, newUsers: 268 },
  { month: 'Dec', users: 1450, newUsers: 312 },
];

export function UsersChart() {
  return (
    <Card className="col-span-1">
      <CardHeader>
        <CardTitle>User Growth</CardTitle>
        <CardDescription>Monthly user registrations</CardDescription>
      </CardHeader>
      <CardContent>
        <div className="h-[300px]">
          <ResponsiveContainer width="100%" height="100%">
            <BarChart
              data={usersData}
              margin={{ top: 10, right: 10, left: 0, bottom: 0 }}
            >
              <CartesianGrid strokeDasharray="3 3" className="stroke-muted" />
              <XAxis
                dataKey="month"
                tick={{ fontSize: 12 }}
                tickLine={false}
                axisLine={false}
                className="text-muted-foreground"
              />
              <YAxis
                tick={{ fontSize: 12 }}
                tickLine={false}
                axisLine={false}
                className="text-muted-foreground"
              />
              <Tooltip
                content={({ active, payload, label }) => {
                  if (active && payload && payload.length) {
                    return (
                      <div className="rounded-lg border bg-background p-2 shadow-md">
                        <p className="text-xs text-muted-foreground">{label}</p>
                        <p className="text-sm font-medium">
                          Total: {payload[0].value?.toLocaleString()}
                        </p>
                        <p className="text-sm text-muted-foreground">
                          New: +{payload[1].value?.toLocaleString()}
                        </p>
                      </div>
                    );
                  }
                  return null;
                }}
              />
              <Bar
                dataKey="users"
                fill="hsl(var(--primary))"
                radius={[4, 4, 0, 0]}
              />
              <Bar
                dataKey="newUsers"
                fill="hsl(var(--primary) / 0.3)"
                radius={[4, 4, 0, 0]}
              />
            </BarChart>
          </ResponsiveContainer>
        </div>
      </CardContent>
    </Card>
  );
}
```

### Step 4: Create Index Exports

Create `apps/web/src/features/admin/components/charts/index.ts`:

```ts
export { RevenueChart } from './revenue-chart';
export { UsersChart } from './users-chart';
```

Create `apps/web/src/features/admin/components/index.ts`:

```ts
export { StatsCards } from './stats-cards';
export * from './charts';
```

Create `apps/web/src/features/admin/index.ts`:

```ts
export * from './components';
```

### Step 5: Update Admin Dashboard Page

Update `apps/web/src/app/(admin)/page.tsx`:

```tsx
import { StatsCards, RevenueChart, UsersChart } from '@/features/admin';

export default function AdminDashboardPage() {
  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-heading font-bold">Dashboard</h1>
        <p className="text-muted-foreground">
          Overview of your platform metrics and analytics
        </p>
      </div>

      {/* Stats Cards */}
      <StatsCards />

      {/* Charts */}
      <div className="grid gap-6 md:grid-cols-2">
        <RevenueChart />
        <UsersChart />
      </div>
    </div>
  );
}
```

## Todo List

- [ ] Create features/admin folder structure
- [ ] Create stats-cards.tsx with mock data
- [ ] Create revenue-chart.tsx (Area chart)
- [ ] Create users-chart.tsx (Bar chart)
- [ ] Create index.ts exports
- [ ] Update (admin)/page.tsx with dashboard components
- [ ] Test charts render correctly
- [ ] Test responsive layout
- [ ] Test chart tooltips work
- [ ] Run `pnpm build` to verify no errors

## Success Criteria

- [ ] 4 stats cards display with icons, values, and change indicators
- [ ] Revenue chart shows area chart with gradient fill
- [ ] Users chart shows bar chart with two series
- [ ] Charts have working tooltips
- [ ] Responsive: cards stack on mobile, 2-col charts on tablet+
- [ ] Colors use project CSS variables (--primary)
- [ ] Build passes

## Security Considerations

- Mock data only - no sensitive info
- Future: API calls should be authenticated

## Next Steps

After completing Phase 3:
- Phase 4: Create placeholder pages for Spaces and Users management
