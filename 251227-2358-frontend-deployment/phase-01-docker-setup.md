# Phase 1: Docker Setup

## Context

- **Parent Plan:** [plan.md](./plan.md)
- **Dependencies:** None
- **Blocks:** Phase 2, Phase 3

## Overview

| Field | Value |
|-------|-------|
| Date | 2025-12-28 |
| Priority | P2 |
| Effort | 30m |
| Status | pending |

**Mục tiêu:** Tạo Dockerfile cho Next.js với multi-stage build và standalone output.

## Requirements

1. Docker image optimized (target < 300MB)
2. Non-root user execution
3. Health check endpoint
4. Compatible với Coolify build system

## Files to Create

```
apps/web/
├── Dockerfile
├── .dockerignore
└── src/app/api/health/route.ts
```

## Implementation Steps

### Step 1: Update next.config.ts

```typescript
// apps/web/next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  output: "standalone",
};

export default nextConfig;
```

### Step 2: Create Health Check Endpoint

```typescript
// apps/web/src/app/api/health/route.ts
export async function GET() {
  return new Response("OK", { status: 200 });
}
```

### Step 3: Create .dockerignore

```dockerignore
# apps/web/.dockerignore
node_modules
.next
.git
.gitignore
*.md
.env*.local
.env.development
.env.test
tests
__tests__
*.test.*
*.spec.*
coverage
.turbo
.eslintcache
.vscode
.idea
Dockerfile*
docker-compose*
```

### Step 4: Create Dockerfile

```dockerfile
# apps/web/Dockerfile
# ===========================================
# Stage 1: Dependencies
# ===========================================
FROM node:22-alpine AS deps
WORKDIR /app

RUN corepack enable && corepack prepare pnpm@10.26.1 --activate

COPY pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/web/package.json ./apps/web/

RUN pnpm install --frozen-lockfile --filter web

# ===========================================
# Stage 2: Builder
# ===========================================
FROM node:22-alpine AS builder
WORKDIR /app

RUN corepack enable && corepack prepare pnpm@10.26.1 --activate

COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/apps/web/node_modules ./apps/web/node_modules

COPY apps/web ./apps/web
COPY pnpm-lock.yaml pnpm-workspace.yaml turbo.json ./

ENV NEXT_TELEMETRY_DISABLED=1
WORKDIR /app/apps/web
RUN pnpm build

# ===========================================
# Stage 3: Runner
# ===========================================
FROM node:22-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

COPY --from=builder --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/apps/web/.next/static ./apps/web/.next/static
COPY --from=builder --chown=nextjs:nodejs /app/apps/web/public ./apps/web/public

USER nextjs

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/api/health || exit 1

CMD ["node", "apps/web/server.js"]
```

### Step 5: Test Local Build

```bash
# From project root
docker build -f apps/web/Dockerfile -t synkao-web:test .

# Check image size (target: < 300MB)
docker images synkao-web:test

# Test run
docker run -p 3000:3000 synkao-web:test

# Test health endpoint
curl http://localhost:3000/api/health
```

## Success Criteria

- [ ] `docker build` thành công
- [ ] Image size < 300MB
- [ ] Container runs as non-root (nextjs user)
- [ ] Health check returns 200 OK
- [ ] App accessible at localhost:3000

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Monorepo context issues | High | Build from root với correct context |
| Standalone path mismatch | Medium | Verify server.js location |
| Missing static files | Medium | Check .next/static copied |

## Next Steps

→ [Phase 2: Coolify Installation](./phase-02-coolify-installation.md)
