# Next.js Docker Deployment Research Report
**Date:** 2025-12-27 | **Focus:** Next.js 16+ Production Deployment

## Executive Summary
Next.js standalone output + multi-stage builds reduce Docker images by **75%** (892MB → 229MB). Combine with health checks, non-root users, and cache optimization for production-ready deployments.

---

## 1. KEY FINDINGS

### Standalone Output Mode
- **Configuration:** Add `output: "standalone"` in `next.config.ts`
- **Impact:** Packages only production-critical files, excludes dev dependencies
- **Result:** 75% image size reduction vs standard builds
- **Files generated:** `.next/standalone` folder contains isolated bundle
- **Citation:** [CodeParrot 2025 Guide](https://codeparrot.ai/blogs/deploy-nextjs-app-with-docker-complete-guide-for-2025), [Medium: Next.js Docker Multi-Stage](https://medium.com/@ysuwansiri/running-next-js-in-docker-a-modern-multi-stage-setup-for-production-0bc348389f85)

### Multi-Stage Build Architecture
- **Stage 1 (Builder):** Install all deps (including dev), run build, generate `.next/standalone`
- **Stage 2 (Runner):** Copy only `.next/standalone`, `.next/static`, `/public`
- **Benefits:** Smaller final image, no full node_modules, enhanced security
- **Base Image:** Use `node:22-alpine` for lightweight foundation
- **Citation:** [Simplr Blog: Next.js Docker](https://blog.simplr.sh/posts/next-js-docker-deployment/), [GitHub Discussion](https://github.com/vercel/next.js/discussions/16995)

### Environment Variables
- **Approach:** Set via runtime environment variables, NOT embedded in image
- **Security:** Avoid secrets in Dockerfile; use orchestrator secret management
- **Telemetry:** Set `ENV NEXT_TELEMETRY_DISABLED=1` to reduce noise
- **Citation:** [Complete Guide to Self-Hosting](https://dlhck.com/thoughts/the-complete-guide-to-self-hosting-nextjs-at-scale)

### Health Check Endpoints
- **Dockerfile Config:**
  ```dockerfile
  EXPOSE 3000
  HEALTHCHECK --interval=12s --timeout=12s --start-period=5s \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000 || exit 1
  ```
- **API Route (Next.js 15+):**
  ```typescript
  // app/api/health/route.ts
  export async function GET() {
    return new Response("OK");
  }
  ```
- **Critical:** Without health checks, orchestrators can't detect replica readiness
- **Citation:** [Medium: Optimizing Containerized Deployments](https://arnab-k.medium.com/optimizing-containerized-deployments-of-next-js-with-docker-c3a1d373cb82)

### Production Optimizations
| Optimization | Impact | Method |
|---|---|---|
| **Image Size** | 892MB → 229MB | Standalone mode + multi-stage |
| **Security** | Non-root execution | Create `nextjs` user in runner stage |
| **Build Caching** | 50%+ faster rebuilds | Copy package files first, app files second |
| **Layer Reduction** | Smaller diffs | Combine RUN commands: `npm ci && npm run build` |
| **.dockerignore** | Faster builds | Exclude node_modules, tests, .git, .env files |

---

## 2. RECOMMENDED DOCKERFILE TEMPLATE

```dockerfile
# Stage 1: Builder
FROM node:22-alpine AS builder
WORKDIR /app
ENV NEXT_TELEMETRY_DISABLED=1

# Copy dependency files first (better caching)
COPY package.json pnpm-lock.yaml ./

# Install deps
RUN npm install -g pnpm && pnpm install --frozen-lockfile

# Copy source and build
COPY . .
RUN pnpm build

# Stage 2: Runner
FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Copy built artifacts from builder
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
COPY --from=builder --chown=nextjs:nodejs /app/public ./public

USER nextjs

EXPOSE 3000

# Health check (critical for orchestration)
HEALTHCHECK --interval=12s --timeout=12s --start-period=5s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000 || exit 1

CMD ["node", "server.js"]
```

---

## 3. COMMON PITFALLS & SOLUTIONS

| Pitfall | Impact | Solution |
|---|---|---|
| **Missing standalone config** | Large images (1.5GB+) | Add `output: "standalone"` to next.config.ts |
| **Copying full node_modules** | Security vulnerabilities, huge images | Use standalone + multi-stage only |
| **Running as root** | Container compromise risk | Create non-root user (`nextjs`) |
| **Cache invalidation** | Slow rebuilds, wasted CI/CD time | Copy package files before app code |
| **Missing health checks** | Restart loops, zero-downtime failures | Implement proper HEALTHCHECK with start-period |
| **Hardcoded secrets** | Credential leaks | Use runtime env vars, orchestrator secrets |
| **Listening on localhost only** | Port unreachable in containers | Bind to `0.0.0.0:3000` (Next.js default) |
| **Cache persistence** | Inconsistent performance across replicas | Use external cache layer (Redis) for multi-replica setups |

---

## 4. ENVIRONMENT SETUP CHECKLIST

- [ ] Enable `output: "standalone"` in next.config.ts
- [ ] Create `.dockerignore` with: node_modules, .git, test files, .env*.local
- [ ] Use Alpine Linux base image (node:22-alpine)
- [ ] Non-root user with UID 1001 (nextjs)
- [ ] Health check endpoint at `/api/health` or similar
- [ ] HEALTHCHECK instruction in Dockerfile
- [ ] Copy package files first for layer caching
- [ ] Copy .next/standalone, .next/static, public only
- [ ] Set NEXT_TELEMETRY_DISABLED=1
- [ ] Test image size post-build (<500MB target)

---

## KEY CITATIONS

- [CodeParrot: NextJS Docker Deployment 2025](https://codeparrot.ai/blogs/deploy-nextjs-app-with-docker-complete-guide-for-2025)
- [Medium: Multi-Stage Setup 2025](https://medium.com/@ysuwansiri/running-next-js-in-docker-a-modern-multi-stage-setup-for-production-0bc348389f85)
- [Simplr: Next.js Docker Deployment](https://blog.simplr.sh/posts/next-js-docker-deployment/)
- [DEV Community: Standalone Mode](https://dev.to/miannemendoza/how-to-use-nextjs-standalone-for-leaner-docker-images-2kn9)
- [GitHub Discussion #16995](https://github.com/vercel/next.js/discussions/16995)
- [Tim Santeford: Production Optimization](https://www.timsanteford.com/posts/optimizing-next-js-docker-images-for-production/)
- [Complete Self-Hosting Guide](https://dlhck.com/thoughts/the-complete-guide-to-self-hosting-nextjs-at-scale)

---

## UNRESOLVED QUESTIONS

1. Should we use `node:22-alpine` or `node:22-slim` for Synkao? (Alpine is smaller but may lack some system utils)
2. Cache persistence strategy for multi-replica deployments—use Redis, shared volume, or CDN?
3. Should health check endpoint include database/Redis connectivity checks or simple 200 OK?
4. Any custom middleware or configuration in Synkao that affects Docker build process?
