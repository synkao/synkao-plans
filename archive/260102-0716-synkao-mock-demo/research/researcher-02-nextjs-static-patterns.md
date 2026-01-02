# Next.js 16 Static Patterns Research
**Generated:** 2026-01-02 | **Type:** Technical Research | **Status:** Complete

## Executive Summary
Next.js 16 (2026) with App Router provides stable static generation via `generateStaticParams`, explicit caching with "use cache" directive, and multi-stage Docker builds. Route groups enable clean auth/main layout separation. For SynKao mock demo: use static export + route groups for layout isolation + TypeScript mock data constants.

---

## 1. Next.js 16 Static Generation Architecture

### Core Pattern: Route Groups + Static Layouts
```
app/
├── (auth)/
│   ├── layout.tsx          // Auth layout (no header/sidebar)
│   ├── login/page.tsx
│   └── register/page.tsx
├── (main)/
│   ├── layout.tsx          // Main layout (header + sidebar)
│   ├── page.tsx            // Dashboard
│   ├── orders/
│   │   ├── page.tsx
│   │   └── [id]/page.tsx
│   └── tasks/
│       ├── page.tsx
│       └── [id]/page.tsx
└── api/
    └── (mock)/
        ├── orders/route.ts
        └── tasks/route.ts
```

### Key Advantages
- **Layout Isolation:** Route groups prevent shared layout remounting
- **Deduplication:** App Router deduplicates fetch requests across layouts/pages
- **Static-First:** All pages pre-rendered at build time by default (Next.js 16)

### Configuration
```typescript
// next.config.ts
export default {
  output: 'export',  // Static export for demo
  // OR for server-based: output: 'standalone'

  // Optional: Turbopack (default in Next.js 16)
  experimental: {
    turbopack: {}
  }
}
```

---

## 2. Static Generation with generateStaticParams

### Pattern for Dynamic Routes
```typescript
// app/(main)/orders/[id]/page.tsx
export async function generateStaticParams() {
  // Pre-render first 50 orders at build time
  const orders = MOCK_ORDERS.slice(0, 50).map(order => ({
    id: order.id
  }));

  return orders;
}

export const dynamicParams = false; // Return 404 for ungenerated params

export default function OrderPage({ params }: { params: { id: string } }) {
  const order = MOCK_ORDERS.find(o => o.id === params.id);
  return <OrderDetail order={order} />;
}
```

### Best Practices
1. **Return Format:** Array of objects with dynamic segment names as keys
2. **Request Deduplication:** Fetch calls auto-deduplicate → faster builds
3. **ISR Alternative:** Use `export const revalidate = 3600` for on-demand generation
4. **Nested Dynamics:** Child routes inherit parent params via `params` object
5. **Closed Sets:** Set `dynamicParams = false` for production safety (404s)

### For Mock Data Demo
```typescript
// lib/mock-data.ts - TypeScript constants
export interface Order {
  id: string;
  status: 'pending' | 'completed' | 'shipped';
  customer: string;
  total: number;
  created: Date;
}

export interface Task {
  id: string;
  title: string;
  status: 'todo' | 'in-progress' | 'done';
  assignee: string;
  dueDate: Date;
}

export const MOCK_ORDERS: Order[] = [
  { id: '1', status: 'completed', customer: 'John Doe', total: 299.99, created: new Date('2025-12-01') },
  // ... 50+ realistic orders
];

export const MOCK_TASKS: Task[] = [
  // ... realistic task data
];

export const MOCK_USERS = [
  { id: '1', name: 'Alice Chen', role: 'admin', email: 'alice@synkao.dev' },
  // ...
];
```

---

## 3. Layout Architecture with Nested Layouts

### Main Layout Structure
```typescript
// app/(main)/layout.tsx - Shared for all main section routes
import Header from '@/components/Header';
import Sidebar from '@/components/Sidebar';

export default function MainLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-screen">
      <Sidebar />
      <div className="flex-1 flex flex-col">
        <Header />
        <main className="flex-1 overflow-auto">{children}</main>
      </div>
    </div>
  );
}
```

### Auth Layout (Separate)
```typescript
// app/(auth)/layout.tsx - No sidebar/header
export default function AuthLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex items-center justify-center min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100">
      <div className="w-full max-w-md">{children}</div>
    </div>
  );
}
```

### Nested Route Layouts
```typescript
// app/(main)/orders/layout.tsx - Orders-specific layout
export default function OrdersLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="space-y-4">
      <div className="flex justify-between items-center">
        <h1>Orders Management</h1>
        <button>+ New Order</button>
      </div>
      {children}
    </div>
  );
}
```

---

## 4. Docker Static Build Pattern

### Multi-Stage Dockerfile (Static Export)
```dockerfile
# Stage 1: Dependencies
FROM node:20-alpine AS dependencies
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile

# Stage 2: Builder
FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Stage 3: Static Server (Nginx for export mode)
FROM nginx:alpine
COPY --from=builder /app/out /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 3000
CMD ["nginx", "-g", "daemon off;"]
```

