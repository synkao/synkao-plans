# Research: Self-Hosted Docker Registry with Nginx & SSL (2025)

**Date:** 2025-12-27 | **Sources:** 4 web searches | **Status:** Current

## Executive Summary

Docker Registry v2 paired with Nginx reverse proxy and Let's Encrypt SSL provides production-grade container image distribution. Key shift in 2025: major cloud registries (Google Container Registry, Bitnami) sunsetting by Q1-Q2, making self-hosted solutions critical. Recommended stack: Registry v2.18+, Nginx with SSL termination, certbot for cert renewal, htpasswd auth. Estimated setup time: 1-2 hours.

## Key Findings

### 1. Docker Registry v2 Core Setup

**Current Version:** registry:3 (latest stable)

Environment variables replace YAML config in containerized deployments:
- `REGISTRY_AUTH=htpasswd` → Enable authentication
- `REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd` → Auth file location
- `REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm` → Auth prompt text
- `REGISTRY_STORAGE_DELETE_ENABLED=true` → Allow image deletion

Basic deployment: `docker run -d -p 5000:5000 --restart always registry:3`

**Configuration Methods:**
- Environment variables (containers)
- YAML config file (advanced)
- Nginx proxy offloads TLS overhead (recommended)

**2025 Context:** Cloud registry deprecations (Google GCR March 2025, Bitnami August 2025) drive adoption.

### 2. Nginx Reverse Proxy & SSL/TLS

**Recommended Pattern:** Nginx handles TLS termination, proxies to registry on localhost:5000

**Standard Nginx Config Block:**
```nginx
server {
  listen 443 ssl http2;
  server_name registry.example.com;

  ssl_certificate /etc/letsencrypt/live/registry.example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/registry.example.com/privkey.pem;
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers HIGH:!aNULL:!MD5;
  ssl_stapling on;

  location / {
    proxy_pass http://localhost:5000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  location /.well-known/acme-challenge/ {
    root /var/www/certbot;
  }
}

server {
  listen 80;
  server_name registry.example.com;
  location /.well-known/acme-challenge/ {
    root /var/www/certbot;
  }
  location / {
    return 301 https://$server_name$request_uri;
  }
}
```

**TLS Version:** Enforce TLSv1.2+ only. OSCP stapling enabled by default.

### 3. Let's Encrypt & Certbot Integration

**Certificate Lifetime:** 90 days. Recommend renewal after 60 days.

**Certbot Issuance:**
```bash
docker run --rm -v /etc/letsencrypt:/etc/letsencrypt \
  -v /var/www/certbot:/var/www/certbot \
  certbot/certbot certonly --webroot \
  --webroot-path=/var/www/certbot \
  --email admin@example.com \
  --agree-tos -d registry.example.com
```

**Automated Renewal:** Certbot loop in container restarts every 12 hours, renewing eligible certs. Cron alternative: `docker-compose exec certbot renew && docker-compose kill -s SIGHUP nginx` (every 60 days).

**Challenge Mechanism:** Certbot places token in `/.well-known/acme-challenge/`. ACME validation requires outbound port 443 access.

**Volume Structure:**
```
./docker-compose.yml
./nginx/conf.d/registry.conf
./certbot/conf/ ← Certs stored here
./certbot/www/ ← Challenge tokens
```

### 4. Authentication (htpasswd)

**User Management:**
```bash
htpasswd -bnB htpasswd username password > auth/htpasswd
htpasswd -bB auth/htpasswd newuser newpass
```

Flags: `-b` (batch), `-n` (display), `-B` (bcrypt encryption).

**Registry Config:**
```yaml
auth:
  htpasswd:
    realm: "Registry"
    path: /auth/htpasswd
```

**Security Notes:**
- TLS mandatory (prevents MITM of credentials)
- Bcrypt hashing default
- Nginx can add rate limiting for brute force protection
- Token-based auth (OAuth2) available but complex for basic setups
- LDAP/AD integration requires auth middleware

### 5. Image Cleanup & Garbage Collection

**Enable Deletion First:**
```bash
REGISTRY_STORAGE_DELETE_ENABLED=true
```

**Garbage Collection Command:**
```bash
docker exec registry registry garbage-collect /etc/docker/registry/config.yml
# Dry run: add -d flag
docker exec registry registry garbage-collect -d /etc/docker/registry/config.yml
```

**Process:**
1. Delete unused manifests/tags (API only removes manifests, not tags)
2. Run GC to remove orphaned blobs
3. Registry requires read-only mode for safe GC (brief downtime)

**Benefits:** Storage reduction up to 60% (AWS 2025 benchmark), improved performance, reduced operational cost.

**Safety:** Stop registry → run GC → restart OR switch to read-only mode during GC.

## Recommended Architecture (2025)

```
┌─────────────────────────────────────────┐
│ External Network (Docker clients)       │
└────────────┬────────────────────────────┘
             │ :443 (TLS)
             ↓
┌─────────────────────────────────────────┐
│ Nginx Container                         │
│ - SSL Termination (certbot)            │
│ - Basic Auth (if needed in Nginx)      │
│ - Rate Limiting                        │
│ - HTTP/2 support                       │
└─────────────┬───────────────────────────┘
              │ :5000 (internal, plain HTTP)
              ↓
┌─────────────────────────────────────────┐
│ Registry v3 Container                   │
│ - htpasswd auth                        │
│ - S3/MinIO backend (optional)          │
│ - GC enabled                           │
│ - Delete enabled                       │
└─────────────────────────────────────────┘
```

## Security Best Practices (2025)

1. **TLS Only:** Never expose registry on plain HTTP
2. **Certificate Management:** Automate renewal to prevent lapses
3. **Strong Credentials:** Use bcrypt-hashed passwords, rotate periodically
4. **Rate Limiting:** Add Nginx `limit_req` for auth endpoints
5. **Storage Backend:** Use S3/MinIO for distributed setups, not local filesystem
6. **Logs:** Monitor registry logs for unauthorized attempts
7. **Firewall:** Restrict registry.example.com to known CI/CD systems
8. **CNCF Distribution:** Registry is CNCF project (since 2019), stable for production

## Unresolved Questions

- Does project require token auth (OAuth2) or is htpasswd sufficient?
- Storage capacity requirements and growth projections?
- CI/CD pipeline authentication method (service accounts vs. API tokens)?
- Backup/disaster recovery strategy for image data?
- Multi-region registry replication needed?

## References

- [Docker Registry Setup 2025](https://dasroot.net/posts/2025/12/docker-registry-setup-and-image/)
- [CNCF Distribution Config](https://distribution.github.io/distribution/about/configuration/)
- [Docker Registry Image](https://hub.docker.com/_/registry)
- [Nginx + Certbot Reverse Proxy](https://medium.com/@dinusai05/setting-up-a-secure-reverse-proxy-with-https-using-docker-compose-nginx-and-certbot-lets-encrypt-cfd012c53ca0)
- [Docker Registry Garbage Collection](https://docs.docker.com/registry/garbage-collection/)
- [Docker Registry Cleanup Guide](https://github.com/lazyfrosch/docker-registry-cleanup)
- [htpasswd Authentication](https://learning-ocean.com/tutorials/docker/docker-docker-registry-basic-authentication/)
