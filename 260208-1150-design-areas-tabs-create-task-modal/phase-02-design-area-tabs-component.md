# Phase 02: Design Area Tabs Component

## Context Links

- [Reference screenshot 2](/tmp/comment2.png) - Shows tabs per item card
- [Constants - DESIGN_POSITION_OPTIONS](../../apps/web/src/features/orders/lib/constants.ts)
- [DesignUploadModal](../../apps/web/src/features/orders/components/order-detail/design-upload-modal.tsx)
- [Phase 01 - Data Model](./phase-01-data-model-mock-updates.md)

## Overview

- **Priority:** P2
- **Status:** completed
- **Description:** Create `DesignAreaTabs` component showing horizontal row of design position tabs per order item

## Key Insights

From reference screenshot:
- "DESIGN AREAS" label above tab row
- Tabs are pill-shaped buttons in a horizontal row
- **Active tabs** (position has entry in `designAreas`): blue border + blue text (e.g., "Front", "Back")
- **Inactive tabs** (position absent from `designAreas`): gray border + gray text
- Clicking inactive tab opens `DesignUploadModal` with position context
- Item 3 (Premium Canvas Artwork) shows all 11+ positions including: Front, Back, Left, Right, Sleeve, Hood, Pocket, LeftChest, RightChest, SleeveLeft, SleeveRight
- Tabs are horizontally scrollable when they overflow

From codebase:
- `DESIGN_POSITION_OPTIONS` has 11 positions but screenshot shows more (LeftChest, RightChest, SleeveLeft, SleeveRight, Hood)
- May need to extend `DESIGN_POSITION_OPTIONS` with additional positions from screenshot

## Requirements

### Functional
- Display "DESIGN AREAS" label above tab row
- Render tab for each position from `DESIGN_POSITION_OPTIONS`
- Active state (blue) when position key exists in `designAreas`
- Inactive state (gray) for positions not in `designAreas`
- Click inactive tab → opens `DesignUploadModal` with `position` prop
- Click active tab → could show design info or re-upload (toast for now)
- Horizontal scroll when tabs overflow container

### Non-Functional
- Component under 200 lines
- Accessible: keyboard navigable, proper ARIA labels
- Responsive: scrollable on mobile

## Architecture

```
DesignAreaTabs
  Props:
    designAreas: Partial<Record<DesignPositionType, DesignFileInfo | null>> | undefined
    itemId: string
    itemName: string
    onUploadComplete?: (file: File, position: string) => void

  Renders:
    <div label="DESIGN AREAS">
      <div scrollable-row>
        {DESIGN_POSITION_OPTIONS.map(pos => (
          <TabButton
            active={pos.value in designAreas}
            onClick → inactive: open DesignUploadModal
                    → active: toast or view action
          />
        ))}
      </div>
    </div>
```

## Related Code Files

### Files to Create
- `apps/web/src/features/orders/components/order-detail/design-area-tabs.tsx`

### Files to Modify
- `apps/web/src/features/orders/components/order-detail/index.ts` - Add export
- `apps/web/src/features/orders/lib/constants.ts` - Potentially extend `DESIGN_POSITION_OPTIONS` with extra positions from screenshot

## Implementation Steps

### 1. Evaluate extending DESIGN_POSITION_OPTIONS
Screenshot shows positions not in current list: Hood, LeftChest, RightChest, SleeveLeft, SleeveRight. Decide:
- **Option A:** Add these 5 positions to `DESIGN_POSITION_OPTIONS` (total 16)
- **Option B:** Keep 11, screenshot shows future state
- **Recommended:** Option A - extend to match UI spec. Add: HOOD, LEFT_CHEST, RIGHT_CHEST, SLEEVE_LEFT, SLEEVE_RIGHT

Update `constants.ts`:
```typescript
export const DESIGN_POSITION_OPTIONS = [
  { value: 'FRONT', label: 'Front' },
  { value: 'BACK', label: 'Back' },
  { value: 'LEFT', label: 'Left' },
  { value: 'RIGHT', label: 'Right' },
  { value: 'SLEEVE', label: 'Sleeve' },
  { value: 'HOOD', label: 'Hood' },
  { value: 'POCKET', label: 'Pocket' },
  { value: 'LEFT_CHEST', label: 'LeftChest' },
  { value: 'RIGHT_CHEST', label: 'RightChest' },
  { value: 'SLEEVE_LEFT', label: 'SleeveLeft' },
  { value: 'SLEEVE_RIGHT', label: 'SleeveRight' },
  { value: 'COLLAR', label: 'Collar' },
  { value: 'FULL_BODY', label: 'Full Body' },
  { value: 'TOP', label: 'Top' },
  { value: 'BOTTOM', label: 'Bottom' },
] as const;
```

