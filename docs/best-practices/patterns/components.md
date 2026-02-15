# Component Patterns

Patterns for building React components.

## CVA (Class Variance Authority)

Use CVA for components with variants:

```typescript
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/components/ui/utils';

const buttonVariants = cva(
  // Base styles (always applied)
  'inline-flex items-center justify-center rounded-md font-medium transition-colors',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        outline: 'border border-input bg-background hover:bg-accent',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
      },
      size: {
        sm: 'h-8 px-3 text-sm',
        md: 'h-10 px-4',
        lg: 'h-12 px-6 text-lg',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'md',
    },
  }
);

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

export function Button({ className, variant, size, ...props }: ButtonProps) {
  return (
    <button className={cn(buttonVariants({ variant, size }), className)} {...props} />
  );
}
```

---

## Composition Pattern

Build complex components from composable parts:

```typescript
// Card with composable sub-components
function Card({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn('rounded-lg border bg-card', className)} {...props} />;
}

function CardHeader({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn('flex flex-col space-y-1.5 p-6', className)} {...props} />;
}

function CardTitle({ className, ...props }: React.HTMLAttributes<HTMLHeadingElement>) {
  return <h3 className={cn('text-lg font-semibold', className)} {...props} />;
}

function CardContent({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn('p-6 pt-0', className)} {...props} />;
}

// Usage
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent>Content here</CardContent>
</Card>
```

---

## Props Interface

Always define explicit props interface:

```typescript
interface ProfileCardProps {
  profile: Profile;
  onFavorite?: (id: string) => void;
  showActions?: boolean;
  className?: string;
}

export function ProfileCard({
  profile,
  onFavorite,
  showActions = true,
  className,
}: ProfileCardProps) {
  return (/* ... */);
}
```

---

## Render Props / Children Pattern

```typescript
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string;
  emptyMessage?: string;
}

export function List<T>({
  items,
  renderItem,
  keyExtractor,
  emptyMessage = 'No items',
}: ListProps<T>) {
  if (items.length === 0) {
    return <p className="text-muted-foreground">{emptyMessage}</p>;
  }

  return (
    <ul className="space-y-2">
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>{renderItem(item, index)}</li>
      ))}
    </ul>
  );
}
```

---

## Controlled vs Uncontrolled

Support both patterns:

```typescript
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  // value + onChange = controlled
  // defaultValue = uncontrolled
}

export function Input({ className, ...props }: InputProps) {
  return (
    <input
      className={cn('flex h-10 w-full rounded-md border px-3 py-2', className)}
      {...props}
    />
  );
}
```

---

## Forwarding Refs

React 19 allows ref as a regular prop:

```typescript
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  ref?: React.Ref<HTMLInputElement>;
}

export function Input({ ref, className, ...props }: InputProps) {
  return <input ref={ref} className={cn('...', className)} {...props} />;
}
```

---

## Loading/Error States

Handle all states in data-fetching components:

```typescript
function ProfileList() {
  const { data, isLoading, error } = useProfiles();

  if (isLoading) return <ProfileListSkeleton />;
  if (error) return <ErrorMessage error={error} />;
  if (!data?.length) return <EmptyState message="No profiles found" />;

  return (
    <div className="grid gap-4">
      {data.map((profile) => (
        <ProfileCard key={profile.id} profile={profile} />
      ))}
    </div>
  );
}
```
