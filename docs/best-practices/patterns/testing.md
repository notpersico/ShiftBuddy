# Testing Patterns

Testing conventions used throughout the codebase.

## Test File Location

Tests are co-located with source files:

```
src/
├── hooks/
│   └── api/
│       ├── useProfiles.ts
│       └── __tests__/
│           └── useProfiles.test.ts
├── components/
│   └── ProfileCard.tsx
│   └── __tests__/
│       └── ProfileCard.test.tsx
```

---

## Test Structure

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

describe('ProfileCard', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  describe('rendering', () => {
    it('displays profile name', () => {
      // Arrange
      const profile = aProfile().withName('Jane').build();

      // Act
      render(<ProfileCard profile={profile} />);

      // Assert
      expect(screen.getByText('Jane')).toBeInTheDocument();
    });
  });

  describe('interactions', () => {
    it('calls onFavorite when favorite button clicked', async () => {
      const user = userEvent.setup();
      const onFavorite = vi.fn();

      render(<ProfileCard profile={aProfile().build()} onFavorite={onFavorite} />);

      await user.click(screen.getByRole('button', { name: /favorite/i }));

      expect(onFavorite).toHaveBeenCalled();
    });
  });
});
```

---

## Query Client Wrapper

```typescript
function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: { retry: false, gcTime: 0 },
      mutations: { retry: false },
    },
  });
}

function createWrapper() {
  const queryClient = createTestQueryClient();
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
}

// Usage
const { result } = renderHook(() => useProfiles(), { wrapper: createWrapper() });
```

---

## Test Data Builders

```typescript
// src/test/builders/profile-builder.ts
export function aProfile() {
  return new ProfileBuilder();
}

class ProfileBuilder {
  private profile: Partial<Profile> = {
    id: 'test-id',
    name: 'Test User',
    email: 'test@example.com',
    isActive: true,
  };

  withId(id: string) {
    this.profile.id = id;
    return this;
  }

  withName(name: string) {
    this.profile.name = name;
    return this;
  }

  isPremium() {
    this.profile.isPremium = true;
    return this;
  }

  isInactive() {
    this.profile.isActive = false;
    return this;
  }

  build(): Profile {
    return this.profile as Profile;
  }
}
```

---

## MSW Handler Pattern

```typescript
// src/test/msw/handlers.ts
import { http, HttpResponse } from 'msw';

export const profileHandlers = [
  http.get('*/rest/v1/profiles', () => {
    return HttpResponse.json([
      aProfile().withName('John').build(),
      aProfile().withName('Jane').build(),
    ]);
  }),

  http.get('*/rest/v1/profiles', ({ request }) => {
    const url = new URL(request.url);
    const idFilter = url.searchParams.get('id');

    if (idFilter?.startsWith('eq.')) {
      const id = idFilter.replace('eq.', '');
      return HttpResponse.json(aProfile().withId(id).build());
    }

    return HttpResponse.json([]);
  }),
];
```

---

## Testing Hooks

```typescript
describe('useProfiles', () => {
  it('fetches profiles', async () => {
    vi.mocked(profileService.search).mockResolvedValue([
      aProfile().withName('Jane').build(),
    ]);

    const { result } = renderHook(() => useProfiles(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => expect(result.current.isSuccess).toBe(true));
    expect(result.current.data?.[0].name).toBe('Jane');
  });

  it('handles errors', async () => {
    vi.mocked(profileService.search).mockRejectedValue(new Error('Failed'));

    const { result } = renderHook(() => useProfiles(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => expect(result.current.isError).toBe(true));
    expect(result.current.error?.message).toBe('Failed');
  });
});
```

---

## Testing Components

```typescript
describe('ProfileList', () => {
  it('shows loading state', () => {
    render(<ProfileList />, { wrapper: createWrapper() });
    expect(screen.getByTestId('skeleton')).toBeInTheDocument();
  });

  it('shows profiles when loaded', async () => {
    render(<ProfileList />, { wrapper: createWrapper() });

    await waitFor(() => {
      expect(screen.getByText('John')).toBeInTheDocument();
    });
  });

  it('shows empty state', async () => {
    server.use(
      http.get('*/rest/v1/profiles*', () => HttpResponse.json([]))
    );

    render(<ProfileList />, { wrapper: createWrapper() });

    await waitFor(() => {
      expect(screen.getByText('No profiles found')).toBeInTheDocument();
    });
  });
});
```

---

## Mock Patterns

### Service Mocks

```typescript
vi.mock('@/services/api/profile.service', () => ({
  profileService: {
    search: vi.fn(),
    getById: vi.fn(),
    update: vi.fn(),
  },
}));
```

### Context Mocks

```typescript
vi.mock('@/contexts/AuthContext', () => ({
  useAuth: vi.fn(() => ({
    user: { id: 'test-user' },
    isAuthenticated: true,
  })),
}));
```
