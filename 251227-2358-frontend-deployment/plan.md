---
title: "Deploy Frontend to VPS with Coolify"
description: "Setup Next.js frontend deployment via Coolify PaaS on VPS"
status: pending
priority: P2
effort: 2h
branch: main
tags: [docker, deployment, frontend, coolify, ssl]
created: 2025-12-27
updated: 2025-12-28
---

# Deploy Frontend to VPS with Coolify

## Overview

Triển khai Next.js frontend lên VPS sử dụng **Coolify** - self-hosted PaaS platform.

**Environments:**
- Production: `synkao.com`
- Staging: `staging.synkao.com`
- Coolify Dashboard: `coolify.synkao.com`

**Architecture:**
```
GitHub Repo → Coolify (auto pull) → Build → Deploy
                    │
                    ├── SSL (auto Let's Encrypt)
                    ├── Reverse Proxy (Traefik)
                    └── Container Management
```

## Phases

| Phase | File | Effort | Description |
|-------|------|--------|-------------|
| 1 | [phase-01-docker-setup.md](./phase-01-docker-setup.md) | 30m | Dockerfile + .dockerignore |
| 2 | [phase-02-coolify-installation.md](./phase-02-coolify-installation.md) | 30m | Install Coolify on VPS |
| 3 | [phase-03-coolify-configuration.md](./phase-03-coolify-configuration.md) | 1h | Configure projects in Coolify |

## Why Coolify?

| Feature | Manual Docker | Coolify |
|---------|---------------|---------|
| SSL | Certbot manual | Auto Let's Encrypt |
| Reverse Proxy | Nginx config | Traefik auto |
| Deployments | Scripts | Git push / UI click |
| Rollback | Manual | One-click UI |
| Logs | CLI | Web dashboard |
| Monitoring | Self-setup | Built-in |

## Key Decisions

1. **Platform:** Coolify v4 (self-hosted PaaS)
2. **Dockerfile:** Multi-stage build với standalone output
3. **SSL:** Auto via Coolify + Let's Encrypt
4. **Deploy trigger:** Manual from Coolify UI (hoặc webhook)

## Files to Create

```
apps/web/
├── Dockerfile
└── .dockerignore
```

**Không cần:** Nginx configs, deploy scripts, Makefile targets, Registry setup

## Research Reports

- [Next.js Docker Best Practices](./research/researcher-nextjs-docker.md)
- [Registry + Nginx + SSL](./research/researcher-registry-nginx-ssl.md) (reference only)

## Success Criteria

- [ ] Coolify dashboard accessible tại `coolify.synkao.com`
- [ ] Production deploy thành công tại `synkao.com`
- [ ] Staging deploy thành công tại `staging.synkao.com`
- [ ] SSL auto-renew hoạt động
- [ ] Manual deploy from Coolify UI works

## Validation Summary

**Validated:** 2025-12-28
**Questions asked:** 7

### Confirmed Decisions

| Decision | Choice |
|----------|--------|
| Platform | **Coolify** (thay đổi từ manual Docker) |
| Health check | Simple 200 OK |
| VPS path | Coolify manages (default /data/coolify) |
| API URLs | staging-api.synkao.com + api.synkao.com |
| Monitoring | Coolify built-in |
| CI/CD | Coolify webhook / manual deploy |

### Plan Changes

- [x] Switched from manual Docker → Coolify
- [x] Removed: Nginx configs, Certbot, Registry, deploy scripts
- [x] Simplified: 3 phases, ~2h total effort
