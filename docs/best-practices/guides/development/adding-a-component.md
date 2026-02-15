# Adding a Component

This guide walks through adding a new UI component to the codebase.

## Overview

Components follow the shadcn/ui pattern:
- Reusable primitives in `src/components/ui/`
- CVA (class-variance-authority) for variants
- Composition-friendly design
- Named exports

---

## Option 1: Use shadcn/ui Component

Check if shadcn/ui has the component you need:

```bash
# Install a shadcn/ui component
pnpm dlx shadcn@latest add button
pnpm dlx shadcn@latest add card
pnpm dlx shadcn@latest add dialog
```

This adds the component to `src/components/ui/` ready to use.

---

## Option 2: Create Custom Component

### Step 1: Create Component File

```typescript
// src/components/ui/badge.tsx
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/components/ui/utils';

// Define variants with CVA
const badgeVariants = cva(
  // Base styles
  'inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-semibold transition-colors',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground',
        secondary: 'bg-secondary text-secondary-foreground',
        destructive: 'bg-destructive text-destructive-foreground',
        outline: 'border border-input bg-background',
      },
      size: {
        sm: 'px-2 py-0.5 text-xs',
        md: 'px-2.5 py-0.5 text-sm',
        lg: 'px-3 py-1 text-base',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'md',
    },
  }
);

// Props interface extending variant props
interface BadgeProps
  extends React.HTMLAttributes<HTMLDivElement>,
    VariantProps<typeof badgeVariants> {}

// Component implementation
export function Badge({ className, variant, size, ...props }: BadgeProps) {
  return (
    <div className={cn(badgeVariants({ variant, size }), className)} {...props} />
  );
}
```

### Step 2: Export from Index (Optional)

If you have a components index file:

```typescript
// src/components/ui/index.ts
export { Badge } from './badge';
export { Button } from './button';
// ...
```

### Step 3: Use the Component

```typescript
import { Badge } from '@/components/ui/badge';

function StatusIndicator({ status }: { status: 'active' | 'pending' | 'error' }) {
  const variant = {
    active: 'default',
    pending: 'secondary',
    error: 'destructive',
  }[status] as const;

  return <Badge variant={variant}>{status}</Badge>;
}
```

---

## Component Patterns

### Composition Pattern

```typescript
// Parent component with composable children
function Card({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn('rounded-lg border bg-card', className)} {...props} />;
}

function CardHeader({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn('flex flex-col space-y-1.5 p-6', className)} {...props} />;
}

function CardContent({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn('p-6 pt-0', className)} {...props} />;
}

// Usage
<Card>
  <CardHeader>
    <h3>Title</h3>
  </CardHeader>
  <CardContent>
    <p>Content goes here</p>
  </CardContent>
</Card>
```

### Polymorphic Pattern

```typescript
// Component that can render as different elements
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  asChild?: boolean;
}

function Button({ asChild, ...props }: ButtonProps) {
  const Comp = asChild ? Slot : 'button';
  return <Comp {...props} />;
}

// Usage
<Button>Regular Button</Button>
<Button asChild>
  <a href="/link">Link Button</a>
</Button>
```

### Controlled/Uncontrolled Pattern

```typescript
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  // Can be controlled (value + onChange) or uncontrolled (defaultValue)
}

function Input({ className, ...props }: InputProps) {
  return (
    <input
      className={cn(
        'flex h-10 w-full rounded-md border bg-background px-3 py-2',
        className
      )}
      {...props}
    />
  );
}
```

---

## Feature Components

For components specific to a feature (not reusable):

```typescript
// src/components/provider/ProviderCard.tsx
import { Card, CardContent, CardHeader } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import type { Provider } from '@/types';

interface Props {
  provider: Provider;
  onFavorite?: (id: string) => void;
}

export function ProviderCard({ provider, onFavorite }: Props) {
  return (
    <Card>
      <CardHeader>
        <img src={provider.avatar} alt={provider.name} />
        <h3>{provider.name}</h3>
        {provider.isPremium && <Badge>Premium</Badge>}
      </CardHeader>
      <CardContent>
        <p>{provider.bio}</p>
        <Button onClick={() => onFavorite?.(provider.id)}>
          Favorite
        </Button>
      </CardContent>
    </Card>
  );
}
```

---

## Styling Guidelines

### Use Design Tokens

```typescript
// Use CSS variables for colors
<div className="bg-background text-foreground" />
<div className="bg-primary text-primary-foreground" />
<div className="border-border" />
```

### Use cn() for Merging Classes

```typescript
import { cn } from '@/components/ui/utils';

// Allows className override and conditional classes
<div className={cn(
  'base-classes',
  isActive && 'active-classes',
  className
)} />
```

### Responsive Design

```typescript
// Mobile-first responsive classes
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4" />
```

---

## Accessibility

```typescript
// Include proper ARIA attributes
<button
  type="button"
  aria-label="Close dialog"
  aria-pressed={isPressed}
  disabled={isDisabled}
  onClick={onClose}
>
  <XIcon aria-hidden="true" />
</button>
```

---

## Checklist

- [ ] Component follows CVA pattern for variants
- [ ] Uses cn() for class merging
- [ ] Props interface properly typed
- [ ] Uses design tokens (not hardcoded colors)
- [ ] Accessible (proper ARIA, keyboard navigation)
- [ ] Named export (not default)
- [ ] Located in correct directory (ui/ vs feature/)
