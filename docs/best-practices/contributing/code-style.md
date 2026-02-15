# Code Style

Formatting and conventions for the codebase.

## TypeScript

### Strict Mode

TypeScript is configured with strict settings:

- `noUnusedLocals`: No unused local variables
- `noUnusedParameters`: No unused function parameters
- `noFallthroughCasesInSwitch`: Explicit case handling

### Type Safety Rules

| Rule | Enforcement |
|------|-------------|
| No `any` | Warning (use proper types) |
| No `@ts-ignore` | Prohibited (fix the issue) |
| Unused variables | Prefix with `_` (e.g., `_unusedParam`) |

```typescript
// ✅ Correct - unused param prefixed
function handler(_event: Event, data: Data) {
  return data;
}

// ❌ Wrong - unused param not prefixed
function handler(event: Event, data: Data) {
  return data;
}
```

---

## Prettier Configuration

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "bracketSpacing": true,
  "bracketSameLine": false,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

Run formatting:
```bash
pnpm format
```

---

## ESLint Rules

| Rule | Description |
|------|-------------|
| React 17+ JSX | No need to import React |
| React Hooks | Rules enforced |
| No console.log | Warning - use `console.warn`/`console.error` |
| Prefer const | No `var` allowed |

Run linting:
```bash
pnpm lint        # Check for issues
pnpm lint:fix    # Auto-fix issues
```

---

## Component Conventions

### Structure

```typescript
// 1. Imports
import { cn } from '@/components/ui/utils';

// 2. Types
interface Props {
  title: string;
  className?: string;
}

// 3. Component (named export)
export function MyComponent({ title, className }: Props) {
  return <div className={cn('base-classes', className)}>{title}</div>;
}
```

### Rules

- **Functional components** with explicit type annotations
- **Named exports** from components
- **Props interface** defined inline or separately
- **Lazy loading** for route-level components

---

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `LoginPage`, `RoleSelection` |
| Hooks | camelCase with `use` | `useAuth`, `useRouter` |
| Services | camelCase + `.service.ts` | `auth.service.ts` |
| Types | PascalCase + `.types.ts` | `router.types.ts` |
| Schemas | camelCase + `.schema.ts` | `profile.schema.ts` |

---

## Import Order

Organize imports in this order:

```typescript
// 1. React imports
import { useState, useEffect } from 'react';

// 2. External libraries
import { useQuery } from '@tanstack/react-query';
import { z } from 'zod';

// 3. Internal contexts/providers
import { useAuth } from '@/contexts/AuthContext';
import { useRouter } from '@/contexts/RouterContext';

// 4. Hooks
import { useProfiles } from '@/hooks/api/useProfiles';

// 5. Components
import { Button } from '@/components/ui/button';
import { Card } from '@/components/ui/card';

// 6. Types
import type { Profile } from '@/types/profile.types';

// 7. Utils/helpers
import { formatDate } from '@/lib/utils';
```

---

## Import Alias

Always use the `@/` alias instead of relative paths:

```typescript
// ✅ Correct
import { Button } from '@/components/ui/button';
import { useAuth } from '@/contexts/AuthContext';

// ❌ Wrong
import { Button } from '../../components/ui/button';
import { useAuth } from '../../../contexts/AuthContext';
```

---

## CSS/Styling

### Tailwind CSS v4

- Use CSS custom properties for design tokens
- Use `cn()` utility for class merging

```typescript
import { cn } from '@/components/ui/utils';

function Card({ className, ...props }: Props) {
  return (
    <div
      className={cn(
        'bg-card rounded-lg p-4',
        className
      )}
      {...props}
    />
  );
}
```

### Design Tokens

Reference CSS variables for consistent theming:

```css
/* Use semantic tokens instead of hardcoded values */
bg-primary        /* Primary brand color */
bg-background     /* App background */
text-foreground   /* Text color */
bg-card           /* Card background */
```

---

## Validation Commands

Always run before committing:

```bash
pnpm typecheck && pnpm lint && pnpm test:run
```

Fix issues before considering work complete.
