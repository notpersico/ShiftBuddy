# Code Patterns

Reusable patterns and conventions used throughout the codebase.

## Contents

| Pattern | Description |
|---------|-------------|
| [Components](./components.md) | Component patterns (CVA, composition) |
| [Hooks](./hooks.md) | React Query and custom hook patterns |
| [Services](./services.md) | Service layer patterns |
| [Contexts](./contexts.md) | React Context patterns |
| [Testing](./testing.md) | Testing conventions |
| [Schemas](./schemas.md) | Zod validation patterns |

---

## Quick Reference

### Component Pattern

```typescript
import { cn } from '@/components/ui/utils';
import { cva, type VariantProps } from 'class-variance-authority';

const variants = cva('base-classes', {
  variants: { size: { sm: '...', md: '...' } },
  defaultVariants: { size: 'md' },
});

interface Props extends VariantProps<typeof variants> {
  className?: string;
  children: React.ReactNode;
}

export function Component({ className, size, children }: Props) {
  return <div className={cn(variants({ size }), className)}>{children}</div>;
}
```

### Hook Pattern

```typescript
export function useResource(id: string) {
  return useQuery({
    queryKey: queryKeys.resources.detail(id),
    queryFn: () => resourceService.getById(id),
    enabled: !!id,
  });
}
```

### Service Pattern

```typescript
export const resourceService = {
  async getById(id: string) {
    const { data, error } = await supabase.from('resources').select('*').eq('id', id).single();
    if (error) throw error;
    return data;
  },
};
```

### Schema Pattern

```typescript
export const resourceSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1),
  createdAt: z.string().datetime(),
});

export type Resource = z.infer<typeof resourceSchema>;
```
