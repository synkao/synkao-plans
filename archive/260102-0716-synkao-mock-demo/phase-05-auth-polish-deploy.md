# Phase 5: Auth Pages + Polish + Deploy

## Context Links
- **Parent Plan:** [plan.md](./plan.md)
- **Dependencies:** All previous phases
- **Design System:** [synkao-design-system.md](../../docs/design/synkao-design-system.md)
- **UI Spec:** Section 4 (Auth), Section 9-10 (Common Patterns, Error States)

---

## Overview

| Property | Value |
|----------|-------|
| Priority | P3 |
| Status | pending |
| Effort | 1.5 days |
| Description | Build auth pages (Login, Forgot Password), add loading states, empty states, polish UI, Docker build, Coolify deploy |

---

## Key Insights

1. **Auth Layout** - Centered glass card, no sidebar
2. **Login Form** - Email, password, remember me, forgot password link
3. **No Real Auth** - Static demo, redirect to dashboard on submit
4. **Loading States** - Skeleton components for tables, cards
5. **Empty States** - Illustration + message + action button
6. **Docker Standalone** - Giữ Dockerfile hiện tại (Node.js server, không static export)

---

## Requirements

### Functional
- F1: Login page with glass card form
- F2: Forgot password page
- F3: "Login" button redirects to dashboard (no real auth)
- F4: Loading skeletons for tables and cards
- F5: Empty state components
- F6: Error state components
- F7: Docker build succeeds
- F8: Deploy to Coolify

### Non-Functional
- NF1: Glass effect on auth cards
- NF2: Consistent spacing and typography
- NF3: Docker build succeeds with existing Dockerfile
- NF4: Health check endpoint works

---

## Architecture

### Auth Layout
```
(auth)/layout.tsx
├── Background (gradient)
└── CenteredContainer
    └── GlassCard
        └── Children (login/forgot-password)
```

### Login Page
```
LoginPage
└── GlassCard
    ├── Logo
    ├── Title
    ├── Form
    │   ├── EmailInput
    │   ├── PasswordInput (with toggle)
    │   ├── RememberCheckbox
    │   └── SubmitButton
    └── ForgotPasswordLink
```

### Common Patterns
```
LoadingState
├── TableSkeleton
├── CardSkeleton
└── PageSkeleton

EmptyState
├── Illustration
├── Title
├── Description
└── ActionButton
```

---

## Related Code Files

### Create - Pages
| File Path | Description |
|-----------|-------------|
| `apps/web/src/app/(auth)/layout.tsx` | Auth layout - centered, gradient bg |
| `apps/web/src/app/(auth)/login/page.tsx` | Login page |
| `apps/web/src/app/(auth)/forgot-password/page.tsx` | Forgot password page |

### Create - Auth Components
| File Path | Description |
|-----------|-------------|
| `apps/web/src/features/auth/components/login-form.tsx` | Email, password, remember me |
| `apps/web/src/features/auth/components/forgot-password-form.tsx` | Email reset form |
| `apps/web/src/features/auth/components/glass-auth-card.tsx` | Glass card wrapper |
| `apps/web/src/features/auth/components/password-input.tsx` | Password với toggle visibility |
| `apps/web/src/features/auth/components/index.ts` | Barrel exports |
| `apps/web/src/features/auth/index.ts` | Feature barrel exports |

### Create - Common Components
| File Path | Description |
|-----------|-------------|
| `apps/web/src/components/common/table-skeleton.tsx` | Table loading skeleton |
| `apps/web/src/components/common/card-skeleton.tsx` | Card loading skeleton |
| `apps/web/src/components/common/page-skeleton.tsx` | Full page loading |
| `apps/web/src/components/common/empty-state.tsx` | Empty state với icon + action |
| `apps/web/src/components/common/error-state.tsx` | Error state với retry |
| `apps/web/src/components/common/index.ts` | Common barrel exports |

### Create - API Route
| File Path | Description |
|-----------|-------------|
| `apps/web/src/app/api/health/route.ts` | Health check endpoint cho Docker |

### Modify
| File Path | Change Description |
|-----------|-------------------|
| `apps/web/src/app/globals.css` | Final polish - glass utilities, spacing fixes |
| `apps/web/src/app/layout.tsx` | Add Be Vietnam Pro font if not added |

### Verify (No Changes)
| File Path | Description |
|-----------|-------------|
| `apps/web/Dockerfile` | Existing standalone build - verify works |
| `apps/web/next.config.ts` | Verify output: 'standalone' đã có |

