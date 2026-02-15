# TanStack Query Best Practices

Modern TanStack Query (v5) patterns for server state management.

## Table of Contents

1. [Object Signature (v5 Standard)](#1-object-signature-v5-standard)
2. [Query Key Factory Pattern](#2-query-key-factory-pattern)
3. [Suspense Integration](#3-suspense-integration)
4. [Error Handling](#4-error-handling)
5. [Mutations](#5-mutations)
6. [Optimistic Updates](#6-optimistic-updates)
7. [Cache Invalidation](#7-cache-invalidation)
8. [Dependent Queries](#8-dependent-queries)
9. [Infinite Queries](#9-infinite-queries)
10. [Prefetching](#10-prefetching)

---

## 1. Object Signature (v5 Standard)

TanStack Query v5 requires the unified object signature:

```typescript
// DEPRECATED (v4)
useQuery(['key'], fn);

// STANDARD (v5)
const { data } = useQuery({
  queryKey: ['invoices', id],
  queryFn: () => api.getInvoice(id),
  staleTime: 1000 * 60 * 5,
  gcTime: 1000 * 60 * 30,
});
```

### Key v5 Changes

| v4 Name | v5 Name | Notes |
|---------|---------|-------|
| `cacheTime` | `gcTime` | Garbage collection time |
| `useErrorBoundary` | `throwOnError` | Error boundary integration |
| `isLoading` | `isPending` | Initial loading (no data yet) |
| `isInitialLoading` | `isLoading` | First fetch in progress |

---

## 2. Query Key Factory Pattern

Use centralized query keys in `src/hooks/api/query-keys.ts`:

```typescript
export const queryKeys = {
  providers: (() => {
    const base = createKeyFactory(['providers'] as const);
    return {
      all: base.all,                                    // ['providers']
      lists: () => base.extend('list'),                 // ['providers', 'list']
      list: (filters: Record<string, unknown>) =>
        base.extend('list', filters),                   // ['providers', 'list', {...}]
      details: () => base.extend('detail'),             // ['providers', 'detail']
      detail: (id: string) => base.extend('detail', id), // ['providers', 'detail', '123']
    };
  })(),
};

// Usage
const { data } = useQuery({
  queryKey: queryKeys.providers.detail(providerId),
  queryFn: () => providerService.getById(providerId),
});

// Invalidation
queryClient.invalidateQueries({ queryKey: queryKeys.providers.all });
queryClient.invalidateQueries({ queryKey: queryKeys.providers.lists() });
```

---

## 3. Suspense Integration

### useSuspenseQuery

Data is guaranteed to be defined when using Suspense:

```typescript
import { useSuspenseQuery } from '@tanstack/react-query';

function UserProfile() {
  // data is NEVER undefined - component suspends until ready
  const { data: profile } = useSuspenseQuery({
    queryKey: queryKeys.users.profile(),
    queryFn: () => userService.getProfile(),
  });

  return <div>{profile.display_name}</div>;
}

// Parent wraps with Suspense
function ProfilePage() {
  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <UserProfile />
    </Suspense>
  );
}
```

### useSuspenseQueries (Parallel)

```typescript
function Dashboard() {
  const [profileQuery, statsQuery] = useSuspenseQueries({
    queries: [
      { queryKey: queryKeys.users.profile(), queryFn: userService.getProfile },
      { queryKey: queryKeys.dashboard.stats(), queryFn: dashboardService.getStats },
    ],
  });

  // Both guaranteed to have data
  return (
    <div>
      <h1>Welcome, {profileQuery.data.name}</h1>
      <Stats data={statsQuery.data} />
    </div>
  );
}
```

---

## 4. Error Handling

### With Error Boundaries

```typescript
import { QueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

function App() {
  return (
    <QueryErrorResetBoundary>
      {({ reset }) => (
        <ErrorBoundary
          onReset={reset}
          fallbackRender={({ error, resetErrorBoundary }) => (
            <div role="alert">
              <p>Something went wrong: {error.message}</p>
              <button onClick={resetErrorBoundary}>Try again</button>
            </div>
          )}
        >
          <Suspense fallback={<Loading />}>
            <MainContent />
          </Suspense>
        </ErrorBoundary>
      )}
    </QueryErrorResetBoundary>
  );
}
```

### throwOnError Option

```typescript
// Always throw (use with Error Boundary)
useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
  throwOnError: true,
});

// Conditional throwing
useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
  throwOnError: (error) => error.status >= 500,
});
```

---

## 5. Mutations

### Basic Pattern

```typescript
export function useUpdateProfile() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (payload: UpdateProfilePayload) =>
      userService.updateProfile(payload),

    onSuccess: (data) => {
      queryClient.setQueryData(queryKeys.users.profile(), data);
      queryClient.invalidateQueries({ queryKey: queryKeys.auth.all });
    },

    onError: (error) => {
      console.error('Profile update failed', error);
    },
  });
}
```

---

## 6. Optimistic Updates

```typescript
export function useToggleFavorite() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (providerId: string) => favoriteService.toggle(providerId),

    onMutate: async (providerId) => {
      // 1. Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: queryKeys.favorites.list() });

      // 2. Snapshot previous value
      const previousFavorites = queryClient.getQueryData<Favorite[]>(
        queryKeys.favorites.list()
      );

      // 3. Optimistically update
      queryClient.setQueryData<Favorite[]>(
        queryKeys.favorites.list(),
        (old = []) => {
          const exists = old.some((f) => f.providerId === providerId);
          return exists
            ? old.filter((f) => f.providerId !== providerId)
            : [...old, { id: 'temp', providerId }];
        }
      );

      // 4. Return context for rollback
      return { previousFavorites };
    },

    onError: (_err, _providerId, context) => {
      // 5. Rollback on error
      if (context?.previousFavorites) {
        queryClient.setQueryData(queryKeys.favorites.list(), context.previousFavorites);
      }
    },

    onSettled: () => {
      // 6. Always refetch to ensure sync
      queryClient.invalidateQueries({ queryKey: queryKeys.favorites.list() });
    },
  });
}
```

---

## 7. Cache Invalidation

```typescript
const queryClient = useQueryClient();

// Invalidate by exact key
queryClient.invalidateQueries({ queryKey: queryKeys.providers.detail('123') });

// Invalidate by prefix (all matching keys)
queryClient.invalidateQueries({ queryKey: queryKeys.providers.all });

// Invalidate with predicate
queryClient.invalidateQueries({
  predicate: (query) =>
    query.queryKey[0] === 'providers' &&
    query.state.dataUpdatedAt < Date.now() - 60000,
});

// Refetch immediately
queryClient.refetchQueries({ queryKey: queryKeys.providers.lists() });

// Remove from cache
queryClient.removeQueries({ queryKey: queryKeys.providers.detail('123') });
```

---

## 8. Dependent Queries

```typescript
// First query: get user
const { data: user } = useQuery({
  queryKey: queryKeys.users.profile(),
  queryFn: userService.getProfile,
});

// Second query: depends on user.id
const { data: preferences } = useQuery({
  queryKey: queryKeys.clientPreferences.current(),
  queryFn: () => preferencesService.get(user!.id),
  enabled: !!user?.id, // Only runs when user.id exists
});
```

**Performance Warning**: Dependent queries create request waterfalls. Consider combining into a single API endpoint when possible.

---

## 9. Infinite Queries

```typescript
export function useProviderSearch(filters: SearchFilters) {
  return useInfiniteQuery({
    queryKey: queryKeys.providers.list(filters),
    queryFn: ({ pageParam }) =>
      providerService.search({ ...filters, cursor: pageParam }),

    initialPageParam: null as string | null,

    getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
  });
}

// Usage
function ProviderList() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } =
    useProviderSearch(filters);

  const allItems = data?.pages.flatMap((page) => page.items) ?? [];

  return (
    <>
      {allItems.map((provider) => (
        <ProviderCard key={provider.id} {...provider} />
      ))}
      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage ? 'Loading...' : hasNextPage ? 'Load More' : 'End'}
      </button>
    </>
  );
}
```

---

## 10. Prefetching

### On Hover

```typescript
function ProviderCard({ provider }: { provider: Provider }) {
  const queryClient = useQueryClient();

  const prefetchProfile = () => {
    queryClient.prefetchQuery({
      queryKey: queryKeys.providers.detail(provider.id),
      queryFn: () => providerService.getById(provider.id),
      staleTime: 1000 * 60 * 5,
    });
  };

  return (
    <Link
      to={`/providers/${provider.id}`}
      onMouseEnter={prefetchProfile}
      onFocus={prefetchProfile}
    >
      {provider.name}
    </Link>
  );
}
```

### Ensure Data Exists

```typescript
// Returns cached data if available, otherwise fetches
await queryClient.ensureQueryData({
  queryKey: queryKeys.providers.detail(providerId),
  queryFn: () => providerService.getById(providerId),
});
```

---

## Quick Reference

```typescript
// Import paths
import { useQuery, useMutation, useSuspenseQuery, useQueryClient } from '@tanstack/react-query';
import { queryKeys } from '@/hooks/api/query-keys';

// Preferred patterns
- Use `useSuspenseQuery` for data that must exist
- Use `queryKeys.entity.method()` - never hardcode strings
- Use optimistic updates for immediate UI feedback
- Wrap with `<Suspense>` + `<ErrorBoundary>` at route level
```
