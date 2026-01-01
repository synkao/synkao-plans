# Brainstorm Report: Frontend Deployment Issue

**Date:** 2025-12-27
**Session ID:** 251227-2254

---

## Problem Statement

Cần setup infrastructure và workflow để deploy Frontend (Next.js) lên VPS với khả năng hỗ trợ cả staging và production environments.

## Requirements Gathered

| Aspect | Decision |
|--------|----------|
| Platform | Self-hosted VPS (local provider) |
| Method | Docker containerization |
| Environments | 2 subdomain (staging.domain + domain) trên cùng VPS |
| Registry | Self-hosted Docker Registry |
| CI/CD | Manual deploy (không auto) |
| Domain | Đã có sẵn |
| SSL | Required (Let's Encrypt) |

## Evaluated Approaches

### Option A: GitHub Actions + GHCR
- **Pros:** Full automation, consistent builds, không cần maintain registry
- **Cons:** Phụ thuộc external service, image public (nếu free)
- **Verdict:** Rejected - User muốn manual control

### Option B: VPS Self-build
- **Pros:** Đơn giản, không cần registry
- **Cons:** VPS tốn resource khi build, slow
- **Verdict:** Not selected

### Option C: Local Build + Self-hosted Registry ✅
- **Pros:** Full control, fast builds trên dev machine
- **Cons:** Manual process, cần maintain registry
- **Verdict:** Selected

## Recommended Solution

### Architecture

```
┌─────────────┐      push      ┌─────────────────────────────────┐
│   Dev PC    │ ────────────▶ │            VPS                   │
│             │               │  ┌─────────────────────────────┐ │
│ docker build│               │  │    Docker Registry          │ │
│ docker push │               │  │    registry.domain.com      │ │
└─────────────┘               │  └─────────────────────────────┘ │
                              │              │                    │
                              │              ▼ pull               │
                              │  ┌─────────────────────────────┐ │
                              │  │    Nginx (Reverse Proxy)    │ │
                              │  │    + SSL (Certbot)          │ │
                              │  └─────────────────────────────┘ │
                              │         │            │           │
                              │         ▼            ▼           │
                              │  ┌───────────┐ ┌───────────┐    │
                              │  │  Staging  │ │Production │    │
                              │  │  :3001    │ │  :3000    │    │
                              │  └───────────┘ └───────────┘    │
                              └─────────────────────────────────┘
```

### Components To Build

1. **Dockerfile** - Multi-stage build cho Next.js
2. **docker-compose.yml** - VPS orchestration
3. **nginx.conf** - Reverse proxy + SSL
4. **deploy.sh** - Build, push, deploy script
5. **Makefile** - Developer-friendly commands

### Issue Content (Proposed)

```markdown
## Deploy Frontend lên VPS

### Mô tả
Setup Docker deployment cho Next.js frontend với:
- Self-hosted Docker Registry
- 2 environments: staging + production
- SSL với Let's Encrypt
- Manual deploy workflow

### Tasks

#### Docker Setup
- [ ] Dockerfile multi-stage cho Next.js
- [ ] .dockerignore
- [ ] docker-compose.local.yml (local dev)

#### VPS Infrastructure
- [ ] docker-compose.prod.yml cho VPS
- [ ] Nginx reverse proxy config
- [ ] SSL setup với Certbot
- [ ] Docker Registry setup

#### Deployment
- [ ] deploy.sh script
- [ ] Makefile targets (make deploy-staging, make deploy-prod)
- [ ] Deployment documentation

### Acceptance Criteria
- [ ] staging.domain.com accessible với HTTPS
- [ ] domain.com accessible với HTTPS
- [ ] Deploy workflow documented
- [ ] Rollback procedure documented
```

## Implementation Considerations

### Risks
1. **Registry security** - Cần auth và SSL cho registry
2. **SSL renewal** - Certbot auto-renewal phải hoạt động
3. **Disk space** - Cần cleanup old images định kỳ

### Mitigations
1. Setup htpasswd auth cho registry
2. Certbot timer systemd
3. Docker prune script trong cron

## Success Metrics

- [ ] Deploy time < 5 phút từ local build đến live
- [ ] Zero downtime deployment với Docker
- [ ] SSL grade A trên ssllabs.com

## Next Steps

1. Tạo issue trên GitHub với nội dung trên
2. (Optional) Tạo implementation plan chi tiết

---

## Unresolved Questions

1. Subdomain cho registry? (Đề xuất: `registry.domain.com`)
2. VPS specs (RAM/CPU) để đảm bảo chạy được registry + 2 app instances?
3. Backup strategy cho registry data?
