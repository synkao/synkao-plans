# Phase 07: UI - Customer Info Enhancement

**Priority:** P2 - Medium
**Status:** ✅ Complete
**Depends on:** Phase 01
**Completion Score:** 9/10
**Completed:** 2026-01-22 20:15

## Overview

Enhance CustomerInfoCard with:
- External ID (WooCommerce Order ID)
- Separate Billing vs Shipping addresses
- Order Notes section (new component)
- Order Metadata card (timestamps, update button)

## Related Files

**Modify:**
- `apps/web/src/features/orders/components/order-detail/customer-info-card.tsx`

**Create:**
- `apps/web/src/features/orders/components/order-detail/order-notes-section.tsx`
- `apps/web/src/features/orders/components/order-detail/order-metadata-card.tsx`

**Update:**
- `apps/web/src/features/orders/components/order-detail/index.ts`
- `apps/web/src/app/(main)/orders/[id]/page.tsx`

## Design

### Enhanced Customer Info Card

```
┌─────────────────────────────────────────────────────────┐
│ CUSTOMER INFORMATION                                    │
├─────────────────────────────────────────────────────────┤
│ 👤 John Smith                                           │
│ ✉️ john.smith@email.com                                 │
│ 📞 +1 (415) 555-1234                                    │
│                                                         │
│ 🏷️ External ID: WC-12345                               │
│ 🌐 Source: WooCommerce                                  │
├─────────────────────────────────────────────────────────┤
│ SHIPPING ADDRESS                                        │
│ 123 Main Street                                         │
│ San Francisco, CA 94102                                 │
│ United States                                           │
├─────────────────────────────────────────────────────────┤
│ BILLING ADDRESS (same as shipping)                      │
│ — or —                                                  │
│ BILLING ADDRESS                                         │
│ 456 Billing Ave                                         │
│ Los Angeles, CA 90001                                   │
│ United States                                           │
└─────────────────────────────────────────────────────────┘
```

### Order Notes Section

```
┌─────────────────────────────────────────────────────────┐
│ ORDER NOTES                               [✎ Edit]      │
├─────────────────────────────────────────────────────────┤
│ Customer requests delivery before 12:00 PM.             │
│ Please call 30 minutes before delivery.                 │
└─────────────────────────────────────────────────────────┘
```

### Order Metadata Card

```
┌─────────────────────────────────────────────────────────┐
│ ORDER INFO                                              │
├─────────────────────────────────────────────────────────┤
│ Created      Jan 20, 2026 08:30                         │
│ Updated      Jan 20, 2026 14:30                         │
│                                                         │
│              [✎ Update Status]                          │
└─────────────────────────────────────────────────────────┘
```

## Implementation

### Enhanced CustomerInfoCard

```tsx
'use client';

import type { MockOrder, MockAddress } from '../../types';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { Separator } from '@/components/ui/separator';
import { Badge } from '@/components/ui/badge';
import { User, Mail, Phone, MapPin, Tag, Globe } from 'lucide-react';

export interface CustomerInfoCardProps {
  order: MockOrder;
}

// Source labels
const SOURCE_LABELS = {
  SYNC: 'WooCommerce',
  CSV_IMPORT: 'CSV Import',
  MANUAL: 'Manual Entry',
};

export function CustomerInfoCard({ order }: CustomerInfoCardProps) {
  const shippingAddress = order.shippingAddress;
  const billingAddress = order.billingAddress;
  const isSameAddress = addressesMatch(shippingAddress, billingAddress);

  return (
    <Card className="border border-gray-200/50 bg-white/60 backdrop-blur-md">
      <CardHeader className="pb-3">
        <CardTitle className="text-sm font-medium text-gray-500 uppercase tracking-wider">
          Customer Information
        </CardTitle>
      </CardHeader>
      <CardContent className="space-y-4">
        {/* Contact Info */}
        <div className="space-y-2">
          <InfoRow icon={User} value={order.customerName} />
          {order.customerEmail && (
            <InfoRow
              icon={Mail}
              value={order.customerEmail}
              href={`mailto:${order.customerEmail}`}
            />
          )}
          {order.customerPhone && (
            <InfoRow
              icon={Phone}
              value={order.customerPhone}
              href={`tel:${order.customerPhone}`}
            />
          )}
        </div>

        {/* External ID & Source */}
        {(order.externalId || order.source) && (
          <>
            <Separator />
            <div className="space-y-2">
              {order.externalId && (
                <InfoRow icon={Tag} label="External ID" value={order.externalId} />
              )}
              {order.source && (
                <InfoRow
                  icon={Globe}
                  label="Source"
                  value={SOURCE_LABELS[order.source] ?? order.source}
                />
              )}
            </div>
          </>
        )}

        {/* Shipping Address */}
        {shippingAddress && (
          <>
            <Separator />
            <AddressSection title="Shipping Address" address={shippingAddress} />
          </>
        )}

        {/* Billing Address */}
        {billingAddress && (
          <>
            <Separator />
            {isSameAddress ? (
              <div className="text-sm text-gray-500 italic">
                Billing address same as shipping
              </div>
            ) : (
              <AddressSection title="Billing Address" address={billingAddress} />
            )}
          </>
        )}
      </CardContent>
    </Card>
  );
}

function InfoRow({
  icon: Icon,
  label,
  value,
  href,
}: {
  icon: typeof User;
  label?: string;
  value: string;
  href?: string;
}) {
  const content = (
    <div className="flex items-start gap-3">
      <Icon className="h-4 w-4 text-gray-400 mt-0.5" />
      {label ? (
        <span className="text-gray-600">
          <span className="text-gray-500">{label}:</span> {value}
        </span>
      ) : (
        <span className="text-gray-900 font-medium">{value}</span>
      )}
    </div>
  );

  if (href) {
    return (
      <a href={href} className="block hover:text-blue-600 transition-colors">
        {content}
      </a>
    );
  }

  return content;
}

function AddressSection({
  title,
  address,
}: {
  title: string;
  address: MockAddress;
}) {
  const lines = [
    address.line1,
    address.line2,
    `${address.city}${address.state ? `, ${address.state}` : ''} ${address.postalCode || ''}`,
    address.country,
  ].filter(Boolean);

  return (
    <div>
      <h4 className="text-xs font-medium text-gray-500 uppercase tracking-wider mb-2">
        {title}
      </h4>
      <div className="flex items-start gap-3">
        <MapPin className="h-4 w-4 text-gray-400 mt-0.5" />
        <div className="text-sm text-gray-600">
          {lines.map((line, idx) => (
            <div key={idx}>{line}</div>
          ))}
        </div>
      </div>
    </div>
  );
}

function addressesMatch(
  a?: MockAddress,
  b?: MockAddress
): boolean {
  if (!a || !b) return false;
  return (
    a.line1 === b.line1 &&
    a.city === b.city &&
    a.postalCode === b.postalCode &&
    a.country === b.country
  );
}
```

