# Anti-Patterns to Avoid

These patterns are flagged immediately in code reviews. Each includes why it's problematic and the correct alternative.

## Table of Contents

1. [useEffect for Data Fetching](#1-useeffect-for-data-fetching)
2. [Syncing State](#2-syncing-state)
3. [God Mode Contexts](#3-god-mode-contexts)
4. [Missing useEffect Dependencies](#4-missing-useeffect-dependencies)
5. [Suppressing the Linter](#5-suppressing-the-linter)
6. [Conditional Hook Calls](#6-conditional-hook-calls)
7. [Derived State in useState](#7-derived-state-in-usestate)
8. [Premature Memoization](#8-premature-memoization)
9. [Hardcoded Query Keys](#9-hardcoded-query-keys)
10. [Missing Loading/Error States](#10-missing-loadingerror-states)
11. [Mutating State Directly](#11-mutating-state-directly)
12. [Index as Key](#12-index-as-key)

---

## 1. useEffect for Data Fetching

### BAD

```typescript
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [userId]);
}
```

### Why It's Wrong

- **Race conditions**: Rapid changes cause out-of-order responses
- **No caching**: Every mount re-fetches
- **No deduplication**: Multiple components = multiple requests
- **Strict mode issues**: Double-fires in React 18+ dev mode

### CORRECT

```typescript
function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading, error } = useQuery({
    queryKey: queryKeys.users.byId(userId),
    queryFn: () => userService.getById(userId),
    enabled: !!userId,
  });
}
```

---

## 2. Syncing State

### BAD

```typescript
function EditableList() {
  const { data } = useQuery({ queryKey: ['items'], queryFn: fetchItems });
  const [localItems, setLocalItems] = useState(data ?? []);

  useEffect(() => {
    if (data) setLocalItems(data);
  }, [data]);
}
```

### Why It's Wrong

- **Breaks single source of truth**: Two copies of same data
- **Sync bugs**: Local edits get overwritten
- **Race conditions**: useEffect fires at unexpected times

### CORRECT

```typescript
// Derive during render
function FilteredList() {
  const { data: items } = useQuery({ queryKey: ['items'], queryFn: fetchItems });
  const [filterTerm, setFilterTerm] = useState('');

  const filteredItems = items?.filter((item) =>
    item.name.toLowerCase().includes(filterTerm.toLowerCase())
  );

  return <ItemList items={filteredItems ?? []} />;
}
```

---

## 3. God Mode Contexts

### BAD

```typescript
const AppContext = createContext<{
  user: User | null;
  theme: 'light' | 'dark';
  notifications: Notification[];
  sidebarOpen: boolean;
  cart: CartItem[];
}>({});

// Every component re-renders when ANY piece changes
function ThemeToggle() {
  const { state, dispatch } = useContext(AppContext);
  // Re-renders when notifications, cart, etc. change
}
```

### Why It's Wrong

- **Excessive re-renders**: Unrelated changes trigger updates
- **Poor separation of concerns**: Everything coupled together
- **Testing difficulty**: Must mock entire context

### CORRECT

```typescript
// Split by concern
const ThemeContext = createContext<{ theme: 'light' | 'dark'; toggle: () => void }>();
const AuthContext = createContext<{ user: User | null }>();
const NotificationContext = createContext<{ notifications: Notification[] }>();

// ThemeToggle only re-renders when theme changes
function ThemeToggle() {
  const { theme, toggle } = useTheme();
  return <button onClick={toggle}>{theme}</button>;
}
```

---

## 4. Missing useEffect Dependencies

### BAD

```typescript
useEffect(() => {
  fetchResults(query).then(setResults);
}, []); // Missing 'query' dependency!
```

### Why It's Wrong

- **Stale closures**: Effect captures initial value
- **Silent bugs**: No error, just incorrect behavior

### CORRECT

```typescript
// Add dependency
useEffect(() => {
  fetchResults(query).then(setResults);
}, [query]);

// Or use React Query (preferred)
const { data: results } = useQuery({
  queryKey: ['search', query],
  queryFn: () => fetchResults(query),
});
```

---

## 5. Suppressing the Linter

### BAD

```typescript
useEffect(() => {
  const handler = () => console.log(items.length);
  window.addEventListener('resize', handler);
  return () => window.removeEventListener('resize', handler);
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []);
```

### Why It's Wrong

- **Stale data**: Handler always logs initial `items.length`
- **Hides bugs**: Linter exists to catch these issues

### CORRECT

```typescript
// Include dependency
useEffect(() => {
  const handler = () => console.log(items.length);
  window.addEventListener('resize', handler);
  return () => window.removeEventListener('resize', handler);
}, [items.length]);

// Or use ref if you need latest value without re-subscribing
const itemsRef = useRef(items);
itemsRef.current = items;

useEffect(() => {
  const handler = () => console.log(itemsRef.current.length);
  window.addEventListener('resize', handler);
  return () => window.removeEventListener('resize', handler);
}, []);
```

---

## 6. Conditional Hook Calls

### BAD

```typescript
function UserProfile({ userId }: { userId: string | null }) {
  if (!userId) {
    return <LoginPrompt />;
  }

  // Hook called conditionally - BREAKS RULES OF HOOKS
  const { data: user } = useQuery({ queryKey: ['user', userId], queryFn: () => fetchUser(userId) });
}
```

### Why It's Wrong

- **Breaks Rules of Hooks**: Hooks must be called in same order every render
- **React throws errors**: Crashes in development

### CORRECT

```typescript
function UserProfile({ userId }: { userId: string | null }) {
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId!),
    enabled: !!userId, // Disable query when no userId
  });

  if (!userId) {
    return <LoginPrompt />;
  }

  return <Profile user={user} />;
}
```

---

## 7. Derived State in useState

### BAD

```typescript
function ProductList({ products, category }: Props) {
  const [filteredProducts, setFilteredProducts] = useState<Product[]>([]);

  useEffect(() => {
    setFilteredProducts(products.filter((p) => p.category === category));
  }, [products, category]);
}
```

### Why It's Wrong

- **Extra complexity**: State, effect, extra render
- **Out-of-sync risk**: Brief moment where data is stale

### CORRECT

```typescript
function ProductList({ products, category }: Props) {
  // Compute during render
  const filteredProducts = products.filter((p) => p.category === category);
  return <Grid items={filteredProducts} />;
}
```

---

## 8. Premature Memoization

### BAD

```typescript
function SimpleForm() {
  const [name, setName] = useState('');
  const isValid = useMemo(() => name.length > 0, [name]); // Unnecessary
  const handleChange = useCallback((e) => setName(e.target.value), []); // Unnecessary
}
```

### Why It's Wrong

- **Adds complexity**: More code to maintain
- **May be slower**: useMemo/useCallback have overhead
- **React 19**: Compiler handles this automatically

### CORRECT

```typescript
function SimpleForm() {
  const [name, setName] = useState('');
  const isValid = name.length > 0; // Just compute it
  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}
```

---

## 9. Hardcoded Query Keys

### BAD

```typescript
useQuery({ queryKey: ['users', 'profile'], queryFn: fetchProfile });
queryClient.invalidateQueries({ queryKey: ['users', 'profiles'] }); // Typo!
```

### Why It's Wrong

- **Typos break caching**: Similar keys don't match
- **No type safety**: Strings don't catch mismatches

### CORRECT

```typescript
// Query key factory
export const queryKeys = {
  users: {
    profile: () => ['users', 'profile'] as const,
  },
};

// Usage everywhere
useQuery({ queryKey: queryKeys.users.profile(), queryFn: fetchProfile });
queryClient.invalidateQueries({ queryKey: queryKeys.users.profile() });
```

---

## 10. Missing Loading/Error States

### BAD

```typescript
function UserList() {
  const { data } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
  // Crashes if data is undefined during loading
  return <ul>{data.map((user) => <li key={user.id}>{user.name}</li>)}</ul>;
}
```

### Why It's Wrong

- **Runtime crashes**: `data` is undefined until resolved
- **Poor UX**: User sees broken UI

### CORRECT

```typescript
function UserList() {
  const { data, isLoading, isError, error } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });

  if (isLoading) return <UserListSkeleton />;
  if (isError) return <ErrorMessage error={error} />;
  if (!data?.length) return <EmptyState message="No users found" />;

  return <ul>{data.map((user) => <li key={user.id}>{user.name}</li>)}</ul>;
}
```

---

## 11. Mutating State Directly

### BAD

```typescript
const addTodo = (text: string) => {
  todos.push({ id: Date.now(), text }); // Direct mutation!
  setTodos(todos); // Same reference, React won't re-render
};
```

### Why It's Wrong

- **No re-render**: React uses reference equality
- **Unpredictable bugs**: Sometimes works, sometimes doesn't

### CORRECT

```typescript
const addTodo = (text: string) => {
  setTodos((prev) => [...prev, { id: Date.now(), text }]);
};
```

---

## 12. Index as Key

### BAD

```typescript
{todos.map((todo, index) => <li key={index}>{todo.text}</li>)}
```

### Why It's Wrong

- **Reordering breaks**: Wrong items get wrong state
- **Deletions break**: Removing item causes state mismatch

### CORRECT

```typescript
{todos.map((todo) => <li key={todo.id}>{todo.text}</li>)}
```

---

## Quick Reference

| Anti-Pattern | Problem | Solution |
|-------------|---------|----------|
| useEffect for fetching | Race conditions, no cache | TanStack Query |
| Syncing state | Dual source of truth | Derive during render |
| God contexts | Excessive re-renders | Split contexts |
| Missing deps | Stale closures | Add all dependencies |
| Suppressing linter | Hidden bugs | Fix the root cause |
| Conditional hooks | Breaks React | Use `enabled` option |
| Derived state in useState | Extra renders | Compute inline |
| Unnecessary memoization | Complexity, overhead | Only when measured |
| Hardcoded query keys | Typos, no type safety | Query key factory |
| Missing loading/error | Crashes, poor UX | Handle all states |
| Mutating state | No re-render | Immutable updates |
| Index as key | Broken state | Stable unique IDs |
