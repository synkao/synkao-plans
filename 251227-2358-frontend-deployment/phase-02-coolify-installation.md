# Phase 2: Coolify Installation

## Context

- **Parent Plan:** [plan.md](./plan.md)
- **Dependencies:** [Phase 1](./phase-01-docker-setup.md) (Dockerfile ready)
- **Blocks:** Phase 3

## Overview

| Field | Value |
|-------|-------|
| Date | 2025-12-28 |
| Priority | P2 |
| Effort | 30m |
| Status | pending |

**Mục tiêu:** Install Coolify v4 trên VPS và setup initial configuration.

## Requirements

1. VPS với root access
2. Domain pointing to VPS IP
3. Ports 80, 443, 8000 open
4. Minimum: 2GB RAM, 2 vCPU

## VPS Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| RAM | 2GB | 4GB |
| CPU | 2 vCPU | 4 vCPU |
| Disk | 30GB | 50GB |
| OS | Ubuntu 22.04+ | Ubuntu 24.04 |

## DNS Setup (Before Installation)

| Record | Type | Value |
|--------|------|-------|
| synkao.com | A | VPS_IP |
| www.synkao.com | CNAME | synkao.com |
| staging.synkao.com | A | VPS_IP |
| coolify.synkao.com | A | VPS_IP |

## Implementation Steps

### Step 1: SSH to VPS

```bash
ssh root@YOUR_VPS_IP
```

### Step 2: Install Coolify (One Command)

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash
```

Quá trình install sẽ:
- Install Docker
- Install Docker Compose
- Setup Coolify containers
- Configure Traefik reverse proxy
- Generate initial admin credentials

### Step 3: Access Coolify Dashboard

1. Open browser: `http://YOUR_VPS_IP:8000`
2. Create admin account
3. Complete initial setup wizard

### Step 4: Configure Custom Domain for Coolify

Trong Coolify Settings:

1. Go to **Settings** → **Configuration**
2. Set **Instance's Domain**: `coolify.synkao.com`
3. Enable **Auto Update** (recommended)
4. Save changes

### Step 5: Setup SSL for Coolify Dashboard

1. Go to **Settings** → **Configuration**
2. Enable **Let's Encrypt**
3. Enter email for SSL notifications
4. Save and wait for certificate

### Step 6: Connect GitHub

1. Go to **Sources** → **Add New**
2. Select **GitHub App** (recommended)
3. Follow OAuth flow to authorize
4. Select repositories to grant access

**Alternative:** Use Deploy Key per project (no GitHub App needed)

## Post-Installation Verification

```bash
# Check Coolify containers
docker ps | grep coolify

# Check Traefik
docker ps | grep traefik

# Test dashboard
curl -I https://coolify.synkao.com
```

## Success Criteria

- [ ] Coolify dashboard accessible at `https://coolify.synkao.com`
- [ ] Admin account created
- [ ] SSL certificate valid
- [ ] GitHub source connected
- [ ] Can create new project

## Troubleshooting

### Port 8000 not accessible
```bash
# Check firewall
ufw status
ufw allow 8000
ufw allow 80
ufw allow 443
```

### Docker not running
```bash
systemctl status docker
systemctl start docker
```

### SSL certificate failed
- Verify DNS propagation: `dig coolify.synkao.com`
- Check Traefik logs: `docker logs coolify-proxy`

## Coolify Architecture

```
┌─────────────────────────────────────────┐
│              Coolify Stack               │
├─────────────────────────────────────────┤
│  coolify         │ Main application     │
│  coolify-db      │ PostgreSQL database  │
│  coolify-redis   │ Redis cache          │
│  coolify-proxy   │ Traefik reverse proxy│
└─────────────────────────────────────────┘
```

## Next Steps

→ [Phase 3: Coolify Configuration](./phase-03-coolify-configuration.md)

## Notes

- Coolify data stored in `/data/coolify`
- Backup: `tar -czvf coolify-backup.tar.gz /data/coolify`
- Update: Settings → Check for updates