Also update `DesignPositionType` accordingly (if in mocks/types.ts, sync the union).

### 2. Create DesignAreaTabs component

File: `apps/web/src/features/orders/components/order-detail/design-area-tabs.tsx`

```tsx
'use client';

import { useState } from 'react';
import { cn } from '@/lib/utils';
import { DESIGN_POSITION_OPTIONS } from '../../lib/constants';
import type { DesignPositionType } from '../../lib/constants';
import type { DesignFileInfo } from '../../types';
import { DesignUploadModal } from './design-upload-modal';
import { toast } from 'sonner';

export interface DesignAreaTabsProps {
  designAreas?: Partial<Record<DesignPositionType, DesignFileInfo | null>>;
  itemId: string;
  itemName: string;
  onUploadComplete?: (file: File, position: string) => void;
}

export function DesignAreaTabs({
  designAreas,
  itemId,
  itemName,
  onUploadComplete,
}: DesignAreaTabsProps) {
  // Don't render if no designAreas
  if (!designAreas) return null;

  const isActive = (position: string) => position in (designAreas ?? {});

  return (
    <div className="mt-3">
      <p className="text-xs text-gray-500 uppercase tracking-wider mb-2">
        Design Areas
      </p>
      <div className="flex gap-1.5 overflow-x-auto pb-1">
        {DESIGN_POSITION_OPTIONS.map((pos) => {
          const active = isActive(pos.value);
          if (active) {
            return (
              <button
                key={pos.value}
                className={cn(
                  'px-3 py-1 rounded-full text-xs font-medium border whitespace-nowrap',
                  'border-blue-500 bg-blue-50 text-blue-700'
                )}
                onClick={() => toast.info(`View ${pos.label} design`)}
              >
                {pos.label}
              </button>
            );
          }
          // Inactive → wrap with DesignUploadModal
          return (
            <DesignUploadModal
              key={pos.value}
              itemId={itemId}
              itemName={itemName}
              position={pos.label}
              trigger={
                <button
                  className={cn(
                    'px-3 py-1 rounded-full text-xs font-medium border whitespace-nowrap',
                    'border-gray-300 text-gray-500 hover:border-gray-400'
                  )}
                >
                  {pos.label}
                </button>
              }
              onUploadComplete={(file) => {
                onUploadComplete?.(file, pos.value);
              }}
            />
          );
        })}
      </div>
    </div>
  );
}
```

### 3. Export from index
In `apps/web/src/features/orders/components/order-detail/index.ts`, add:
```typescript
export { DesignAreaTabs } from './design-area-tabs';
export type { DesignAreaTabsProps } from './design-area-tabs';
```

## Todo List

- [ ] Evaluate and extend `DESIGN_POSITION_OPTIONS` with additional positions from screenshot
- [ ] Update `DesignPositionType` union accordingly
- [ ] Create `design-area-tabs.tsx` component
- [ ] Style: active (blue border/bg/text), inactive (gray border/text)
- [ ] Horizontal scroll with `overflow-x-auto`
- [ ] Inactive tab click opens `DesignUploadModal` with position
- [ ] Active tab click shows toast (placeholder for view action)
- [ ] Export from `index.ts`
- [ ] Verify compile

## Success Criteria

- Tabs render for items with `designAreas`
- No tabs render for items without `designAreas`
- Active tabs blue, inactive tabs gray
- Clicking inactive tab opens upload modal with "Upload Design - {Position}" title
- Horizontal scroll works when tabs overflow
- Component under 200 lines

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Position list mismatch between screenshot and code | UI inconsistency | Extend DESIGN_POSITION_OPTIONS to match screenshot |
| Many DesignUploadModal instances (one per inactive tab) | DOM bloat | Acceptable for <16 items; could optimize with single shared modal later |

## Security Considerations

- None - display-only component with existing modal

## Next Steps

- Phase 04 integrates this into `OrderItemCard`