### Delete
| File Path | Reason |
|-----------|--------|
| `apps/web/src/features/auth/.gitkeep` | Folder sẽ có files |

---

## Implementation Steps

### 1. Create Auth Layout (30 min)
```tsx
// apps/web/src/app/(auth)/layout.tsx
export default function AuthLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-slate-50 to-blue-50">
      <div className="w-full max-w-md">
        {children}
      </div>
      <footer className="absolute bottom-4 text-center text-sm text-muted">
        2025 Synkao. All rights reserved.
      </footer>
    </div>
  );
}
```

### 2. Create GlassAuthCard (20 min)
```tsx
export function GlassAuthCard({ children }: { children: React.ReactNode }) {
  return (
    <Card className="bg-white/80 backdrop-blur-lg border-white/20 shadow-xl">
      <CardContent className="p-8">
        {children}
      </CardContent>
    </Card>
  );
}
```

### 3. Create LoginForm (45 min)
```tsx
export function LoginForm() {
  const router = useRouter();

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    router.push('/');
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div className="text-center mb-6">
        <Logo className="mx-auto" />
        <h1 className="text-2xl font-semibold mt-4">Đăng nhập vào hệ thống</h1>
      </div>

      <div className="space-y-2">
        <Label htmlFor="email">Email</Label>
        <Input id="email" type="email" placeholder="email@example.com" />
      </div>

      <div className="space-y-2">
        <Label htmlFor="password">Mật khẩu</Label>
        <div className="relative">
          <Input id="password" type="password" placeholder="********" />
          <TogglePasswordButton />
        </div>
      </div>

      <div className="flex items-center space-x-2">
        <Checkbox id="remember" />
        <Label htmlFor="remember">Ghi nhớ đăng nhập</Label>
      </div>

      <Button type="submit" className="w-full">Đăng nhập</Button>

      <div className="text-center">
        <Link href="/forgot-password" className="text-sm text-blue-600 hover:underline">
          Quên mật khẩu?
        </Link>
      </div>
    </form>
  );
}
```

### 4. Create Login Page (15 min)
```tsx
export default function LoginPage() {
  return (
    <GlassAuthCard>
      <LoginForm />
    </GlassAuthCard>
  );
}
```

### 5. Create ForgotPasswordForm (30 min)
Similar to login, with:
- Email input
- Submit button
- Back to login link
- Success state (static)

### 6. Create Forgot Password Page (15 min)

### 7. Create Skeleton Components (45 min)

**TableSkeleton:**
```tsx
export function TableSkeleton({ rows = 5 }: { rows?: number }) {
  return (
    <div className="space-y-2">
      {Array.from({ length: rows }).map((_, i) => (
        <Skeleton key={i} className="h-12 w-full" />
      ))}
    </div>
  );
}
```

**CardSkeleton:**
```tsx
export function CardSkeleton() {
  return (
    <Card>
      <CardContent className="p-4">
        <Skeleton className="h-4 w-3/4 mb-2" />
        <Skeleton className="h-4 w-1/2" />
      </CardContent>
    </Card>
  );
}
```

### 8. Create EmptyState Component (30 min)
```tsx
interface EmptyStateProps {
  icon?: LucideIcon;
  title: string;
  description: string;
  action?: {
    label: string;
    href: string;
  };
}

export function EmptyState({ icon: Icon, title, description, action }: EmptyStateProps) {
  return (
    <div className="flex flex-col items-center justify-center py-12 text-center">
      {Icon && (
        <div className="p-4 rounded-full bg-muted mb-4">
          <Icon className="h-8 w-8 text-muted-foreground" />
        </div>
      )}
      <h3 className="text-lg font-medium">{title}</h3>
      <p className="text-muted-foreground mt-1 max-w-sm">{description}</p>
      {action && (
        <Button asChild className="mt-4">
          <Link href={action.href}>{action.label}</Link>
        </Button>
      )}
    </div>
  );
}
```

### 9. Create ErrorState Component (20 min)
Similar to EmptyState with retry button

### 10. Create Health Check Endpoint (15 min)
```typescript
// apps/web/src/app/api/health/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  return NextResponse.json({ status: 'ok', timestamp: new Date().toISOString() });
}
```

### 11. Verify Dockerfile Works (20 min)
Existing Dockerfile đã có standalone build. Chỉ cần verify:
```bash
cd apps/web
docker build -t synkao-demo .
docker run -p 3000:3000 synkao-demo
# Test: http://localhost:3000
# Test: http://localhost:3000/api/health
```

### 12. Final Polish (1 hour)
- Review all pages for consistency
- Check spacing and typography
- Verify glass effects render correctly
- Test navigation flows
- Check responsive at 1280px+

