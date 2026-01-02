# Phase 03: Next.js Integration

## Context

- **Plan:** [plan.md](./plan.md)
- **Issue:** #3
- **Depends on:** Phase 01, Phase 02
- **Working dir:** `apps/web/`

## Overview

| Field | Value |
|-------|-------|
| Date | 2025-12-24 |
| Priority | P0 |
| Effort | 1h |
| Status | Pending |

Integrate MSW với Next.js App Router. Cần setup cho cả client-side (browser) và server-side (SSR).

## Requirements

1. Create MSW provider component
2. Integrate với app layout
3. Conditional loading (dev only)
4. Test mock API calls

## Key Considerations

### MSW 2.x + Next.js Challenges

1. **Client-side:** Need to wait for service worker registration before rendering
2. **Server-side (SSR):** Use `msw/node` server
3. **App Router:** Need client component for provider
4. **Production:** Must not include MSW in production bundle

## Implementation Steps

### Step 1: Create MSW Provider

```typescript
// src/components/providers/msw-provider.tsx
'use client';

import { useEffect, useState, type ReactNode } from 'react';

interface MSWProviderProps {
  children: ReactNode;
}

export function MSWProvider({ children }: MSWProviderProps) {
  const [isReady, setIsReady] = useState(false);

  useEffect(() => {
    async function enableMocking() {
      if (process.env.NODE_ENV !== 'development') {
        setIsReady(true);
        return;
      }

      const { worker } = await import('@/mocks/browser');

      await worker.start({
        onUnhandledRequest: 'bypass', // Don't warn for unhandled requests
        serviceWorker: {
          url: '/mockServiceWorker.js',
        },
      });

      setIsReady(true);
    }

    enableMocking();
  }, []);

  if (!isReady) {
    return null; // Or loading spinner
  }

  return <>{children}</>;
}
```

### Step 2: Create Conditional Export

```typescript
// src/components/providers/index.tsx
'use client';

import { type ReactNode } from 'react';

// Dynamic import MSW provider only in development
const MSWProvider =
  process.env.NODE_ENV === 'development'
    ? require('./msw-provider').MSWProvider
    : ({ children }: { children: ReactNode }) => children;

interface ProvidersProps {
  children: ReactNode;
}

export function Providers({ children }: ProvidersProps) {
  return (
    <MSWProvider>
      {children}
    </MSWProvider>
  );
}
```

### Step 3: Update App Layout

```typescript
// src/app/layout.tsx
import type { Metadata } from "next";
import { Inter, JetBrains_Mono } from "next/font/google";
import { Providers } from "@/components/providers";
import "./globals.css";

const inter = Inter({
  variable: "--font-inter",
  subsets: ["latin"],
});

const jetbrainsMono = JetBrains_Mono({
  variable: "--font-jetbrains-mono",
  subsets: ["latin"],
});

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
      <body className={`${inter.variable} ${jetbrainsMono.variable} antialiased`}>
        <Providers>
          {children}
        </Providers>
      </body>
    </html>
  );
}
```

### Step 4: Setup Server-side Mocking (Optional)

Nếu cần mock API calls trong Server Components:

```typescript
// src/app/layout.tsx (alternative approach)
import type { Metadata } from "next";
// ... imports

// Enable server-side mocking in development
async function enableServerMocking() {
  if (process.env.NODE_ENV === 'development') {
    const { server } = await import('@/mocks/server');
    server.listen({ onUnhandledRequest: 'bypass' });
  }
}

// Call once at module level
if (typeof window === 'undefined') {
  enableServerMocking();
}

// ... rest of layout
```

### Step 5: Create Test Page

```typescript
// src/app/api-test/page.tsx
'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';

export default function ApiTestPage() {
  const [result, setResult] = useState<string>('');
  const [loading, setLoading] = useState(false);

  async function testAuth() {
    setLoading(true);
    try {
      const res = await fetch('/api/auth/me');
      const data = await res.json();
      setResult(JSON.stringify(data, null, 2));
    } catch (error) {
      setResult(`Error: ${error}`);
    }
    setLoading(false);
  }

  async function testTasks() {
    setLoading(true);
    try {
      const res = await fetch('/api/tasks');
      const data = await res.json();
      setResult(JSON.stringify(data, null, 2));
    } catch (error) {
      setResult(`Error: ${error}`);
    }
    setLoading(false);
  }

  async function testOrders() {
    setLoading(true);
    try {
      const res = await fetch('/api/orders');
      const data = await res.json();
      setResult(JSON.stringify(data, null, 2));
    } catch (error) {
      setResult(`Error: ${error}`);
    }
    setLoading(false);
  }

  return (
    <main className="min-h-screen bg-gradient-to-br from-slate-100 via-slate-50 to-white p-8">
      <div className="mx-auto max-w-4xl space-y-8">
        <h1 className="text-3xl font-bold text-slate-900">
          MSW API Test
        </h1>

        <div className="flex gap-4">
          <Button onClick={testAuth} disabled={loading}>
            Test /api/auth/me
          </Button>
          <Button onClick={testTasks} disabled={loading}>
            Test /api/tasks
          </Button>
          <Button onClick={testOrders} disabled={loading}>
            Test /api/orders
          </Button>
        </div>

        <Card>
          <CardHeader>
            <CardTitle>Response</CardTitle>
          </CardHeader>
          <CardContent>
            <pre className="bg-slate-100 p-4 rounded-lg overflow-auto text-sm font-mono">
              {result || 'Click a button to test API'}
            </pre>
          </CardContent>
        </Card>
      </div>
    </main>
  );
}
```

### Step 6: Update TypeScript Config (if needed)

Đảm bảo TypeScript hiểu được MSW imports:

```json
// tsconfig.json - thường không cần thay đổi nếu dùng moduleResolution: bundler
{
  "compilerOptions": {
    "moduleResolution": "bundler"
  }
}
```

## Verification Steps

1. Start dev server: `pnpm dev`
2. Open browser console, should see MSW logs
3. Navigate to `/api-test`
4. Click test buttons, verify mock responses

## Expected Console Output

```
[MSW] Mocking enabled.
[MSW] 09:00:00 GET /api/auth/me (200 OK)
[MSW] 09:00:01 GET /api/tasks (200 OK)
```

## Troubleshooting

### Service Worker Not Found

```bash
# Regenerate service worker
pnpm dlx msw init public/ --save
```

### TypeScript Errors

```bash
# Install missing types
pnpm add -D @types/node
```

### MSW Not Working

1. Check browser console for errors
2. Verify `mockServiceWorker.js` exists in `public/`
3. Check if provider is wrapping the app

## Success Criteria

- [ ] MSW loads in development
- [ ] Console shows MSW enabled message
- [ ] `/api-test` page works
- [ ] All mock endpoints respond correctly
- [ ] No MSW in production build

## Files Created/Modified

```
apps/web/src/
├── components/
│   └── providers/
│       ├── index.tsx
│       └── msw-provider.tsx
├── app/
│   ├── layout.tsx (modified)
│   └── api-test/
│       └── page.tsx
└── mocks/
    └── (from phase 2)
```

## Next Steps

- Issue #4: Tạo mock data structure
- Issue #5: Setup Zustand + TanStack Query
