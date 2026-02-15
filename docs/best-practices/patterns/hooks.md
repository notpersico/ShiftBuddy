# Hook Patterns

Patterns for React Query and custom hooks.

## Query Hook Pattern

```typescript
export function useProfiles(filters?: ProfileFilters) {
  return useQuery({
    queryKey: queryKeys.profiles.list(filters ?? {}),
    queryFn: () => profileService.search(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
}

export function useProfile(id: string) {
  return useQuery({
    queryKey: queryKeys.profiles.detail(id),
    queryFn: () => profileService.getById(id),
    enabled: !!id, // Disable when no ID
  });
}
```

---

## Mutation Hook Pattern

```typescript
export function useUpdateProfile() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateProfileInput }) =>
      profileService.update(id, data),

    onSuccess: (data) => {
      // Update cache directly
      queryClient.setQueryData(queryKeys.profiles.detail(data.id), data);
      // Invalidate lists
      queryClient.invalidateQueries({ queryKey: queryKeys.profiles.lists() });
    },

    onError: (error) => {
      console.error('Update failed:', error);
    },
  });
}
```

---

## Optimistic Mutation Pattern

```typescript
export function useToggleFavorite() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: favoriteService.toggle,

    onMutate: async (providerId) => {
      await queryClient.cancelQueries({ queryKey: queryKeys.favorites.list() });

      const previous = queryClient.getQueryData<string[]>(queryKeys.favorites.list());

      queryClient.setQueryData<string[]>(
        queryKeys.favorites.list(),
        (old = []) =>
          old.includes(providerId)
            ? old.filter((id) => id !== providerId)
            : [...old, providerId]
      );

      return { previous };
    },

    onError: (_err, _vars, context) => {
      if (context?.previous) {
        queryClient.setQueryData(queryKeys.favorites.list(), context.previous);
      }
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.favorites.list() });
    },
  });
}
```

---

## Suspense Query Pattern

```typescript
// Use with Suspense boundary
export function useProfileSuspense(id: string) {
  return useSuspenseQuery({
    queryKey: queryKeys.profiles.detail(id),
    queryFn: () => profileService.getById(id),
  });
}

// Component - data is guaranteed
function ProfileDetails({ id }: { id: string }) {
  const { data: profile } = useProfileSuspense(id);
  return <div>{profile.name}</div>; // No undefined check needed
}

// Parent wraps with Suspense
<Suspense fallback={<ProfileSkeleton />}>
  <ProfileDetails id={profileId} />
</Suspense>
```

---

## Custom Hook Composition

```typescript
// Compose multiple hooks
export function useProviderDashboard() {
  const { data: profile, isLoading: profileLoading } = useProviderProfile();
  const { data: stats, isLoading: statsLoading } = useProviderStats();
  const { data: messages, isLoading: messagesLoading } = useUnreadMessages();

  return {
    profile,
    stats,
    messages,
    isLoading: profileLoading || statsLoading || messagesLoading,
  };
}
```

---

## Dependent Queries

```typescript
// Second query depends on first
export function useProviderWithPreferences(providerId: string) {
  const { data: provider } = useProvider(providerId);

  const { data: preferences } = useQuery({
    queryKey: ['preferences', provider?.userId],
    queryFn: () => preferencesService.get(provider!.userId),
    enabled: !!provider?.userId, // Only runs when provider loaded
  });

  return { provider, preferences };
}
```

---

## Infinite Query Pattern

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
const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useProviderSearch(filters);
const allProviders = data?.pages.flatMap((page) => page.items) ?? [];
```