### 13. Test Docker Build (30 min)
```bash
# Build từ root của monorepo
docker build -f apps/web/Dockerfile -t synkao-demo .
docker run -p 3000:3000 synkao-demo

# Test all routes:
curl http://localhost:3000/api/health
# Navigate to all pages manually
```

### 14. Deploy to Coolify (30 min)
- Push to Git
- Configure Coolify with Docker
- Set port 3000
- Deploy and verify

---

## Todo List

### Auth Feature
- [ ] Create `apps/web/src/app/(auth)/layout.tsx`
- [ ] Create `apps/web/src/features/auth/components/glass-auth-card.tsx`
- [ ] Create `apps/web/src/features/auth/components/password-input.tsx`
- [ ] Create `apps/web/src/features/auth/components/login-form.tsx`
- [ ] Create `apps/web/src/features/auth/components/forgot-password-form.tsx`
- [ ] Create `apps/web/src/features/auth/components/index.ts`
- [ ] Create `apps/web/src/features/auth/index.ts`
- [ ] Create `apps/web/src/app/(auth)/login/page.tsx`
- [ ] Create `apps/web/src/app/(auth)/forgot-password/page.tsx`

### Common Components
- [ ] Create `apps/web/src/components/common/table-skeleton.tsx`
- [ ] Create `apps/web/src/components/common/card-skeleton.tsx`
- [ ] Create `apps/web/src/components/common/page-skeleton.tsx`
- [ ] Create `apps/web/src/components/common/empty-state.tsx`
- [ ] Create `apps/web/src/components/common/error-state.tsx`
- [ ] Create `apps/web/src/components/common/index.ts`

### API & Config
- [ ] Create `apps/web/src/app/api/health/route.ts`
- [ ] Update `apps/web/src/app/globals.css` - final polish

### Testing - Local
- [ ] Test login form redirects to dashboard
- [ ] Test forgot password page renders
- [ ] Test loading skeletons display
- [ ] Test empty states display
- [ ] Run `pnpm build` successfully

### Testing - Docker
- [ ] Run Docker build successfully
- [ ] Test Docker container locally
- [ ] Test /api/health endpoint

### Deployment
- [ ] Deploy to Coolify
- [ ] Verify all 12 pages in production

---

## Success Criteria

### Auth Pages
- [ ] Login page renders with glass effect
- [ ] "Đăng nhập" redirects to dashboard
- [ ] Forgot password page works

### Common Components
- [ ] Skeleton components available and usable
- [ ] Empty state components available
- [ ] Error state components available

### Build & Deploy
- [ ] `pnpm build` completes without errors
- [ ] Docker image builds successfully (standalone)
- [ ] Container runs và serves pages
- [ ] /api/health endpoint returns 200
- [ ] Coolify deployment successful
- [ ] All 12 pages accessible in production

---

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Docker build fails | Medium | Test locally first, use existing Dockerfile |
| Health check fails | Low | Add /api/health route |
| Coolify config | Medium | Document settings, use same port 3000 |
| Font loading | Low | Use next/font for optimization |

---

## Deployment Checklist

Before deploying:
1. [ ] All pages render correctly locally
2. [ ] `pnpm build` succeeds
3. [ ] Docker build succeeds (from monorepo root)
4. [ ] Container runs locally on port 3000
5. [ ] All routes accessible
6. [ ] Glass effects visible
7. [ ] /api/health returns 200

Coolify settings:
- Source: Git repository
- Build pack: Dockerfile
- Dockerfile location: `apps/web/Dockerfile`
- Port: 3000
- Health check: /api/health

---

## Final Verification

After deployment, verify all pages:

| Page | Route | Status |
|------|-------|--------|
| Login | /login | [ ] |
| Forgot Password | /forgot-password | [ ] |
| Dashboard | / | [ ] |
| Orders List | /orders | [ ] |
| Order Detail | /orders/[id] | [ ] |
| Import Wizard | /orders/import | [ ] |
| Design Backlog | /design/backlog | [ ] |
| Workspace List | /design/workspace | [ ] |
| Kanban Board | /design/workspace/[id] | [ ] |
| Team Settings | /settings/team | [ ] |
| Workspace Settings | /settings/workspaces | [ ] |

---

## Post-Demo Notes

This is a **static UI demo** for stakeholder validation. After approval:
1. Remove mock data, connect to real API
2. Add proper authentication
3. Implement form validation
4. Add drag-drop to Kanban
5. Add mobile responsive
6. Add real-time updates
