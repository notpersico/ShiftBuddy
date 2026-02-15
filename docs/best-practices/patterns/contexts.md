# Context Patterns

Patterns for React Context usage.

## Basic Context Pattern

```typescript
import { createContext, useContext, useState, useMemo, type ReactNode } from 'react';

interface ThemeContextValue {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('dark');

  const toggleTheme = () => {
    setTheme((t) => (t === 'light' ? 'dark' : 'light'));
  };

  const value = useMemo(() => ({ theme, toggleTheme }), [theme]);

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}
```

---

## Context with Reducer

For complex state with multiple actions:

```typescript
interface FilterState {
  location: string | null;
  minPrice: number | null;
  maxPrice: number | null;
  services: string[];
}

type FilterAction =
  | { type: 'SET_LOCATION'; location: string | null }
  | { type: 'SET_PRICE_RANGE'; min: number | null; max: number | null }
  | { type: 'TOGGLE_SERVICE'; service: string }
  | { type: 'RESET' };

const initialState: FilterState = {
  location: null,
  minPrice: null,
  maxPrice: null,
  services: [],
};

function filterReducer(state: FilterState, action: FilterAction): FilterState {
  switch (action.type) {
    case 'SET_LOCATION':
      return { ...state, location: action.location };
    case 'SET_PRICE_RANGE':
      return { ...state, minPrice: action.min, maxPrice: action.max };
    case 'TOGGLE_SERVICE':
      return {
        ...state,
        services: state.services.includes(action.service)
          ? state.services.filter((s) => s !== action.service)
          : [...state.services, action.service],
      };
    case 'RESET':
      return initialState;
    default:
      return state;
  }
}

export function FilterProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(filterReducer, initialState);

  const value = useMemo(() => ({ state, dispatch }), [state]);

  return <FilterContext.Provider value={value}>{children}</FilterContext.Provider>;
}
```

---

## Split Context Pattern

Separate read and write for better performance:

```typescript
const AuthStateContext = createContext<AuthState | null>(null);
const AuthActionsContext = createContext<AuthActions | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  const state = useMemo(() => ({ user, isLoading }), [user, isLoading]);

  const actions = useMemo(
    () => ({
      login: async (email: string, password: string) => {
        // Login logic
      },
      logout: async () => {
        // Logout logic
      },
    }),
    []
  );

  return (
    <AuthStateContext.Provider value={state}>
      <AuthActionsContext.Provider value={actions}>
        {children}
      </AuthActionsContext.Provider>
    </AuthStateContext.Provider>
  );
}

// Separate hooks for state vs actions
export function useAuthState() {
  const context = useContext(AuthStateContext);
  if (!context) throw new Error('useAuthState must be used within AuthProvider');
  return context;
}

export function useAuthActions() {
  const context = useContext(AuthActionsContext);
  if (!context) throw new Error('useAuthActions must be used within AuthProvider');
  return context;
}
```

---

## Context with URL Sync

For state that should sync with URL:

```typescript
export function useFilterParams() {
  const [searchParams, setSearchParams] = useSearchParams();

  const filters = useMemo(
    () => ({
      location: searchParams.get('location') ?? null,
      minPrice: searchParams.get('minPrice')
        ? Number(searchParams.get('minPrice'))
        : null,
      maxPrice: searchParams.get('maxPrice')
        ? Number(searchParams.get('maxPrice'))
        : null,
    }),
    [searchParams]
  );

  const setFilters = (updates: Partial<typeof filters>) => {
    const params = new URLSearchParams(searchParams);
    Object.entries(updates).forEach(([key, value]) => {
      if (value === null || value === undefined) {
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

---

## Provider Composition

Compose multiple providers cleanly:

```typescript
function AppProviders({ children }: { children: ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      <AuthProvider>
        <RouterProvider>
          <ThemeProvider>{children}</ThemeProvider>
        </RouterProvider>
      </AuthProvider>
    </QueryClientProvider>
  );
}

// Or with a compose utility
function composeProviders(...providers: React.ComponentType<{ children: ReactNode }>[]) {
  return providers.reduce(
    (Prev, Curr) =>
      ({ children }) =>
        (
          <Prev>
            <Curr>{children}</Curr>
          </Prev>
        ),
    ({ children }) => <>{children}</>
  );
}

const AppProviders = composeProviders(
  QueryClientProvider,
  AuthProvider,
  RouterProvider,
  ThemeProvider
);
```
