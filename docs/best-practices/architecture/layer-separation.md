# Layer Separation

## Overview

The codebase follows a strict layered architecture where each layer has a single responsibility and communicates only with adjacent layers.

```
┌─────────────────────────────────────────────────────────────┐
│                       Components                             │
│                    (Presentation Layer)                      │
├─────────────────────────────────────────────────────────────┤
│                         Hooks                                │
│                    (Data Access Layer)                       │
├─────────────────────────────────────────────────────────────┤
│                        Services                              │
│                    (Business Logic Layer)                    │
├─────────────────────────────────────────────────────────────┤
│                       Supabase                               │
│                      (Data Layer)                            │
└─────────────────────────────────────────────────────────────┘
```

---

## Components Layer

**Location**: `src/components/`

**Responsibility**: UI rendering only

### Rules

- Accept data via props
- Render JSX based on props
- Emit events via callbacks
- **Never** call services or Supabase directly
- **Never** manage server state

### Example

```typescript
// src/components/provider/ProviderCard.tsx
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
      </CardHeader>
      <CardContent>
        <p>{provider.bio}</p>
      </CardContent>
      <CardFooter>
        <Button onClick={() => onFavorite?.(provider.id)}>
          Favorite
        </Button>
      </CardFooter>
    </Card>
  );
}
```

---

## Hooks Layer

**Location**: `src/hooks/api/`

**Responsibility**: Data fetching and state management

### Rules

- Wrap React Query hooks
- Use query key factory
- Call service layer functions
- Handle loading/error states
- **Never** call Supabase directly

### Example

```typescript
// src/hooks/api/useProviders.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { queryKeys } from './query-keys';
import { providerService } from '@/services/api/provider.service';

export function useProviders(filters?: ProviderFilters) {
  return useQuery({
    queryKey: queryKeys.providers.list(filters ?? {}),
    queryFn: () => providerService.search(filters),
    staleTime: 5 * 60 * 1000,
  });
}

export function useProviderDetail(id: string) {
  return useQuery({
    queryKey: queryKeys.providers.detail(id),
    queryFn: () => providerService.getById(id),
    enabled: !!id,
  });
}

export function useToggleFavorite() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: providerService.toggleFavorite,
    onSuccess: () => {
      queryClient.invalidateQueries({
        queryKey: queryKeys.favorites.all,
      });
    },
  });
}
```

---

## Services Layer

**Location**: `src/services/`

**Responsibility**: Business logic and API communication

### Rules

- Handle Supabase queries
- Transform data as needed
- Implement business logic
- Return typed data
- **Never** use React hooks

### Example

```typescript
// src/services/api/provider.service.ts
import { supabase } from '@/integrations/supabase/client';
import type { Provider, ProviderFilters } from '@/types';

export const providerService = {
  async search(filters?: ProviderFilters) {
    let query = supabase
      .from('provider_profiles')
      .select('*', { count: 'exact' })
      .eq('is_active', true);

    if (filters?.location) {
      query = query.ilike('location', `%${filters.location}%`);
    }

    if (filters?.minRate) {
      query = query.gte('hourly_rate', filters.minRate);
    }

    if (filters?.maxRate) {
      query = query.lte('hourly_rate', filters.maxRate);
    }

    const { data, error, count } = await query;

    if (error) throw error;
    return { data, count };
  },

  async getById(id: string) {
    const { data, error } = await supabase
      .from('provider_profiles')
      .select(`
        *,
        photos:provider_photos(*),
        services:provider_services(*)
      `)
      .eq('id', id)
      .single();

    if (error) throw error;
    return data;
  },

  async toggleFavorite(providerId: string) {
    const { data: existing } = await supabase
      .from('favorites')
      .select('id')
      .eq('provider_id', providerId)
      .single();

    if (existing) {
      await supabase.from('favorites').delete().eq('id', existing.id);
      return { favorited: false };
    }

    await supabase.from('favorites').insert({ provider_id: providerId });
    return { favorited: true };
  },
};
```

---

## Integration Example

Here's how the layers work together in a feature:

```typescript
// Page component connects everything
function ProviderListPage() {
  const { filters } = useFilterParams(); // URL state
  const { data, isLoading } = useProviders(filters); // Hook layer
  const toggleFavorite = useToggleFavorite(); // Mutation hook

  if (isLoading) return <ProviderListSkeleton />;

  return (
    <div className="grid grid-cols-3 gap-4">
      {data?.data.map((provider) => (
        <ProviderCard
          key={provider.id}
          provider={provider}
          onFavorite={(id) => toggleFavorite.mutate(id)}
        />
      ))}
    </div>
  );
}
```

### Data Flow

```
User clicks "Favorite"
        │
        ▼
ProviderCard.onFavorite(id)
        │
        ▼
toggleFavorite.mutate(id)
        │
        ▼
providerService.toggleFavorite(id)
        │
        ▼
supabase.from('favorites')...
        │
        ▼
onSuccess: invalidateQueries
        │
        ▼
UI automatically re-renders
```

---

## Benefits

### Testability

Each layer can be tested independently:

```typescript
// Test service layer without React
describe('providerService', () => {
  it('applies location filter', async () => {
    const result = await providerService.search({ location: 'NYC' });
    expect(mockSupabase.ilike).toHaveBeenCalledWith('location', '%NYC%');
  });
});

// Test hook layer with mocked service
describe('useProviders', () => {
  it('returns providers', async () => {
    vi.mocked(providerService.search).mockResolvedValue({ data: [...] });
    const { result } = renderHook(() => useProviders(), { wrapper });
    await waitFor(() => expect(result.current.isSuccess).toBe(true));
  });
});

// Test component with mocked hook
describe('ProviderCard', () => {
  it('renders provider name', () => {
    render(<ProviderCard provider={mockProvider} />);
    expect(screen.getByText('Jane Doe')).toBeInTheDocument();
  });
});
```

### Replaceability

Any layer can be swapped without affecting others:

- Replace Supabase with another backend
- Replace React Query with SWR
- Replace components with different UI library

### Maintainability

Changes are localized to specific layers:

- UI changes → Components only
- Business logic changes → Services only
- Data fetching patterns → Hooks only
