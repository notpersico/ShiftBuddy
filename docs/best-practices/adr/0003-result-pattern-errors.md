# ADR-0003: Result Pattern for Error Handling

## Status

**Accepted**

## Context

Error handling in JavaScript/TypeScript traditionally uses try-catch:

```typescript
try {
  const user = await getUser(id);
  // use user
} catch (error) {
  // handle error
}
```

This approach has issues:
- Errors can be forgotten (no compile-time enforcement)
- Error types are typically `unknown`
- Control flow is disrupted
- Nesting try-catch blocks becomes unwieldy

## Decision

Use the **Result Pattern** for operations that can fail in expected ways. Services return a Result type that must be handled.

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

// Consumer must handle both cases
const result = await getUser(id);
if (!result.ok) {
  return <ErrorMessage error={result.error} />;
}
// TypeScript knows result.value is User here
return <UserProfile user={result.value} />;
```

## Consequences

### Positive

- **Type safety**: Compiler enforces error handling
- **Explicit failures**: No hidden exceptions
- **Composable**: Results can be chained and transformed
- **Self-documenting**: Function signatures show what can fail

### Negative

- **Verbosity**: More code than try-catch for simple cases
- **Learning curve**: Pattern unfamiliar to some developers
- **Library choice**: No standard Result type (we define our own)

### Neutral

- Unexpected errors (bugs) still throw exceptions
- React Query can handle both patterns

## When to Use Each

| Scenario | Approach |
|----------|----------|
| Expected failures (validation, not found) | Result pattern |
| Unexpected failures (bugs, network issues) | Throw exception |
| React Query mutations | Either (Query handles both) |

## Implementation

```typescript
// src/types/result.ts
export type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

// Helper functions
export function ok<T>(value: T): Result<T, never> {
  return { ok: true, value };
}

export function err<E>(error: E): Result<never, E> {
  return { ok: false, error };
}
```

## Amendment (2026-02-12)

The codebase now has two error handling strategies:

| Layer | Pattern | Rationale |
|-------|---------|-----------|
| `src/services/` | Returns `Result<T, E>` | Original pattern per this ADR |
| `src/modules/*/infrastructure/` | Throws exceptions | Caught by React Query's `onError`; simpler for hexagonal adapters |

Both are acceptable. React Query handles thrown errors gracefully via its `error` state, so modules that interact only through React Query hooks may throw instead of wrapping in Result. The Result pattern remains preferred for service functions called outside of React Query.

## References

- [Rust Result Type](https://doc.rust-lang.org/std/result/)
- [TypeScript Error Handling](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
