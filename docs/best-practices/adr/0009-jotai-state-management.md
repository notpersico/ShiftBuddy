# ADR-0009: Jotai State Management

## Status

**Accepted** - Supersedes ADR-0006 (Split Contexts)

## Context

The application originally used split React Contexts for client state (see ADR-0006). While an improvement over a single "God Context", this still had issues:

- **Provider nesting**: Multiple providers stacked in the app root
- **Re-render scope**: Entire subtree re-renders when any value in a context changes
- **Boilerplate**: Each context requires a provider, hook, and state file
- **Testing overhead**: Mocking nested providers in tests is verbose

Jotai provides atomic state management with granular subscriptions, no provider nesting, and minimal boilerplate.

## Decision

Use **Jotai** for all client-side state management. Atoms are organized by domain in `src/atoms/`.

```
src/atoms/
  auth/      # Session, user profile, roles
  filters/   # Search filters with URL sync
  ui/        # Sidebar, modals, toasts
  language/  # i18n locale
```

Key patterns:

```typescript
// Atom definition with reset capability
import { atomWithReset } from 'jotai/utils';

export const filtersAtom = atomWithReset<FilterState>(defaultFilters);

// URL sync for filter atoms (bidirectional)
// Filters in URL <-> Jotai atom kept in sync via useFilterUrlSync()

// Usage in components
import { useAtom } from 'jotai';
const [filters, setFilters] = useAtom(filtersAtom);

// Derived/selector atoms
export const activeFilterCountAtom = atom((get) => {
  const filters = get(filtersAtom);
  return Object.values(filters).filter(Boolean).length;
});
```

React Contexts are retained **only** for dependency injection (e.g., Supabase client, query client providers).

## Consequences

### Positive

- **Granular re-renders**: Components subscribe to individual atoms, not entire context trees
- **No provider nesting**: Jotai's `Provider` is optional; atoms work without it
- **Minimal boilerplate**: An atom is a single line; no reducer/action/dispatch pattern
- **URL sync**: Filter atoms sync bidirectionally with TanStack Router search params
- **Easy testing**: Atoms can be set directly in tests without provider wrappers

### Negative

- **New paradigm**: Team must learn Jotai's atom model
- **Debugging**: Less structured than Context (mitigated by Jotai DevTools)
- **Implicit dependencies**: Derived atoms create dependency graphs that aren't visible in imports

### Neutral

- Server state remains in React Query; Jotai handles client state only
- Existing DI-only contexts (QueryClientProvider, Supabase) unchanged

## References

- [Jotai Documentation](https://jotai.org/)
- Atoms are organized by domain in `src/atoms/`
- ADR-0006: Split Contexts (superseded, not included)
