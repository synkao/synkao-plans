# Phase 01: Thiết lập Next.js 14 + TypeScript + Tailwind

## Context Links

- Plan chính: [plan.md](./plan.md)
- Frontend Spec: [synkao-frontend-spec.md](../../docs/synkao-frontend-spec.md)
- Design System: [synkao-design-system.md](../../docs/synkao-design-system.md)
- Tham khảo Issue: [ISSUES.md #1](../../.claude/skills/synkao-github-skill/ISSUES.md)

## Tổng quan

| Trường | Giá trị |
|--------|---------|
| Độ ưu tiên | P0 |
| Effort | 2h |
| Trạng thái | Done |
| Issue | #1 |

Khởi tạo dự án Next.js 14 với App Router, TypeScript strict mode, Tailwind CSS. Tạo cấu trúc thư mục đầy đủ theo frontend specification.

## Yêu cầu

### Yêu cầu chức năng
- Next.js 14 với App Router (không phải Pages Router)
- TypeScript với strict mode enabled
- Tailwind CSS đã cấu hình
- Cấu trúc thư mục theo frontend spec

### Yêu cầu phi chức năng
- Sử dụng pnpm làm package manager
- Dự án tại `apps/web/` cho cấu trúc monorepo
- ESLint cấu hình với Next.js defaults

## Kiến trúc

```
synkao/
├── apps/
│   └── web/                    # Next.js Frontend
│       ├── src/
│       │   ├── app/            # App Router
│       │   │   ├── (auth)/     # Auth routes (group)
│       │   │   ├── (dashboard)/ # Main app (group)
│       │   │   ├── layout.tsx
│       │   │   ├── page.tsx
│       │   │   └── globals.css
│       │   ├── components/
│       │   │   ├── ui/         # shadcn (tương lai)
│       │   │   └── layout/     # Layout components
│       │   ├── features/       # Feature modules
│       │   │   ├── auth/
│       │   │   ├── design/
│       │   │   └── orders/
│       │   ├── hooks/          # Shared hooks
│       │   ├── lib/            # Utilities
│       │   ├── stores/         # Zustand stores
│       │   ├── types/          # Global types
│       │   └── mocks/          # MSW handlers
│       ├── public/
│       ├── package.json
│       ├── tsconfig.json
│       ├── tailwind.config.ts
│       ├── postcss.config.mjs
│       └── next.config.ts
├── docs/                       # Docs hiện có
└── plans/                      # Plan này
```

## Files cần tạo

| Đường dẫn | Hành động | Mô tả |
|-----------|----------|-------|
| `apps/web/` | Tạo | Next.js project root |
| `apps/web/src/app/layout.tsx` | Sửa | Root layout |
| `apps/web/src/app/page.tsx` | Sửa | Landing page |
| `apps/web/src/app/globals.css` | Sửa | Tailwind imports |
| `apps/web/src/app/(auth)/` | Tạo | Auth route group |
| `apps/web/src/app/(dashboard)/` | Tạo | Dashboard route group |
| `apps/web/src/components/ui/.gitkeep` | Tạo | Placeholder cho shadcn |
| `apps/web/src/components/layout/.gitkeep` | Tạo | Placeholder |
| `apps/web/src/features/auth/.gitkeep` | Tạo | Placeholder |
| `apps/web/src/features/design/.gitkeep` | Tạo | Placeholder |
| `apps/web/src/features/orders/.gitkeep` | Tạo | Placeholder |
| `apps/web/src/hooks/.gitkeep` | Tạo | Placeholder |
| `apps/web/src/lib/utils.ts` | Tạo | cn() helper |
| `apps/web/src/stores/.gitkeep` | Tạo | Placeholder |
| `apps/web/src/types/index.ts` | Tạo | Base types export |
| `apps/web/src/mocks/.gitkeep` | Tạo | Placeholder |
| `apps/web/tsconfig.json` | Sửa | Strict mode |

## Các bước triển khai

### Bước 1: Tạo thư mục apps
```bash
mkdir -p apps
```

### Bước 2: Khởi tạo dự án Next.js
```bash
cd apps
pnpm create next-app@latest web --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --use-pnpm
```
Options:
- TypeScript: Yes
- ESLint: Yes
- Tailwind CSS: Yes
- Thư mục `src/`: Yes
- App Router: Yes
- Import alias: `@/*`

### Bước 3: Bật TypeScript strict mode
Cập nhật `apps/web/tsconfig.json`:
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

### Bước 4: Tạo cấu trúc thư mục
```bash
cd apps/web/src

# App router groups
mkdir -p app/(auth)
mkdir -p app/(dashboard)

# Components
mkdir -p components/ui
mkdir -p components/layout

# Features
mkdir -p features/auth
mkdir -p features/design
mkdir -p features/orders

# Các thư mục khác
mkdir -p hooks
mkdir -p lib
mkdir -p stores
mkdir -p types
mkdir -p mocks
```

### Bước 5: Tạo file placeholder
Tạo `.gitkeep` trong các thư mục trống để giữ cấu trúc.

### Bước 6: Tạo lib/utils.ts
```typescript
// apps/web/src/lib/utils.ts
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

Cài đặt dependencies:
```bash
pnpm add clsx tailwind-merge
```

### Bước 7: Tạo types/index.ts
```typescript
// apps/web/src/types/index.ts
export type {};
// Types sẽ được thêm trong các issues tương lai
```

### Bước 8: Cập nhật globals.css
```css
/* apps/web/src/app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Glass effect base (sử dụng sau) */
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    /* Thêm CSS variables trong Issue #2 */
  }
}
```

### Bước 9: Cập nhật root layout
```tsx
// apps/web/src/app/layout.tsx
import type { Metadata } from "next";
import { Inter } from "next/font/google";
import "./globals.css";