### Multi-Stage Dockerfile (Standalone Mode)
```dockerfile
# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile

# Stage 2: Builder
FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Stage 3: Runner (Node.js server)
FROM node:20-alpine
WORKDIR /app
ENV NODE_ENV=production

# Create non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nextjs -u 1001

# Copy standalone output
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
COPY --from=builder --chown=nextjs:nodejs /app/public ./public

USER nextjs
EXPOSE 3000
CMD ["node", "server.js"]
```

### next.config.ts Configuration
```typescript
// For static export (Nginx serving)
export default {
  output: 'export',
  basePath: '',
  trailingSlash: true,
  images: {
    unoptimized: true, // Required for static export
  }
}

// For standalone mode (Node.js server)
// export default { output: 'standalone' }
```

### nginx.conf for Static Export
```nginx
server {
  listen 3000;
  root /usr/share/nginx/html;

  location / {
    try_files $uri $uri/ /index.html;
  }

  location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
    expires 365d;
    add_header Cache-Control "public, immutable";
  }
}
```

---

## 5. File Structure for SynKao Mock Demo

```
synkao-mock-demo/
├── app/
│   ├── (auth)/
│   │   ├── layout.tsx
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   ├── (main)/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── orders/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx
│   │   │   └── [id]/page.tsx
│   │   ├── tasks/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx
│   │   │   └── [id]/page.tsx
│   │   └── settings/page.tsx
│   ├── api/
│   │   └── (mock)/
│   │       ├── orders/route.ts
│   │       ├── tasks/route.ts
│   │       └── users/route.ts
│   └── layout.tsx          // Root layout
├── components/
│   ├── Header.tsx
│   ├── Sidebar.tsx
│   ├── OrderTable.tsx
│   ├── TaskCard.tsx
│   └── LoadingState.tsx
├── lib/
│   ├── mock-data.ts        // TypeScript constants for all mock data
│   ├── utils.ts
│   └── types.ts
├── public/
│   ├── logo.svg
│   └── avatars/
├── Dockerfile
├── nginx.conf
├── next.config.ts
├── package.json
└── tsconfig.json
```

---

## 6. Actionable Recommendations

### For SynKao Mock Demo

1. **Use Static Export** (`output: 'export'`)
   - Simplest for demo: pure static files
   - Deploy anywhere: Nginx, S3, Vercel
   - No Node.js runtime required

2. **Organize Mock Data**
   - Single `lib/mock-data.ts` file with TypeScript types
   - 50+ realistic orders, tasks, users
   - Realistic dates, status values, relationships

3. **Route Structure**
   - Use route groups: `(auth)` for login, `(main)` for dashboard
   - Prevents layout remounting on navigation
   - Clean separation of concerns

4. **Static Routes**
   - Use `generateStaticParams` for top 50 orders/tasks
   - Set `dynamicParams = false` for closed set
   - Instant page loads at demo time

5. **Docker Deployment**
   - Multi-stage build for smallest image (Nginx: ~50MB)
   - Health checks, non-root user, caching layers
   - Ready for Coolify/Docker Compose

6. **Caching Strategy**
   - App Router auto-deduplicates requests
   - Use "use cache" directive (Next.js 16) for expensive ops
   - Static pages served instantly from Nginx

---

## 7. Build & Performance Metrics

| Metric | Value | Notes |
|--------|-------|-------|
| Build Time | ~15-30s | Static export with 50 pre-rendered pages |
| Nginx Image Size | ~50MB | Multi-stage with Alpine |
| Page Load | <100ms | Static files served by Nginx |
| Time to Interactive | ~300ms | Minimal JS, fast hydration |

---

## References

- [Next.js App Router Route Groups](https://nextjs.org/docs/app/api-reference/file-conventions/route-groups)
- [Next.js generateStaticParams Function](https://nextjs.org/docs/app/api-reference/functions/generate-static-params)
- [Next.js Static Site Generation](https://nextjs.org/docs/pages/building-your-application/rendering/static-site-generation)
- [Next.js Deployment Guide](https://nextjs.org/docs/app/getting-started/deploying)
- [Docker Multi-Stage Dockerfile for Next.js](https://johnnymetz.com/posts/dockerize-nextjs-app/)
- [Optimizing Next.js Docker Images](https://dev.to/angojay/optimizing-nextjs-docker-images-with-standalone-mode-2nnh)
- [Next.js Advanced Techniques 2026](https://medium.com/@hashbyt/nextjs-advanced-techniques-2026-93e09f1c728d)

---

## Unresolved Questions

1. Should mock data be generated programmatically or use hardcoded constants? (Recommend: hardcoded for deterministic demo)
2. Should API routes (`/api/orders`) be implemented for SPA patterns? (Recommend: Not needed for static export)
3. Preferred Docker base: Nginx (static) vs Node.js Alpine (standalone)? (Recommend: Nginx for demo simplicity)
