# ADR-0008: TanStack Router

## Status

**Accepted** - Supersedes ADR-0002 (Custom Routing)

## Context

The application originally used a custom `RouterContext` with a switch-case in `App.tsx` (see ADR-0002). As the app grew, this approach showed limitations:

- No type-safe route parameters or search params
- Manual browser history management
- No built-in code splitting per route
- Auth guards implemented ad-hoc per component
- No URL-based state for filters (required manual sync)

TanStack Router provides type-safe routing with first-class support for Zod-validated search params, `beforeLoad` auth guards, and file-based code splitting.

## Decision

Use **TanStack Router** as the routing solution. The route tree is defined in `src/router.tsx`.

Key patterns:

```typescript
// Route definition with Zod search params
const searchRoute = createRoute({
  path: '/search',
  validateSearch: z.object({
    city: z.string().optional(),
    category: z.string().optional(),
  }),
});

// Auth guard via beforeLoad
const dashboardRoute = createRoute({
  path: '/dashboard',
  beforeLoad: ({ context }) => {
    if (!context.auth.session) {
      throw redirect({ to: '/login' });
    }
  },
});

// Type-safe navigation
<Link to="/search" search={{ city: 'Milan' }}>Search</Link>
```

## Consequences

### Positive

- **Type-safe routes**: Route params and search params are typed end-to-end
- **URL-based filter state**: Search params validated with Zod, synced with Jotai atoms
- **Auth guards**: `beforeLoad` centralizes authentication checks per route
- **Code splitting**: Lazy route loading built into the route tree
- **DevTools**: TanStack Router DevTools available for debugging

### Negative

- **Bundle size**: Larger than the custom solution (~20KB vs ~2KB)
- **Learning curve**: Team must learn TanStack Router conventions
- **Dependency**: Tied to TanStack Router release cycle

### Neutral

- Route tree replaces switch-case; same logical structure, different syntax
- Navigation uses `Link` component and `useNavigate` hook instead of custom `navigate()`

## References

- [TanStack Router Documentation](https://tanstack.com/router/latest)
- Route tree defined in `src/router.tsx`
- ADR-0002: Custom Routing (superseded, not included)