const inter = Inter({ subsets: ["latin"] });

export const metadata: Metadata = {
  title: "Synkao - POD Order Management",
  description: "Design task management for Print-on-Demand",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="vi">
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

### Bước 10: Cập nhật page.tsx
```tsx
// apps/web/src/app/page.tsx
export default function Home() {
  return (
    <main className="flex min-h-screen items-center justify-center">
      <h1 className="text-4xl font-bold">Synkao</h1>
    </main>
  );
}
```

### Bước 11: Xác minh build
```bash
cd apps/web
pnpm build
pnpm dev
```

## Danh sách việc cần làm

- [ ] Tạo thư mục `apps/`
- [ ] Chạy `pnpm create next-app@latest` với options đúng
- [ ] Bật TypeScript strict mode trong tsconfig.json
- [ ] Tạo cấu trúc thư mục (app groups, components, features, v.v.)
- [ ] Tạo file `.gitkeep` placeholder
- [ ] Cài đặt clsx, tailwind-merge dependencies
- [ ] Tạo `lib/utils.ts` với cn() helper
- [ ] Tạo `types/index.ts` placeholder
- [ ] Cập nhật globals.css với Tailwind imports
- [ ] Cập nhật root layout.tsx
- [ ] Cập nhật page.tsx với nội dung cơ bản
- [ ] Chạy build để xác minh không có lỗi
- [ ] Chạy dev server để xác minh app load

## Tiêu chí thành công

- [ ] `pnpm build` pass không lỗi
- [ ] `pnpm dev` start server, app load tại localhost:3000
- [ ] TypeScript strict mode active (xác minh bằng lỗi type cố ý)
- [ ] Tailwind classes hoạt động (xác minh h1 styling)
- [ ] Cấu trúc thư mục đúng theo frontend spec

## Đánh giá rủi ro

| Rủi ro | Khả năng | Tác động | Giải pháp |
|--------|----------|----------|-----------|
| Phiên bản Node không tương thích | Thấp | Cao | Dùng nvm, yêu cầu Node 18+ |
| pnpm chưa cài đặt | Thấp | Thấp | Cài qua npm i -g pnpm |
| Port 3000 đang được dùng | Trung bình | Thấp | Dùng port khác |

## Cân nhắc bảo mật

- Không có secrets hay API keys ở giai đoạn này
- Đảm bảo .gitignore bao gồm node_modules, .next, .env*

## Bước tiếp theo

Sau khi hoàn thành:
1. Close Issue #1 qua `gh issue close 1 --repo synkao/synkao`
2. Commit với `feat(#1): setup Next.js 14 với TypeScript và Tailwind`
3. Tiến hành Issue #2: **Cấu hình shadcn/ui + custom theme (Glass)**
