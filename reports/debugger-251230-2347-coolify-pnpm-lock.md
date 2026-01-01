# Debug Report: Coolify Build Failed - pnpm-lock.yaml Not Found

**Date**: 2025-12-30 23:47
**Issue**: Docker build error `#8 ERROR: failed to calculate checksum of ref ... "/pnpm-lock.yaml": not found`
**Impact**: Không deploy được lên Coolify

---

## 1. Root Cause

**pnpm-lock.yaml bị ignore trong git repository**

File `pnpm-lock.yaml` tồn tại local nhưng không được commit vào git vì:
- `.gitignore` line 41: `pnpm-lock.yaml` (block tất cả lock files)
- Khi Coolify clone repo từ GitHub → không có `pnpm-lock.yaml`
- Dockerfile line 9 `COPY pnpm-lock.yaml` → FAIL

---

## 2. Evidence

### 2.1. File Status
```bash
# File tồn tại locally
$ ls -la pnpm-lock.yaml
-rw-r--r-- 1 tuntran tuntran 150294 Dec 25 15:47 pnpm-lock.yaml

# Nhưng KHÔNG tracked by git
$ git ls-files | grep pnpm-lock.yaml
(no output)
```

### 2.2. .gitignore Configuration
```gitignore
# Line 38-41
# package manager
package-lock.json
yarn.lock
pnpm-lock.yaml  ← CAUSE: Block lock file
```

### 2.3. Dockerfile Requirements
```dockerfile
# Stage 1: deps - Line 9
COPY pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/web/package.json ./apps/web/

# Stage 2: builder - Line 26
COPY pnpm-lock.yaml pnpm-workspace.yaml turbo.json ./
```

**Dockerfile cần pnpm-lock.yaml ở 2 stages** nhưng file không có trong git.

### 2.4. Recent History
```
048f5d2 Revert "fix: add root .dockerignore for Coolify build"
dd4eaf8 fix: add root .dockerignore for Coolify build
```

User đã thử fix bằng root `.dockerignore` nhưng revert lại → không giải quyết vấn đề gốc.

---

## 3. Recommended Fixes

### Option 1: Commit pnpm-lock.yaml (RECOMMENDED)
**Tại sao**: Lock file nên được commit để đảm bảo reproducible builds.

**Steps**:
1. Remove line 41 trong `.gitignore`:
   ```diff
   # package manager
   package-lock.json
   yarn.lock
   - pnpm-lock.yaml
   ```

2. Commit lock file:
   ```bash
   git add pnpm-lock.yaml
   git commit -m "fix: track pnpm-lock.yaml for reproducible builds"
   git push
   ```

**Pros**:
- ✅ Chuẩn best practice (pnpm docs recommend commit lock file)
- ✅ Reproducible builds across environments
- ✅ Dependency version consistency
- ✅ Fix vĩnh viễn cho mọi deployment platform

**Cons**:
- ❌ Lock file thay đổi thường xuyên → nhiều merge conflicts (minor)

---

### Option 2: Generate lock file trong build
**Tại sao**: Tránh commit lock file nhưng phải modify Dockerfile.

**Steps**:
Thay đổi Dockerfile stage 1:
```dockerfile
FROM node:22-alpine AS deps
WORKDIR /app

RUN corepack enable && corepack prepare pnpm@10.26.1 --activate

COPY pnpm-workspace.yaml ./
COPY apps/web/package.json ./apps/web/

# Generate lock file from package.json
RUN pnpm install --frozen-lockfile=false --filter web
```

**Pros**:
- ✅ Không cần commit lock file

**Cons**:
- ❌ KHÔNG reproducible (mỗi lần build có thể pull versions khác)
- ❌ Chậm hơn (resolve dependencies mỗi lần)
- ❌ Risk security issues (dependencies change unexpectedly)
- ❌ Vi phạm Docker best practices

---

### Option 3: Use .dockerignore allowlist
**Tại sao**: Keep lock file in git-ignore nhưng allow trong Docker context.

**Steps**:
1. Create root `.dockerignore`:
   ```dockerignore
   # Start with ignore all
   *

   # Then allowlist needed files
   !pnpm-lock.yaml
   !pnpm-workspace.yaml
   !turbo.json
   !apps/web/
   ```

**Pros**:
- ✅ Control chính xác files trong Docker context

**Cons**:
- ❌ Vẫn cần local có pnpm-lock.yaml
- ❌ Coolify clone từ git → VẪN FAIL vì git không có file
- ❌ KHÔNG GIẢI QUYẾT được vấn đề

---

## 4. Architecture Analysis

### Coolify Build Flow
```
GitHub Repo (no pnpm-lock.yaml)
    ↓ git clone
Coolify Build Server
    ↓ docker build
COPY pnpm-lock.yaml → ERROR: not found
```

### Why .dockerignore Won't Help
- `.dockerignore` chỉ filter files **ĐÃ CÓ** trong build context
- Build context = files from git clone
- Git clone không có `pnpm-lock.yaml` → `.dockerignore` vô dụng

### Root Issue
- **Development**: File tồn tại, build OK
- **Production**: Clone từ git, file bị ignore → build FAIL
- **Gap**: .gitignore conflict với Dockerfile requirements

---

## 5. Verification Checklist

Sau khi apply fix, verify:

```bash
# 1. Check file được tracked
git ls-files | grep pnpm-lock.yaml

# 2. Check file có trên remote
git ls-tree origin/main pnpm-lock.yaml

# 3. Test fresh clone
cd /tmp
git clone https://github.com/synkao/synkao.git test-clone
cd test-clone
ls -la pnpm-lock.yaml  # Should exist

# 4. Test Docker build locally
docker build -f apps/web/Dockerfile -t test .

# 5. Push và trigger Coolify build
```

---

## 6. Related Files

- `/home/tuntran/application/github.com/synkao/synkao/.gitignore` (line 41)
- `/home/tuntran/application/github.com/synkao/synkao/apps/web/Dockerfile` (lines 9, 26)
- `/home/tuntran/application/github.com/synkao/synkao/pnpm-lock.yaml` (exists locally, not in git)

---

## 7. Recommendation

**→ Apply Option 1: Commit pnpm-lock.yaml**

**Rationale**:
1. Industry standard (npm, yarn, pnpm đều recommend)
2. Eliminates "works on my machine" issues
3. Proper fix, không workaround
4. CI/CD best practice
5. pnpm official docs: "Lockfiles should be committed to version control"

**Priority**: HIGH (blocks deployment)
**Complexity**: LOW (1 file change, 2 commands)
**Risk**: MINIMAL

---

## Unresolved Questions

1. Tại sao lock file ban đầu được add vào .gitignore? (Check git history để hiểu context)
2. Có tools/scripts nào khác phụ thuộc vào việc lock file không được tracked không?
3. Team có policy về lock files không? (Có thể cần confirm trước khi commit)
