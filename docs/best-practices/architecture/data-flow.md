# Data Flow

This document describes how data flows through the application, from user interaction to database and back.

## Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                          User Interaction                         │
│                     (Click, Type, Navigate)                       │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                           Component                               │
│                    (Event Handler Triggered)                      │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                          React Query                              │
│              (Query/Mutation Initiated or Invalidated)            │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                           Service                                 │
│                   (Business Logic Executed)                       │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                          Supabase                                 │
│                    (Database Operation)                           │
└──────────────────────────────────────────────────────────────────┘
```

---

## Query Flow (Read Operations)

### Example: Loading Provider List

```
1. Component mounts
       │
       ▼
2. useProviders() hook called
       │
       ▼
3. React Query checks cache
       │
       ├── Cache hit (fresh) ──► Return cached data
       │
       └── Cache miss/stale ──► Continue to step 4
       │
       ▼
4. queryFn() executes: providerService.search()
       │
       ▼
5. Supabase query: supabase.from('providers').select()
       │
       ▼
6. Database returns data (filtered by RLS)
       │
       ▼
7. Service returns data
       │
       ▼
8. React Query caches data
       │
       ▼
9. Component re-renders with data
```

### Code Example

```typescript
// Component
function ProviderList() {
  const { data, isLoading, error } = useProviders(filters);

  if (isLoading) return <Skeleton />;
  if (error) return <ErrorMessage error={error} />;
  return <Grid items={data} />;
}

// Hook
function useProviders(filters: Filters) {
  return useQuery({
    queryKey: ['providers', filters],
    queryFn: () => providerService.search(filters),
    staleTime: 5 * 60 * 1000,
  });
}

// Service
const providerService = {
  async search(filters: Filters) {
    const { data, error } = await supabase
      .from('providers')
      .select('*')
      .match(filters);

    if (error) throw error;
    return data;
  },
};
```

---

## Mutation Flow (Write Operations)

### Example: Favoriting a Provider

```
1. User clicks "Favorite" button
       │
       ▼
2. onClick handler calls mutation.mutate(providerId)
       │
       ▼
3. onMutate callback (optimistic update)
   - Cancel outgoing refetches
   - Snapshot previous state
   - Update cache optimistically
       │
       ▼
4. mutationFn executes: favoriteService.toggle()
       │
       ▼
5. Supabase insert/delete: supabase.from('favorites')...
       │
       ├── Success ──► onSuccess callback
       │                - Invalidate related queries
       │                - Show success toast
       │
       └── Failure ──► onError callback
                       - Roll back optimistic update
                       - Show error toast
       │
       ▼
6. onSettled callback (always runs)
   - Refetch to ensure sync with server
       │
       ▼
7. UI reflects final state
```

### Code Example

```typescript
// Component
function FavoriteButton({ providerId }: Props) {
  const toggleFavorite = useToggleFavorite();

  return (
    <Button
      onClick={() => toggleFavorite.mutate(providerId)}
      disabled={toggleFavorite.isPending}
    >
      {toggleFavorite.isPending ? 'Saving...' : 'Favorite'}
    </Button>
  );
}

// Hook
function useToggleFavorite() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: favoriteService.toggle,

    onMutate: async (providerId) => {
      await queryClient.cancelQueries({ queryKey: ['favorites'] });
      const previous = queryClient.getQueryData(['favorites']);

      queryClient.setQueryData(['favorites'], (old: string[] = []) =>
        old.includes(providerId)
          ? old.filter(id => id !== providerId)
          : [...old, providerId]
      );

      return { previous };
    },

    onError: (err, providerId, context) => {
      queryClient.setQueryData(['favorites'], context?.previous);
      toast.error('Failed to update favorite');
    },

    onSuccess: () => {
      toast.success('Favorite updated');
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['favorites'] });
    },
  });
}
```

---

## Form Submission Flow

### Example: Profile Update

```
1. User fills form fields
       │
       ▼
