# Testing React Query Hooks

Patterns for testing hooks that use TanStack Query.

## Test Setup

### QueryClient Wrapper

Every hook using React Query needs a QueryClient:

```typescript
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

function createWrapper() {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,        // Don't retry in tests
        gcTime: 0,           // Garbage collect immediately
        staleTime: 0,        // Always consider stale
      },
      mutations: {
        retry: false,
      },
    },
  });

  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

### Reusable Test Utility

```typescript
// src/test/utils.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { AuthProvider } from '@/contexts/AuthContext';

export function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: { retry: false, gcTime: 0 },
      mutations: { retry: false },
    },
  });
}

export function createWrapper(options?: { authenticated?: boolean }) {
  const queryClient = createTestQueryClient();

  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {options?.authenticated ? (
        <AuthProvider initialUser={mockUser}>
          {children}
        </AuthProvider>
      ) : (
        children
      )}
    </QueryClientProvider>
  );
}
```

---

## Testing Query Hooks

### Basic Query Test

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { renderHook, waitFor } from '@testing-library/react';
import { useProfiles } from '../useProfiles';
import { profileService } from '@/services/api/profile.service';
import { createWrapper } from '@/test/utils';

vi.mock('@/services/api/profile.service');

describe('useProfiles', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should fetch profiles successfully', async () => {
    const mockProfiles = [
      { id: '1', name: 'John' },
      { id: '2', name: 'Jane' },
    ];
    vi.mocked(profileService.getAll).mockResolvedValue(mockProfiles);

    const { result } = renderHook(() => useProfiles(), {
      wrapper: createWrapper(),
    });

    // Initially loading
    expect(result.current.isLoading).toBe(true);
    expect(result.current.data).toBeUndefined();

    // Wait for success
    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    // Data available
    expect(result.current.data).toEqual(mockProfiles);
    expect(profileService.getAll).toHaveBeenCalledTimes(1);
  });

  it('should handle errors', async () => {
    const error = new Error('Network error');
    vi.mocked(profileService.getAll).mockRejectedValue(error);

    const { result } = renderHook(() => useProfiles(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isError).toBe(true);
    });

    expect(result.current.error).toBe(error);
  });
});
```

### Testing with Filters

```typescript
it('should pass filters to service', async () => {
  const filters = { category: 'premium', city: 'NYC' };
  vi.mocked(profileService.search).mockResolvedValue([]);

  const { result } = renderHook(() => useProfiles(filters), {
    wrapper: createWrapper(),
  });

  await waitFor(() => {
    expect(result.current.isSuccess).toBe(true);
  });

  expect(profileService.search).toHaveBeenCalledWith(filters);
});
```

### Testing Enabled Option

```typescript
it('should not fetch when disabled', async () => {
  const { result } = renderHook(
    () => useProfile(''), // Empty ID should disable
    { wrapper: createWrapper() }
  );

  // Should not be loading when disabled
  expect(result.current.isLoading).toBe(false);
  expect(result.current.fetchStatus).toBe('idle');
  expect(profileService.getById).not.toHaveBeenCalled();
});
```

---

## Testing Mutation Hooks

### Basic Mutation Test

```typescript
describe('useCreateProfile', () => {
  it('should create profile successfully', async () => {
    const newProfile = { name: 'New User', email: 'new@test.com' };
    const createdProfile = { id: '123', ...newProfile };

    vi.mocked(profileService.create).mockResolvedValue(createdProfile);

    const { result } = renderHook(() => useCreateProfile(), {
      wrapper: createWrapper(),
    });

    // Execute mutation
    result.current.mutate(newProfile);

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    expect(result.current.data).toEqual(createdProfile);
    expect(profileService.create).toHaveBeenCalledWith(newProfile);
  });

  it('should handle mutation errors', async () => {
    const error = new Error('Validation failed');
    vi.mocked(profileService.create).mockRejectedValue(error);

    const { result } = renderHook(() => useCreateProfile(), {
      wrapper: createWrapper(),
    });

    result.current.mutate({ name: '', email: '' });

    await waitFor(() => {
      expect(result.current.isError).toBe(true);
    });

    expect(result.current.error).toBe(error);
  });
});
```

### Testing Cache Invalidation

```typescript
it('should invalidate queries on success', async () => {
  const queryClient = createTestQueryClient();
  const invalidateSpy = vi.spyOn(queryClient, 'invalidateQueries');

  vi.mocked(profileService.create).mockResolvedValue({ id: '1', name: 'Test' });

  const wrapper = ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );

  const { result } = renderHook(() => useCreateProfile(), { wrapper });

  result.current.mutate({ name: 'Test' });

  await waitFor(() => {
    expect(result.current.isSuccess).toBe(true);
  });

  expect(invalidateSpy).toHaveBeenCalledWith({
    queryKey: ['profiles'],
  });
});
```

### Testing Optimistic Updates

```typescript
it('should apply optimistic update', async () => {
  const queryClient = createTestQueryClient();

  // Pre-populate cache
  queryClient.setQueryData(['profiles'], [{ id: '1', name: 'John' }]);

  vi.mocked(profileService.update).mockResolvedValue({ id: '1', name: 'Updated' });

  const wrapper = ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );

  const { result } = renderHook(() => useUpdateProfile(), { wrapper });

  // Start mutation
  result.current.mutate({ id: '1', name: 'Updated' });

  // Check optimistic update was applied
  await waitFor(() => {
    const cached = queryClient.getQueryData(['profiles', '1']);
    expect(cached).toMatchObject({ name: 'Updated' });
  });
});
```

---

## Testing with MSW

For more realistic tests, use MSW instead of mocking services:

```typescript
import { http, HttpResponse } from 'msw';
import { server } from '@/test/msw/server';

describe('useProfiles with MSW', () => {
  it('should fetch profiles', async () => {
    // Handler is in default handlers
    const { result } = renderHook(() => useProfiles(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });

    expect(result.current.data).toHaveLength(2);
  });

  it('should handle server error', async () => {
    server.use(
      http.get('*/rest/v1/profiles*', () => {
        return new HttpResponse(null, { status: 500 });
      })
    );

    const { result } = renderHook(() => useProfiles(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isError).toBe(true);
    });
  });
});
```

---

## Common Patterns

### Re-rendering with New Props

```typescript
it('should refetch when ID changes', async () => {
  const { result, rerender } = renderHook(
    ({ id }) => useProfile(id),
    {
      wrapper: createWrapper(),
      initialProps: { id: '1' },
    }
  );

  await waitFor(() => {
    expect(result.current.isSuccess).toBe(true);
  });

  // Change props
  rerender({ id: '2' });

  await waitFor(() => {
    expect(result.current.isFetching).toBe(true);
  });

  await waitFor(() => {
    expect(result.current.isSuccess).toBe(true);
  });
});
```

### Testing Custom Hook Logic

```typescript
it('should derive computed value', async () => {
  vi.mocked(profileService.getAll).mockResolvedValue([
    { id: '1', rating: 4 },
    { id: '2', rating: 5 },
  ]);

  const { result } = renderHook(() => useProfilesWithStats(), {
    wrapper: createWrapper(),
  });

  await waitFor(() => {
    expect(result.current.isSuccess).toBe(true);
  });

  // Test computed value
  expect(result.current.averageRating).toBe(4.5);
});
```

---

## Debugging Tips

1. **Use `screen.debug()`** to see current state
2. **Check `result.current.error`** for error details
3. **Use `waitFor` assertions** not arbitrary delays
4. **Reset handlers in afterEach** to prevent test pollution
