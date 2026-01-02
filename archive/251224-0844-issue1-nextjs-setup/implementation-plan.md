# Kế hoạch triển khai: Issue #1

**Issue:** #1 - Setup Next.js 14 + TypeScript + Tailwind
**Ước tính:** 2h
**Độ ưu tiên:** P0

---

## Phase 1: Thiết lập Root Monorepo (15 phút)

### Bước 1.1: Init pnpm workspace
```bash
pnpm init
```

### Bước 1.2: Tạo workspace config
File: `pnpm-workspace.yaml`
```yaml
packages:
  - "apps/*"
  - "packages/*"
```

### Bước 1.3: Tạo turbo.json
```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "dev": {
      "cache": false,
      "persistent": true
    },
    "build": {
      "outputs": [".next/**", "!.next/cache/**"]
    },
    "lint": {}
  }
}
```

### Bước 1.4: Cập nhật root package.json
```json
{
  "name": "synkao",
  "private": true,
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "lint": "turbo lint"
  },
  "devDependencies": {
    "turbo": "^2"
  }
}
```

---

## Phase 2: Tạo Next.js App (30 phút)

### Bước 2.1: Tạo thư mục apps
```bash
mkdir -p apps
```

### Bước 2.2: Tạo Next.js app
```bash
cd apps && npx create-next-app@latest web \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*" \
  --use-pnpm
```

Options:
- ✅ TypeScript
- ✅ ESLint
- ✅ Tailwind CSS
- ✅ Thư mục `src/`
- ✅ App Router
- ❌ Turbopack (tuỳ chọn)
- ✅ Import alias `@/*`

---

## Phase 3: TypeScript Strict Mode (10 phút)

### Bước 3.1: Cập nhật tsconfig.json
File: `apps/web/tsconfig.json`
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true,
    // ... giữ các options có sẵn
  }
}
```

---

## Phase 4: Cấu trúc thư mục (15 phút)

### Bước 4.1: Tạo các thư mục
```bash
cd apps/web/src

mkdir -p components/ui
mkdir -p components/layout
mkdir -p features/orders/components
mkdir -p features/orders/hooks
mkdir -p features/design/components
mkdir -p features/design/hooks
mkdir -p hooks
mkdir -p lib
mkdir -p stores
mkdir -p types
```

### Bước 4.2: Tạo file placeholder

**lib/utils.ts**
```typescript
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

**types/index.ts**
```typescript
// Global types - sẽ được thêm khi build
export type {};
```

**stores/.gitkeep**
```
# Zustand stores - Issue #5
```

---

## Phase 5: Tạo Placeholder cho API (5 phút)

```bash
mkdir -p apps/api
echo "# Synkao API\n\nGolang backend - bắt đầu từ Issue #27" > apps/api/README.md
```

---

## Phase 6: Root Files (10 phút)

### Makefile
```makefile
.PHONY: dev build lint install

install:
	pnpm install

dev:
	pnpm dev

build:
	pnpm build

lint:
	pnpm lint

# Tương lai: API commands
api-dev:
	cd apps/api && go run cmd/main.go
```

### .nvmrc
```
20
```

### .gitignore (cập nhật root)
```gitignore
node_modules
.next
.turbo
dist
.env*.local
```

---

## Phase 7: Xác minh Setup (15 phút)

### Bước 7.1: Cài đặt dependencies
```bash
pnpm install
```

### Bước 7.2: Chạy dev server
```bash
pnpm dev
```

### Bước 7.3: Checklist xác minh
- [ ] http://localhost:3000 load được
- [ ] Không có lỗi TypeScript
- [ ] Tailwind styles hoạt động
- [ ] Cấu trúc thư mục đã tạo

---

## Sản phẩm bàn giao

| Mục | Trạng thái |
|-----|------------|
| pnpm workspace | ⬜ |
| turbo.json | ⬜ |
| Next.js app đã tạo | ⬜ |
| TypeScript strict | ⬜ |
| Cấu trúc thư mục | ⬜ |
| Dev server chạy | ⬜ |

---

## Dependencies cho Issues tiếp theo

| Issue | Phụ thuộc vào |
|-------|---------------|
| #2 shadcn/ui | Issue này hoàn thành |
| #3 MSW | Issue này hoàn thành |
| #5 Zustand | Issue này hoàn thành |
| #6 Layout | #2 hoàn thành |
