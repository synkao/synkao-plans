# Phase 6: Login Page (~1h)

## Context

- Plan: [plan.md](./plan.md)
- Depends on: [Phase 5](./phase-05-auth-hooks.md)
- Issue: #63
- Current: [login/page.tsx](../../apps/web/src/app/(auth)/login/page.tsx)

## Overview

Update login page with functional form using react-hook-form + zod validation. Connect to useLogin hook and handle redirect on success.

## Requirements

1. Add react-hook-form integration
2. Add zod schema validation
3. Connect to useLogin mutation
4. Handle loading/error states
5. Redirect to / on success
6. Keep glass morphism styling

## Architecture

### Form Schema

```typescript
const loginSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(1, 'Password required'),
});

type LoginFormData = z.infer<typeof loginSchema>;
```

### Component Structure

```
LoginPage
├── Form (react-hook-form)
│   ├── Email Input
│   ├── Password Input
│   ├── Error Display
│   └── Submit Button
└── Redirect Logic (useRouter)
```

### Error Handling

| Error Type | Display |
|------------|---------|
| Validation | Under input field |
| API error | Toast or inline alert |
| Network | Generic error message |

## Related Files

| File | Action |
|------|--------|
| `apps/web/src/app/(auth)/login/page.tsx` | UPDATE |
| `apps/web/src/hooks/use-auth.ts` | IMPORT |

## Implementation Steps

### Step 1: Add imports

```typescript
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useRouter } from 'next/navigation';
import { useLogin } from '@/hooks/use-auth';
```

### Step 2: Define schema

```typescript
const loginSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(1, 'Password is required'),
});

type LoginFormData = z.infer<typeof loginSchema>;
```

### Step 3: Setup form

```typescript
export default function LoginPage() {
  const router = useRouter();
  const loginMutation = useLogin();

  const form = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: 'admin@synkao.com',
      password: 'password',
    },
  });

  const onSubmit = (data: LoginFormData) => {
    loginMutation.mutate(data, {
      onSuccess: () => router.push('/'),
      onError: (error) => {
        form.setError('root', { message: error.message });
      },
    });
  };

  // ... rest of component
}
```

### Step 4: Update form JSX

```tsx
<form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
  <div className="space-y-2">
    <label htmlFor="email" className="text-sm font-medium">
      Email
    </label>
    <Input
      id="email"
      type="email"
      placeholder="name@company.com"
      {...form.register('email')}
    />
    {form.formState.errors.email && (
      <p className="text-sm text-destructive">
        {form.formState.errors.email.message}
      </p>
    )}
  </div>

  <div className="space-y-2">
    <label htmlFor="password" className="text-sm font-medium">
      Password
    </label>
    <Input
      id="password"
      type="password"
      placeholder="••••••••"
      {...form.register('password')}
    />
    {form.formState.errors.password && (
      <p className="text-sm text-destructive">
        {form.formState.errors.password.message}
      </p>
    )}
  </div>

  {form.formState.errors.root && (
    <p className="text-sm text-destructive text-center">
      {form.formState.errors.root.message}
    </p>
  )}

  <Button
    className="w-full"
    type="submit"
    disabled={loginMutation.isPending}
  >
    {loginMutation.isPending ? 'Signing in...' : 'Sign in'}
  </Button>
</form>
```

### Step 5: Add redirect if already logged in

```typescript
import { useEffect } from 'react';
import { authClient } from '@/lib/auth-client';

export default function LoginPage() {
  const router = useRouter();

  useEffect(() => {
    if (authClient.hasToken()) {
      router.replace('/');
    }
  }, [router]);

  // ... rest
}
```

## Todo List

- [ ] Add 'use client' directive
- [ ] Import react-hook-form, zod, useLogin
- [ ] Define loginSchema
- [ ] Setup useForm with zodResolver
- [ ] Connect onSubmit to loginMutation
- [ ] Update Input components with register
- [ ] Add error message display
- [ ] Add loading state to button
- [ ] Add redirect on success
- [ ] Add redirect if already logged in
- [ ] Test form validation
- [ ] Test successful login flow

## Success Criteria

- [ ] Form validates email format
- [ ] Form requires password
- [ ] Submit button shows loading state
- [ ] Successful login redirects to /
- [ ] Failed login shows error message
- [ ] Already logged in redirects away from /login
- [ ] Glass morphism styling preserved
