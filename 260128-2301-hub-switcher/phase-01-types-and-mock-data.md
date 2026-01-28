# Phase 1: Types & Mock Data

## Context Links
- [Main Plan](./plan.md)
- [Existing types](../../apps/web/src/types/index.ts)
- [Mock factory pattern](../../apps/web/src/mocks/factories/mock-factory.ts)
- [Mock data pattern](../../apps/web/src/mocks/data/workspaces.data.ts)

## Overview
- **Priority**: P1 (Foundation)
- **Status**: ✅ completed
- **Effort**: 0.5h
- **Completed**: 2026-01-28

## Key Insights
- Hub concept differs from existing Workspace (Hub = organization/team, Workspace = design kanban board)
- Need HubType enum for type-based styling
- Follow existing mock factory patterns for ID generation

## Requirements

### Functional
- Hub type definitions matching data model
- Mock seed data for 3 hubs

### Non-functional
- Type-safe interfaces
- Consistent with existing type patterns

## Architecture

```typescript
// Hub type enum
type HubType = 'sale' | 'design' | 'freelance';

// Hub interface
interface Hub {
  id: string;
  name: string;
  slug: string;
  type: HubType;
  memberCount: number;
  avatar?: string; // emoji
}
```

## Related Code Files

### Files to Create
1. `apps/web/src/types/hub.ts` - Hub type definitions
2. `apps/web/src/mocks/data/hubs.data.ts` - Mock hubs seed data

### Files to Modify
1. `apps/web/src/types/index.ts` - Export hub types
2. `apps/web/src/mocks/factories/mock-factory.ts` - Add HUB_IDS constant

## Implementation Steps

### Step 1: Create Hub Type Definitions
Create `apps/web/src/types/hub.ts`:
```typescript
// Hub type enum
export const HubType = {
  SALE: 'sale',
  DESIGN: 'design',
  FREELANCE: 'freelance',
} as const;
export type HubTypeValue = (typeof HubType)[keyof typeof HubType];

// Hub interface
export interface Hub {
  id: string;
  name: string;
  slug: string;
  type: HubTypeValue;
  memberCount: number;
  avatar?: string; // emoji
}
```

### Step 2: Export from Types Index
Update `apps/web/src/types/index.ts`:
```typescript
export * from "./hub";
```

### Step 3: Add Hub IDs to Mock Factory
Update `apps/web/src/mocks/factories/mock-factory.ts`:
```typescript
export const HUB_IDS = {
  saleTeamAbc: '33333333-3333-4333-8333-333333333333',
  designStudioA: '44444444-4444-4444-8444-444444444444',
  freelanceTeamB: '55555555-5555-4555-8555-555555555555',
} as const;
```

### Step 4: Create Mock Hubs Data
Create `apps/web/src/mocks/data/hubs.data.ts`:
```typescript
import type { Hub } from '@/types';
import { HUB_IDS } from '../factories/mock-factory';

export const mockHubs: Hub[] = [
  {
    id: HUB_IDS.saleTeamAbc,
    name: 'Sale Team ABC',
    slug: 'sale-team-abc',
    type: 'sale',
    memberCount: 12,
    avatar: '💼',
  },
  {
    id: HUB_IDS.designStudioA,
    name: 'Design Studio A',
    slug: 'design-studio-a',
    type: 'design',
    memberCount: 8,
    avatar: '🎨',
  },
  {
    id: HUB_IDS.freelanceTeamB,
    name: 'Freelance Team B',
    slug: 'freelance-team-b',
    type: 'freelance',
    memberCount: 5,
    avatar: '🚀',
  },
];

export const findHub = (id: string) => mockHubs.find((h) => h.id === id);
```

## Todo List
- [x] Create hub.ts type definitions
- [x] Export from types/index.ts
- [x] Add HUB_IDS to mock-factory.ts
- [x] Create hubs.data.ts mock data
- [x] Verify TypeScript compilation

## Success Criteria
- [x] Hub and HubType types are properly defined
- [x] Types exported from index.ts
- [x] Mock hubs data matches spec (3 hubs with correct data)
- [x] No TypeScript errors

## Review Notes
All type definitions properly implemented. Clean TypeScript compilation. Mock data follows established patterns.

## Risk Assessment
- **Low risk** - Simple type definitions and mock data
- No external dependencies

## Security Considerations
- None for this phase (mock data only)

## Next Steps
- Phase 2: Create Hub Store with Zustand
