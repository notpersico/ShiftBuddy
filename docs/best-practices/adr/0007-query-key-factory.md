# ADR-0007: Query Key Factory Pattern

## Status

**Accepted**

## Context

React Query uses query keys for caching and invalidation:

```typescript
// Scattered query keys - prone to errors
useQuery({ queryKey: ['users', 'profile'], queryFn: fetchProfile });
queryClient.invalidateQueries({ queryKey: ['users', 'profiles'] }); // Typo!
useQuery({ queryKey: ['user', 'profile'], queryFn: fetchProfile }); // Inconsistent!
```

Problems:
- **Typos break caching**: Similar-looking keys don't match
- **No type safety**: Strings don't catch mismatches at compile time
- **Refactoring nightmare**: Find-and-replace across codebase
- **Inconsistent structure**: Different developers use different patterns

## Decision

Implement a **Query Key Factory** that provides type-safe, centralized query keys.

```typescript
// src/hooks/api/query-keys.ts
export const queryKeys = {
  users: {
    all: ['users'] as const,
    profile: () => ['users', 'profile'] as const,
    detail: (id: string) => ['users', 'detail', id] as const,
  },
  providers: {
    all: ['providers'] as const,
    lists: () => ['providers', 'list'] as const,
    list: (filters: Filters) => ['providers', 'list', filters] as const,
    detail: (id: string) => ['providers', 'detail', id] as const,
  },
};

// Usage
useQuery({
  queryKey: queryKeys.users.profile(),
  queryFn: userService.getProfile,
});

queryClient.invalidateQueries({
  queryKey: queryKeys.users.profile(), // Same function = same key
});
```

## Consequences

### Positive

- **Type safety**: TypeScript catches key mismatches
- **Single source of truth**: Keys defined in one place
- **IntelliSense**: Autocomplete shows available keys
- **Hierarchical invalidation**: Easy to invalidate all of an entity

### Negative

- **Indirection**: Must look up factory for key structure
- **Initial setup**: More upfront work than raw strings
- **Learning curve**: Pattern unfamiliar to some

### Neutral

- Factory must be imported where used
- New entities require adding to factory

## Key Hierarchy

```
queryKeys.providers.all         → ['providers']
queryKeys.providers.lists()     → ['providers', 'list']
queryKeys.providers.list(f)     → ['providers', 'list', { city: 'NYC' }]
queryKeys.providers.details()   → ['providers', 'detail']
queryKeys.providers.detail(id)  → ['providers', 'detail', '123']
```

## Invalidation Patterns

```typescript
// Invalidate all provider queries
queryClient.invalidateQueries({ queryKey: queryKeys.providers.all });

// Invalidate all list queries
queryClient.invalidateQueries({ queryKey: queryKeys.providers.lists() });

// Invalidate specific detail
queryClient.invalidateQueries({ queryKey: queryKeys.providers.detail('123') });
```

## Implementation

```typescript
// Helper to create consistent key factories
type KeySegment = string | number | Record<string, unknown>;

function createKeyFactory<TBase extends readonly string[]>(base: TBase) {
  return {
    all: base,
    extend: <T extends KeySegment[]>(...segments: T) =>
      [...base, ...segments] as const,
  };
}

export const queryKeys = {
  providers: (() => {
    const base = createKeyFactory(['providers'] as const);
    return {
      all: base.all,
      lists: () => base.extend('list'),
      list: (filters: Record<string, unknown>) => base.extend('list', filters),
      details: () => base.extend('detail'),
      detail: (id: string) => base.extend('detail', id),
    };
  })(),
};
```

## References

- [TanStack Query Keys](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys)
- [Effective React Query Keys](https://tkdodo.eu/blog/effective-react-query-keys)
- Query key factory defined in `src/hooks/api/query-keys.ts`