2. User clicks "Save"
       │
       ▼
3. Form validation (Zod schema)
       │
       ├── Invalid ──► Show validation errors
       │
       └── Valid ──► Continue to step 4
       │
       ▼
4. useActionState or mutation.mutate() called
       │
       ▼
5. Service validates business rules
       │
       ├── Invalid ──► Return error result
       │
       └── Valid ──► Continue to step 6
       │
       ▼
6. Supabase update: supabase.from('profiles').update()
       │
       ├── RLS blocks ──► 403 error
       │
       └── Allowed ──► Update succeeds
       │
       ▼
7. Invalidate profile queries
       │
       ▼
8. Show success message, redirect if needed
```

### Code Example

```typescript
// Using useActionState (React 19)
async function updateProfile(
  prevState: FormState,
  formData: FormData
): Promise<FormState> {
  // 1. Validate
  const result = profileSchema.safeParse(Object.fromEntries(formData));
  if (!result.success) {
    return {
      success: false,
      errors: result.error.flatten().fieldErrors,
    };
  }

  // 2. Update database
  const { error } = await supabase
    .from('profiles')
    .update(result.data)
    .eq('id', userId);

  if (error) {
    return { success: false, message: error.message };
  }

  // 3. Invalidate cache
  queryClient.invalidateQueries({ queryKey: ['profile'] });

  return { success: true, message: 'Profile updated!' };
}

function ProfileForm() {
  const [state, formAction, isPending] = useActionState(updateProfile, {
    success: false,
  });

  return (
    <form action={formAction}>
      {/* form fields */}
      <button type="submit" disabled={isPending}>
        {isPending ? 'Saving...' : 'Save'}
      </button>
      {state.message && <p>{state.message}</p>}
    </form>
  );
}
```

---

## Real-time Updates Flow

### Example: Chat Messages

```
1. Subscribe to channel on component mount
       │
       ▼
2. Supabase Realtime connection established
       │
       ▼
3. Listen for postgres_changes events
       │
       ├── INSERT ──► Add message to local state
       │
       ├── UPDATE ──► Update message in local state
       │
       └── DELETE ──► Remove message from local state
       │
       ▼
4. UI updates automatically
       │
       ▼
5. Unsubscribe on component unmount
```

### Code Example

```typescript
function useMessageSubscription(conversationId: string) {
  const queryClient = useQueryClient();

  useEffect(() => {
    const channel = supabase
      .channel(`messages:${conversationId}`)
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'messages',
          filter: `conversation_id=eq.${conversationId}`,
        },
        (payload) => {
          queryClient.setQueryData(
            ['messages', conversationId],
            (old: Message[] = []) => [...old, payload.new as Message]
          );
        }
      )
      .subscribe();

    return () => {
      channel.unsubscribe();
    };
  }, [conversationId, queryClient]);
}
```

---

## Error Flow

```
Error occurs at any layer
       │
       ├── Supabase error ──► Thrown by service
       │
       ├── Network error ──► Caught by React Query
       │
       └── Validation error ──► Returned to form
       │
       ▼
React Query handles:
  - Retry logic (configurable)
  - Error state in hook result
  - ErrorBoundary integration (throwOnError)
       │
       ▼
Component displays error:
  - Inline error message
  - Error boundary fallback
  - Toast notification
```

---

## Cache Invalidation Patterns

### After Mutation

```typescript
// Invalidate specific query
queryClient.invalidateQueries({ queryKey: ['profile', id] });

// Invalidate by prefix (all lists)
queryClient.invalidateQueries({ queryKey: ['providers'] });

// Set data directly
queryClient.setQueryData(['profile', id], newData);
```

### Cross-Entity Invalidation

```typescript
// When favoriting a provider, invalidate both
useMutation({
  mutationFn: favoriteService.toggle,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['favorites'] });
    queryClient.invalidateQueries({ queryKey: ['providers'] }); // If favorite count shown
  },
});
```
