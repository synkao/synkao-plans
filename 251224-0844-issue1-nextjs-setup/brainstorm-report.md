# Báo cáo Brainstorm: Issue #1 - Setup Next.js 14

**Ngày:** 2024-12-24
**Issue:** [#1 - Setup Next.js 14 + TypeScript + Tailwind](https://github.com/synkao/synkao/issues/1)
**Trạng thái:** Hoàn tất lên kế hoạch

---

## Mô tả vấn đề

Thiết lập cấu trúc dự án với Next.js App Router cho hệ thống Synkao - POD Order Management.

## Quyết định đã có

| Khía cạnh | Quyết định | Nguồn |
|-----------|------------|-------|
| UI Components | shadcn/ui | Issue #2 |
| State Management | Zustand | Issue #5 |
| Data Fetching | TanStack Query | Issue #5 |
| API Mocking | MSW | Issue #3 |
| Design Theme | Glass effect | Issue #2 |
| Backend | Golang + MySQL | SKILL.md |

## Các phương án đánh giá

### A. Single Next.js App
- Đơn giản, nhanh setup
- Không phù hợp với cấu trúc `apps/web` + `apps/api`

### B. Monorepo với Turborepo ✅ ĐÃ CHỌN
- Phù hợp với cấu trúc đã plan
- Cho phép share code giữa packages
- Tốt cho team development

### C. Lerna/Nx
- Overkill cho dự án này
- Không cần thiết

## Quyết định cuối cùng: Cấu trúc Monorepo

```
synkao/
├── apps/
│   ├── web/                 # Next.js Frontend
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── (auth)/
│   │   │   │   ├── (dashboard)/
│   │   │   │   │   ├── orders/
│   │   │   │   │   └── design/
│   │   │   │   └── layout.tsx
│   │   │   ├── components/
│   │   │   │   ├── ui/          # shadcn
│   │   │   │   └── layout/
│   │   │   ├── features/
│   │   │   │   ├── orders/
│   │   │   │   └── design/
│   │   │   ├── hooks/
│   │   │   ├── lib/
│   │   │   ├── stores/
│   │   │   └── types/
│   │   └── package.json
│   │
│   └── api/                 # Golang Backend (Issue #27+)
│       ├── cmd/
│       ├── internal/
│       └── go.mod
│
├── packages/                # Shared (tương lai)
│   └── types/
│
├── docker-compose.yml
├── Makefile
├── pnpm-workspace.yaml
└── turbo.json
```

## Checklist triển khai

### 1. Thiết lập Root
- [ ] Init pnpm workspace
- [ ] Tạo turbo.json
- [ ] Tạo root package.json
- [ ] Tạo Makefile

### 2. Next.js App (`apps/web`)
- [ ] `npx create-next-app@latest` với options đúng
- [ ] Cấu hình TypeScript strict mode
- [ ] Tailwind đã được include
- [ ] Tạo cấu trúc thư mục

### 3. Cấu hình TypeScript
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

### 4. Cấu trúc thư mục cần tạo
```
src/
├── components/ui/       # shadcn (Issue #2)
├── components/layout/   # AppShell (Issue #6)
├── features/orders/
├── features/design/
├── hooks/
├── lib/utils.ts
├── stores/
└── types/
```

## Rủi ro & Giải pháp

| Rủi ro | Giải pháp |
|--------|-----------|
| Turborepo learning curve | Dùng config tối thiểu, thêm dần |
| shadcn override tailwind | Setup tối thiểu, Issue #2 sẽ config |
| MSW 2.x với App Router | Xử lý ở Issue #3 |

## Commands

```bash
# Setup workspace
pnpm init
echo 'packages:\n  - "apps/*"\n  - "packages/*"' > pnpm-workspace.yaml

# Tạo Next.js app
cd apps && npx create-next-app@latest web \
  --typescript --tailwind --eslint --app \
  --src-dir --import-alias "@/*" --use-pnpm

# Tạo thư mục
mkdir -p apps/web/src/{components/{ui,layout},features/{orders,design},hooks,lib,stores,types}
mkdir -p apps/api
```

## Tiêu chí thành công

- [ ] `pnpm install` chạy thành công
- [ ] `pnpm dev` start Next.js
- [ ] TypeScript strict mode enabled
- [ ] Cấu trúc thư mục đúng spec
- [ ] Tailwind hoạt động

## Bước tiếp theo

1. Issue #2: Setup shadcn/ui + Glass theme
2. Issue #3: Setup MSW
3. Issue #4: Mock data structure
4. Issue #5: Zustand + TanStack Query

---

## Câu hỏi chưa giải quyết

*Không có - đã làm rõ tất cả*