### OrderNotesSection

```tsx
// order-notes-section.tsx

'use client';

import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Edit } from 'lucide-react';

interface OrderNotesSectionProps {
  notes?: string;
  onEditClick?: () => void;
}

export function OrderNotesSection({ notes, onEditClick }: OrderNotesSectionProps) {
  return (
    <Card className="border border-gray-200/50 bg-white/60 backdrop-blur-md">
      <CardHeader className="pb-3 flex flex-row items-center justify-between">
        <CardTitle className="text-sm font-medium text-gray-500 uppercase tracking-wider">
          Order Notes
        </CardTitle>
        {onEditClick && (
          <Button variant="ghost" size="sm" onClick={onEditClick}>
            <Edit className="h-4 w-4" />
          </Button>
        )}
      </CardHeader>
      <CardContent>
        {notes ? (
          <p className="text-sm text-gray-600 whitespace-pre-wrap">{notes}</p>
        ) : (
          <p className="text-sm text-gray-400 italic">No notes added</p>
        )}
      </CardContent>
    </Card>
  );
}
```

### OrderMetadataCard

```tsx
// order-metadata-card.tsx

'use client';

import type { MockOrder } from '../../types';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Clock, RefreshCw, Edit } from 'lucide-react';

interface OrderMetadataCardProps {
  order: MockOrder;
  onUpdateStatus?: () => void;
}

export function OrderMetadataCard({ order, onUpdateStatus }: OrderMetadataCardProps) {
  const formatDateTime = (dateStr: string) => {
    return new Date(dateStr).toLocaleDateString('en-US', {
      month: 'short',
      day: 'numeric',
      year: 'numeric',
      hour: '2-digit',
      minute: '2-digit',
    });
  };

  return (
    <Card className="border border-gray-200/50 bg-white/60 backdrop-blur-md">
      <CardHeader className="pb-3">
        <CardTitle className="text-sm font-medium text-gray-500 uppercase tracking-wider">
          Order Info
        </CardTitle>
      </CardHeader>
      <CardContent className="space-y-3">
        <div className="flex items-center justify-between text-sm">
          <span className="text-gray-500 flex items-center gap-2">
            <Clock className="h-4 w-4" />
            Created
          </span>
          <span className="text-gray-700">{formatDateTime(order.createdAt)}</span>
        </div>

        {order.updatedAt && (
          <div className="flex items-center justify-between text-sm">
            <span className="text-gray-500 flex items-center gap-2">
              <RefreshCw className="h-4 w-4" />
              Updated
            </span>
            <span className="text-gray-700">{formatDateTime(order.updatedAt)}</span>
          </div>
        )}

        {onUpdateStatus && (
          <Button
            variant="outline"
            size="sm"
            className="w-full mt-2"
            onClick={onUpdateStatus}
          >
            <Edit className="mr-2 h-4 w-4" />
            Update Status
          </Button>
        )}
      </CardContent>
    </Card>
  );
}
```

### Update page.tsx

```tsx
// Add to imports
import { OrderNotesSection, OrderMetadataCard } from '@/features/orders';

// In Left Column, after CustomerInfoCard:
<OrderNotesSection
  notes={order.notes}
  onEditClick={() => setEditNotesOpen(true)}
/>

// In Right Column, before OrderStatusStepper:
<OrderMetadataCard
  order={order}
  onUpdateStatus={() => toast.info('Update status (coming soon)')}
/>
```

## Todo

- [x] Update CustomerInfoCard with External ID, Source ✅
- [x] Add billing address section with same-address check ✅
- [x] Create OrderNotesSection component ✅
- [x] Create OrderMetadataCard component ✅
- [x] Export new components ✅
- [x] Add components to page.tsx ✅
- [x] Wire up Edit button to dialog ✅
- [x] Test with orders with/without billing address ✅
- [x] Test with/without notes ✅

## Success Criteria

- External ID and Source displayed in Customer Info
- Billing address shown separately (or "same as shipping")
- Order Notes section with Edit button
- Order Metadata shows created/updated timestamps
- Update Status button present (placeholder)
- All sections responsive on mobile

## Completion

After all phases complete:

- [ ] Run `pnpm build` to verify no errors
- [ ] Test all 12 acceptance criteria
- [ ] Test mobile responsive
- [ ] Update issue #73 with implementation notes
