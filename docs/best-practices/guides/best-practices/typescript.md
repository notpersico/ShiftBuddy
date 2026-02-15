# TypeScript Best Practices

Strict TypeScript patterns and conventions for this codebase.

## Configuration

This project uses strict TypeScript:

```json
{
  "compilerOptions": {
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

---

## Core Rules

### No `any` Type

```typescript
// BAD
function processData(data: any) {
  return data.value;
}

// GOOD - use unknown and type guards
function processData(data: unknown): string {
  if (typeof data === 'object' && data !== null && 'value' in data) {
    return String((data as { value: unknown }).value);
  }
  throw new Error('Invalid data');
}

// GOOD - use generics
function processData<T extends { value: string }>(data: T): string {
  return data.value;
}
```

### No `@ts-ignore`

```typescript
// BAD
// @ts-ignore
const result = someFunction();

// GOOD - fix the type issue
const result = someFunction() as ExpectedType;

// Or use proper type guards
if (isExpectedType(result)) {
  // TypeScript now knows the type
}
```

### Prefix Unused Variables

```typescript
// BAD
function handleClick(event: MouseEvent) {
  // event not used - causes warning
  console.log('clicked');
}

// GOOD
function handleClick(_event: MouseEvent) {
  console.log('clicked');
}

// Also works for destructuring
const { id, name: _name } = user;
```

---

## Type Patterns

### Interface vs Type

```typescript
// Use interface for objects that may be extended
interface User {
  id: string;
  name: string;
}

interface AdminUser extends User {
  permissions: string[];
}

// Use type for unions, intersections, and utilities
type Status = 'pending' | 'active' | 'inactive';
type UserWithStatus = User & { status: Status };
type PartialUser = Partial<User>;
```

### Props Interface

```typescript
// Always define Props interface for components
interface Props {
  title: string;
  description?: string;
  onAction: (id: string) => void;
  children: React.ReactNode;
}

export function Card({ title, description, onAction, children }: Props) {
  return (/* ... */);
}
```

### Generic Components

```typescript
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string;
}

export function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}

// Usage
<List
  items={users}
  renderItem={(user) => <span>{user.name}</span>}
  keyExtractor={(user) => user.id}
/>
```

---

## Type Guards

### User-Defined Type Guards

```typescript
interface User {
  type: 'user';
  id: string;
  name: string;
}

interface Admin {
  type: 'admin';
  id: string;
  name: string;
  permissions: string[];
}

type Person = User | Admin;

// Type guard function
function isAdmin(person: Person): person is Admin {
  return person.type === 'admin';
}

// Usage
function greet(person: Person) {
  if (isAdmin(person)) {
    // TypeScript knows person is Admin here
    console.log(`Admin ${person.name} has ${person.permissions.length} permissions`);
  } else {
    // TypeScript knows person is User here
    console.log(`User ${person.name}`);
  }
}
```

### Discriminated Unions

```typescript
type Result<T> =
  | { ok: true; value: T }
  | { ok: false; error: Error };

function handleResult<T>(result: Result<T>) {
  if (result.ok) {
    // TypeScript knows result.value exists
    return result.value;
  } else {
    // TypeScript knows result.error exists
    throw result.error;
  }
}
```

---

## Utility Types

### Common Built-in Types

```typescript
// Partial - all properties optional
type PartialUser = Partial<User>;

// Required - all properties required
type RequiredUser = Required<User>;

// Pick - select specific properties
type UserName = Pick<User, 'name' | 'email'>;

// Omit - exclude specific properties
type UserWithoutId = Omit<User, 'id'>;

// Record - create object type
type UserMap = Record<string, User>;

// ReturnType - get function return type
type FetchResult = ReturnType<typeof fetchUser>;

// Parameters - get function parameter types
type FetchParams = Parameters<typeof fetchUser>;
```

### Custom Utility Types

```typescript
// Make specific properties optional
type WithOptional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

type CreateUserInput = WithOptional<User, 'id' | 'createdAt'>;

// Make specific properties required
type WithRequired<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>>;

type UpdateUserInput = WithRequired<Partial<User>, 'id'>;
```

---

## Zod Integration

### Schema to Type

```typescript
import { z } from 'zod';

// Define schema
const userSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(100),
  email: z.string().email(),
  role: z.enum(['admin', 'user', 'guest']),
});

// Infer type from schema
type User = z.infer<typeof userSchema>;

// Use in validation
function validateUser(data: unknown): User {
  return userSchema.parse(data);
}

// Safe validation
function safeValidateUser(data: unknown): User | null {
  const result = userSchema.safeParse(data);
  return result.success ? result.data : null;
}
```

---

## Import/Export Patterns

### Named Exports (Preferred)

```typescript
// components/Button.tsx
export function Button({ children }: Props) {
  return <button>{children}</button>;
}

// Usage
import { Button } from '@/components/Button';
```

### Default Export (Only for Lazy Loading)

```typescript
// pages/HomePage.tsx
function HomePage() {
  return <div>Home</div>;
}

// Default export for lazy loading
export default HomePage;

// Usage with lazy
const HomePage = lazy(() => import('@/pages/HomePage'));
```

### Type-Only Imports

```typescript
// When importing only types
import type { User, Profile } from '@/types';

// Mixed import
import { userService } from '@/services/user.service';
import type { User } from '@/types';
```

---

## Common Patterns

### Event Handlers

```typescript
// Form events
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault();
};

const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  setValue(e.target.value);
};

// Mouse events
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  console.log(e.clientX, e.clientY);
};

// Keyboard events
const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
  if (e.key === 'Enter') {
    submit();
  }
};
```

### Async Functions

```typescript
// Explicit return type for clarity
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error('Failed to fetch user');
  }
  return response.json();
}

// With error handling
async function safeFetchUser(id: string): Promise<User | null> {
  try {
    return await fetchUser(id);
  } catch {
    return null;
  }
}
```

---

## ESLint Rules

The project enforces:
- `@typescript-eslint/no-explicit-any` - warns on `any` usage
- `@typescript-eslint/no-unused-vars` - with `argsIgnorePattern: '^_'`
- `@typescript-eslint/no-non-null-assertion` - warns on `!` assertions
