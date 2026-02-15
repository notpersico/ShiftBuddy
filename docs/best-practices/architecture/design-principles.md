# Design Principles

## Core Principles

### Single Responsibility

Each module, class, or function should have one reason to change.

```typescript
// BAD: Component handles data fetching and UI
function UserProfile() {
  const [user, setUser] = useState(null);
  useEffect(() => { fetch('/api/user').then(...) }, []);
  return <div>{user?.name}</div>;
}

// GOOD: Separation of concerns
function UserProfile() {
  const { data: user } = useUserProfile(); // Hook handles data
  return <ProfileView user={user} />;       // Component handles UI
}
```

### Open/Closed Principle

Software entities should be open for extension but closed for modification.

```typescript
// GOOD: Extend behavior via composition
interface Validator {
  validate(value: unknown): boolean;
}

const emailValidator: Validator = {
  validate: (value) => /^[^@]+@[^@]+$/.test(String(value)),
};

const requiredValidator: Validator = {
  validate: (value) => value !== null && value !== undefined && value !== '',
};

// Compose validators without modifying existing ones
const validators = [requiredValidator, emailValidator];
```

### Dependency Inversion

High-level modules should not depend on low-level modules. Both should depend on abstractions.

```typescript
// BAD: Direct Supabase dependency in component
function UserList() {
  const users = await supabase.from('users').select('*');
  return <List items={users} />;
}

// GOOD: Abstraction through service layer
function UserList() {
  const { data: users } = useUsers(); // Abstraction
  return <List items={users} />;
}

// Service can be mocked, replaced, or tested independently
export const userService = {
  getAll: () => supabase.from('users').select('*'),
};
```

---

## React-Specific Principles

### Composition Over Inheritance

Build complex UIs by composing simple components.

```typescript
// GOOD: Composable components
function Dialog({ children }: { children: React.ReactNode }) {
  return <div className="dialog">{children}</div>;
}

function DialogHeader({ children }: { children: React.ReactNode }) {
  return <header className="dialog-header">{children}</header>;
}

function DialogContent({ children }: { children: React.ReactNode }) {
  return <div className="dialog-content">{children}</div>;
}

// Usage: Compose as needed
<Dialog>
  <DialogHeader>Title</DialogHeader>
  <DialogContent>Body text</DialogContent>
</Dialog>
```

### Unidirectional Data Flow

Data flows down through props; events flow up through callbacks.

```typescript
// Parent owns the state
function FilterContainer() {
  const [filters, setFilters] = useState(initialFilters);

  return (
    <>
      <FilterPanel
        filters={filters}
        onFilterChange={setFilters} // Events flow up
      />
      <ResultsList filters={filters} /> {/* Data flows down */}
    </>
  );
}
```

### Render Pure, Fetch with Hooks

Components should be pure functions of their props. Side effects belong in hooks.

```typescript
// GOOD: Pure component + hook for side effects
function ProductCard({ product }: { product: Product }) {
  // No side effects - pure render
  return (
    <Card>
      <h2>{product.name}</h2>
      <p>{product.price}</p>
    </Card>
  );
}

function ProductList() {
  const { data: products } = useProducts(); // Side effects in hook
  return products?.map(p => <ProductCard key={p.id} product={p} />);
}
```

---

## State Management Principles

### Derive, Don't Sync

Calculate derived state during render instead of syncing with useEffect.

```typescript
// BAD: Syncing derived state
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

### Single Source of Truth

Each piece of state should have exactly one owner.

| State Type | Owner | Location |
|------------|-------|----------|
| Server data | React Query | Cache |
| URL state | Browser | URL params |
| Form state | Component | useState |
| Auth state | Context | AuthContext |

### Colocation

Keep state as close as possible to where it's used.

```typescript
// BAD: Global state for local concern
const globalModalState = createStore({ isOpen: false });

// GOOD: Local state for local concern
function ConfirmDialog() {
  const [isOpen, setIsOpen] = useState(false);
  return /* ... */;
}
```

---

## Error Handling Principles

### Fail Fast

Validate early and fail with clear error messages.

```typescript
// GOOD: Validate at boundaries
async function createUser(input: unknown) {
  const result = userSchema.safeParse(input);

  if (!result.success) {
    throw new ValidationError(result.error.issues);
  }

  return userService.create(result.data);
}
```

### Result Pattern

Use Result types instead of throwing for expected failures.

```typescript
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

// Service returns Result
async function getUser(id: string): Promise<Result<User>> {
  const { data, error } = await supabase
    .from('users')
    .select('*')
    .eq('id', id)
    .single();

  if (error) return { ok: false, error };
  return { ok: true, value: data };
}

// Consumer handles both cases
const result = await getUser(id);
if (!result.ok) {
  // Handle error
}
// Use result.value safely
```

---

## Performance Principles

### Measure First

Only optimize what you've measured to be slow.

```typescript
// DON'T: Premature optimization
const value = useMemo(() => simple + calculation, [simple, calculation]);

// DO: Profile first, then optimize if needed
const value = simple + calculation;

// DO: Optimize when profiler shows it's necessary
const expensiveValue = useMemo(
  () => sort1000Items(items),
  [items]
);
```

### Minimize Re-renders

Use React 19's automatic memoization; only add manual optimization when measured.

```typescript
// React 19: Compiler handles memoization
function Component({ data }) {
  const processed = expensiveOperation(data);
  return <div>{processed}</div>;
}

// Manual optimization only when compiler can't help
// (e.g., third-party library requirements)
```
