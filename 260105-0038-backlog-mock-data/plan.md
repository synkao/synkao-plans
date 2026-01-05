---
title: "Expand Backlog Mock Data"
description: "Tăng mock data từ 15 lên 40 backlog tasks để test pagination"
status: completed
priority: P2
effort: 30m
branch: main
tags: [mock-data, testing, pagination]
created: 2026-01-05
---

# Expand Backlog Mock Data

## Overview

Tăng số lượng tasks trong backlog từ 15 lên ~40 để test pagination với perPage=20.

## Current State

| Data | Count |
|------|-------|
| Orders | 15 |
| Order Items | 45 (3/order) |
| Tasks | 45 (1:1 với items) |
| Backlog (TODO) | 15 |

## Target State

| Data | Count |
|------|-------|
| Orders | 20 |
| Order Items | 60 (3/order) |
| Tasks | 60 |
| Backlog (TODO) | 40 |

## Phases

| # | Phase | Status | Effort | Link |
|---|-------|--------|--------|------|
| 1 | Expand Mock Data | Pending | 30m | [phase-01](./phase-01-expand-mock-data.md) |

## Files to Modify

1. `apps/web/src/mocks/data/orders.data.ts` - Thêm 5 orders (15→20)
2. `apps/web/src/mocks/data/tasks.data.ts` - Điều chỉnh distribution (TODO: 40)

## Technical Approach

Thay đổi `statusDistribution` trong tasks.data.ts:
```ts
// OLD: TODO:15, IN_PROGRESS:10, REVIEW:8, REVISION:5, APPROVED:7 = 45
// NEW: TODO:40, IN_PROGRESS:8, REVIEW:6, REVISION:3, APPROVED:3 = 60
```
