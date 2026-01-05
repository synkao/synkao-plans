# Phase 01: Update Types & Constants

## Context

- **Parent Plan:** [plan.md](./plan.md)
- **Dependencies:** None
- **Related Docs:** [Brainstorm Report](../reports/brainstorm-260105-1003-simplify-priority-to-urgent-flag.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-05 |
| Priority | P1 |
| Status | ✅ Done |
| Review | ✅ Completed |
| Effort | 30m |
| Completed | 2026-01-05 |

**Description:** Xóa Priority type/enum và thay bằng isUrgent boolean trong type definitions.

## Key Insights

- Priority được define ở 2 nơi: `src/types/status.ts` và `src/mocks/types.ts`
- Constants liên quan ở: `src/lib/constants.ts`, `src/features/design/lib/constants.ts`
- Cần giữ lại config cho URGENT color/styling

## Related Files

| File | Change Type |
|------|-------------|
| `apps/web/src/types/status.ts` | DELETE Priority type |
| `apps/web/src/lib/constants.ts` | REMOVE PRIORITY_COLORS |
| `apps/web/src/mocks/types.ts` | MODIFY Priority enum → remove, add isUrgent |
| `apps/web/src/features/design/lib/constants.ts` | SIMPLIFY PRIORITY_CONFIG → URGENT_CONFIG |

## Implementation Steps

### Step 1: Update `src/mocks/types.ts`

```typescript
// BEFORE (line 86-92)
export const Priority = {
  LOW: 'LOW',
  NORMAL: 'NORMAL',
  HIGH: 'HIGH',
  URGENT: 'URGENT',
} as const;
export type PriorityType = (typeof Priority)[keyof typeof Priority];

// AFTER - XÓA hoàn toàn enum Priority
// Không cần thay thế, sẽ dùng isUrgent: boolean trực tiếp
```

### Step 2: Update MockTask interface in `src/mocks/types.ts`

```typescript
// BEFORE (line 179)
priority: PriorityType;

// AFTER
isUrgent: boolean;
```

### Step 3: Update `src/types/status.ts`

```typescript
// BEFORE (line 10)
export type Priority = "LOW" | "NORMAL" | "HIGH" | "URGENT";

// AFTER - XÓA hoàn toàn dòng này
```

### Step 4: Update `src/lib/constants.ts`

```typescript
// BEFORE (line 28-34)
export const PRIORITY_COLORS = {
  URGENT: { text: "text-red-500", bg: "bg-red-50" },
  HIGH: { text: "text-amber-500", bg: "bg-amber-50" },
  NORMAL: { text: "text-blue-500", bg: "bg-blue-50" },
  LOW: { text: "text-gray-500", bg: "bg-gray-100" },
} as const;

// AFTER
export const URGENT_STYLE = {
  text: "text-red-500",
  bg: "bg-red-50",
} as const;
```

### Step 5: Update `src/features/design/lib/constants.ts`

```typescript
// BEFORE (line 23-37)
export interface PriorityConfig { ... }
export const PRIORITY_CONFIG: Record<PriorityType, PriorityConfig> = { ... };
export const getPriorityConfig = (priority: PriorityType) => PRIORITY_CONFIG[priority];

// AFTER
export const URGENT_CONFIG = {
  label: 'Urgent',
  color: 'red',
  bgClass: 'bg-red-50',
  textClass: 'text-red-600',
  icon: Flame,
} as const;
```

## Todo List

- [ ] Update `src/mocks/types.ts` - xóa Priority enum
- [ ] Update MockTask interface - đổi priority → isUrgent
- [ ] Update `src/types/status.ts` - xóa Priority type
- [ ] Update `src/lib/constants.ts` - đổi PRIORITY_COLORS → URGENT_STYLE
- [ ] Update `src/features/design/lib/constants.ts` - đổi PRIORITY_CONFIG → URGENT_CONFIG
- [ ] Verify TypeScript compilation

## Success Criteria

- [ ] Không còn reference nào đến `Priority` type/enum
- [ ] `isUrgent: boolean` được define trong MockTask
- [ ] URGENT_CONFIG/URGENT_STYLE available cho components
- [ ] `pnpm typecheck` pass

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| TypeScript errors cascade | High | Medium | Fix imports incrementally |

## Security Considerations

None - type refactoring only.

## Next Steps

After completion → Phase 02: Update Mock Data
