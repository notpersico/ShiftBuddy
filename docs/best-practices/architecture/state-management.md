# State Management

This codebase uses the "State Ladder" approach - each type of state has a specific home based on its characteristics.

## Decision Flowchart

```
Is this data from the database/API?
    YES → Use TanStack Query (Server State)
    NO  → Continue...

Should this state persist on page refresh?
    YES → Use URL State (searchParams)
    NO  → Continue...

Is this state shared across multiple components?
    YES → Is it performance-critical with frequent updates?
        YES → Use Zustand/Jotai
        NO  → Use React Context + useReducer
    NO  → Use useState (Local State)
```

---

## 1. Server State (TanStack Query)

**Rule**: All data from the database/API is managed by TanStack Query.

### Why Not useState?

| Problem | TanStack Query Solution |
|---------|------------------------|
| Race conditions | Automatic request deduplication |
| No caching | Built-in cache with stale time |
| Manual refetching | Background refetching |
| Loading/error states | Built-in status flags |
| Memory leaks | Automatic garbage collection |

### Query Key Factory

We use a centralized query key factory in `src/hooks/api/query-keys.ts`:

```typescript
export const queryKeys = {
  providers: (() => {
    const base = createKeyFactory(['providers'] as const);
    return {
      all: base.all,                                    // ['providers']
      lists: () => base.extend('list'),                 // ['providers', 'list']
      list: (filters: Record<string, unknown>) =>       // ['providers', 'list', { city: 'NYC' }]
        base.extend('list', filters),
      details: () => base.extend('detail'),             // ['providers', 'detail']
      detail: (id: string) => base.extend('detail', id), // ['providers', 'detail', '123']
    };
  })(),
};
```

### Usage Pattern

```typescript
// Query hook
export function useProviders(filters?: ProviderFilters) {
  return useQuery({
    queryKey: queryKeys.providers.list(filters ?? {}),
    queryFn: () => providerService.search(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
}

// Mutation with cache invalidation
export function useUpdateProvider() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: providerService.update,
    onSuccess: (data, variables) => {
      queryClient.setQueryData(
        queryKeys.providers.detail(variables.id),
        data
      );
      queryClient.invalidateQueries({
        queryKey: queryKeys.providers.lists(),
      });
    },
  });
}
```

---

## 2. URL State

**Rule**: State that should persist on refresh or be shareable goes in the URL.

### When to Use

- Search filters
- Pagination
- Sorting
- Active tabs
- Selected items (for deep linking)

### Implementation

```typescript
// hooks/useFilterParams.ts
export function useFilterParams() {
  const [searchParams, setSearchParams] = useSearchParams();

  const filters = {
    category: searchParams.get('category') ?? undefined,
    minPrice: searchParams.get('minPrice')
      ? Number(searchParams.get('minPrice'))
      : undefined,
    page: searchParams.get('page')
      ? Number(searchParams.get('page'))
      : 1,
  };

  const setFilters = (updates: Partial<typeof filters>) => {
    const params = new URLSearchParams(searchParams);
    Object.entries(updates).forEach(([key, value]) => {
      if (value === undefined) {
        params.delete(key);
      } else {
        params.set(key, String(value));
      }
    });
    setSearchParams(params);
  };

  return { filters, setFilters };
}
```

### URL State Drives Queries

```typescript
function ProviderList() {
  const { filters } = useFilterParams(); // URL state

  const { data } = useQuery({
    queryKey: queryKeys.providers.list(filters),
    queryFn: () => providerService.list(filters),
    // URL changes trigger new queries automatically
  });

  return <ProviderGrid data={data} />;
}
```

---

## 3. Shared Client State (Context)

**Rule**: State shared across components that doesn't need URL persistence.

### When to Use

- Auth state
- Theme settings
- Complex form state
- Feature flags

### Implementation

```typescript
// contexts/FilterContext.tsx
interface FilterState {
  isExpanded: boolean;
  activeSection: string | null;
}

type FilterAction =
  | { type: 'TOGGLE_EXPANDED' }
  | { type: 'SET_SECTION'; section: string | null };

function filterReducer(state: FilterState, action: FilterAction): FilterState {
  switch (action.type) {
    case 'TOGGLE_EXPANDED':
      return { ...state, isExpanded: !state.isExpanded };
    case 'SET_SECTION':
      return { ...state, activeSection: action.section };
    default:
      return state;
  }
}

export function FilterProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(filterReducer, initialState);

  const value = useMemo(() => ({ state, dispatch }), [state]);

  return (
    <FilterContext.Provider value={value}>
      {children}
    </FilterContext.Provider>
  );
}
```

### When to Split Contexts

Split contexts when different parts of state update independently:

```typescript
// BAD: God context (everything re-renders together)
const AppContext = createContext<{
  user: User;
  theme: Theme;
  notifications: Notification[];
}>(...);

// GOOD: Split contexts
const AuthContext = createContext<{ user: User }>(...);
const ThemeContext = createContext<{ theme: Theme }>(...);
const NotificationContext = createContext<{ notifications: Notification[] }>(...);
```

---

## 4. Local State (useState)

**Rule**: State that belongs to a single component.

### When to Use

- Form input values
- Modal open/close
- Hover/focus states
- Animation states

### Example

```typescript
function SearchInput({ onSearch }: { onSearch: (term: string) => void }) {
  const [value, setValue] = useState(''); // Local to this component

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    onSearch(value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={value}
        onChange={(e) => setValue(e.target.value)}
      />
    </form>
  );
}
```

---

## State Synchronization

### Derived State

**Rule**: Compute during render, don't sync with useEffect.

```typescript
// BAD: Syncing with useEffect
function ProductList({ products, category }) {
  const [filtered, setFiltered] = useState([]);

  useEffect(() => {
    setFiltered(products.filter(p => p.category === category));
  }, [products, category]);

  return <List items={filtered} />;
}

// GOOD: Derive during render
function ProductList({ products, category }) {
  const filtered = products.filter(p => p.category === category);
  return <List items={filtered} />;
}
```

### Expensive Derived State

```typescript
// Use useMemo only when computation is genuinely expensive
function DataGrid({ items, sortConfig }) {
  const sortedItems = useMemo(
    () => [...items].sort((a, b) => /* complex sorting */),
    [items, sortConfig]
  );

  return <Table data={sortedItems} />;
}
```

---

## Quick Reference

| State Type | Persistence | Tool | Example |
|------------|-------------|------|---------|
| Server | Database | TanStack Query | User profiles, listings |
| URL | Survives refresh | useSearchParams | Filters, pagination |
| Shared | Session only | Context + useReducer | Auth, theme |
| Local | Component only | useState | Form inputs, modals |

---

## Codebase Patterns

| Pattern | Location | Purpose |
|---------|----------|---------|
| Query Key Factory | `src/hooks/api/query-keys.ts` | Centralized, type-safe query keys |
| Auth Context | `src/contexts/AuthContext.tsx` | Authentication state |
| Router Context | `src/contexts/RouterContext.tsx` | Navigation state |
| Filter Context | `src/contexts/FilterContext.tsx` | Complex filter state |
