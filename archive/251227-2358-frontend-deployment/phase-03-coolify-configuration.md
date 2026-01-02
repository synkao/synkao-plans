# Phase 3: Coolify Configuration

## Context

- **Parent Plan:** [plan.md](./plan.md)
- **Dependencies:** [Phase 1](./phase-01-docker-setup.md), [Phase 2](./phase-02-coolify-installation.md)
- **Blocks:** None (final phase)

## Overview

| Field | Value |
|-------|-------|
| Date | 2025-12-28 |
| Priority | P2 |
| Effort | 1h |
| Status | pending |

**Mục tiêu:** Configure 2 projects trong Coolify: Production và Staging.

## Implementation Steps

### Step 1: Create Production Project

1. **Dashboard** → **Projects** → **Add New**
2. **Name:** `synkao-production`
3. **Description:** Production frontend

### Step 2: Add Production Resource

1. Click **Add New Resource**
2. Select **Public Repository** hoặc **Private Repository** (if GitHub connected)
3. **Repository:** `https://github.com/synkao/synkao`
4. **Branch:** `main`
5. **Build Pack:** Docker

### Step 3: Configure Production Build

| Setting | Value |
|---------|-------|
| Dockerfile Location | `apps/web/Dockerfile` |
| Docker Build Context | `.` (root) |
| Port | 3000 |

### Step 4: Configure Production Domain

1. **Domains** → Add domain
2. **Domain:** `synkao.com`
3. **Add also:** `www.synkao.com`
4. **Enable:** Let's Encrypt SSL

### Step 5: Set Production Environment Variables

```
NODE_ENV=production
NEXT_PUBLIC_API_URL=https://api.synkao.com
```

### Step 6: Create Staging Project

Repeat Steps 1-5 với:

| Setting | Value |
|---------|-------|
| Project Name | `synkao-staging` |
| Branch | `develop` (hoặc `main`) |
| Domain | `staging.synkao.com` |
| NEXT_PUBLIC_API_URL | `https://staging-api.synkao.com` |

### Step 7: Configure Health Check

Trong mỗi project → **Health Check**:

| Setting | Value |
|---------|-------|
| Path | `/api/health` |
| Port | 3000 |
| Interval | 30s |

### Step 8: Test Deployment

1. Click **Deploy** button
2. Watch build logs
3. Wait for health check to pass
4. Access domain to verify

## Deployment Workflow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Git Push   │ ──► │   Coolify   │ ──► │   Deploy    │
│  (manual)   │     │  Build Log  │     │   Live!     │
└─────────────┘     └─────────────┘     └─────────────┘
```

**Manual Deploy:** Click "Deploy" trong Coolify UI

**Auto Deploy (optional):** Enable webhook trong project settings

## Rollback Procedure

1. Go to **Deployments** tab
2. Find previous successful deployment
3. Click **Rollback to this deployment**

## Environment-Specific Settings

### Production
| Setting | Value |
|---------|-------|
| Domain | synkao.com |
| Branch | main |
| Auto Deploy | Off (manual) |

### Staging
| Setting | Value |
|---------|-------|
| Domain | staging.synkao.com |
| Branch | develop |
| Auto Deploy | Optional (webhook) |

## Success Criteria

- [ ] Production accessible at `https://synkao.com`
- [ ] Staging accessible at `https://staging.synkao.com`
- [ ] SSL certificates valid
- [ ] Health checks passing
- [ ] Can manually trigger deploy
- [ ] Rollback tested

## Monitoring in Coolify

### Logs
- Real-time logs in Coolify UI
- Click project → Logs tab

### Metrics
- CPU, Memory usage
- Container status
- Deployment history

### Alerts
- Settings → Notifications
- Configure Slack/Discord/Email alerts

## Common Issues

### Build fails
- Check Dockerfile path
- Verify build context is root (`.`)
- Review build logs for errors

### Domain not working
- Verify DNS pointing to VPS
- Check SSL certificate status
- Review Traefik logs

### Health check fails
- Ensure `/api/health` endpoint exists
- Container logs for startup errors
- Port mapping correct (3000)

## Next Steps (Post-Deployment)

1. Monitor first few deployments
2. Setup notifications for failures
3. Document deploy process trong team
4. Consider auto-deploy for staging

## Notes

- Coolify stores build cache - subsequent builds faster
- Each deploy creates new container - zero downtime
- Old containers auto-removed after successful deploy
