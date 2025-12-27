---
title: "Setup Next.js 14 + TypeScript + Tailwind"
description: "Khởi tạo dự án Next.js 14 với App Router, TypeScript strict mode, Tailwind CSS, và cấu trúc thư mục theo frontend spec"
status: done
completed: 2025-12-24
priority: P0
effort: 2h
issue: 1
branch: main
tags: [fe, setup, nextjs, typescript, tailwind]
created: 2025-12-24
---

# Issue #1: Setup Next.js 14 + TypeScript + Tailwind

## Tổng quan

Khởi tạo dự án frontend Synkao với Next.js 14 App Router, TypeScript strict mode, Tailwind CSS. Tạo cấu trúc thư mục theo [synkao-frontend-spec.md](../../docs/synkao-frontend-spec.md).

## Các phase

| # | Phase | Trạng thái | Effort | Link |
|---|-------|------------|--------|------|
| 1 | Thiết lập dự án & Cấu trúc thư mục | Done | 2h | [phase-01](./phase-01-nextjs-setup.md) |

## Dependencies

- Node.js 18+ (khuyên dùng LTS)
- npm/pnpm/yarn

## Quyết định chính

| Quyết định | Lựa chọn | Lý do |
|------------|----------|-------|
| Package Manager | pnpm | Nhanh hơn, tiết kiệm dung lượng |
| Vị trí dự án | `apps/web/` | Cấu trúc monorepo cho backend tương lai |
| Router | App Router | Mặc định Next.js 14, hỗ trợ RSC |
| Styling | Tailwind CSS | Tương thích design system, utility-first |

## Tài liệu liên quan

- [Frontend Spec](../../docs/synkao-frontend-spec.md)
- [Design System](../../docs/synkao-design-system.md)
- [PRD v2](../../docs/synkao-prd-v2.md)
- [ISSUES.md](../../.claude/skills/synkao-github-skill/ISSUES.md)

## Issue tiếp theo

Sau khi hoàn thành: **#2 - Cấu hình shadcn/ui + custom theme (Glass)**
