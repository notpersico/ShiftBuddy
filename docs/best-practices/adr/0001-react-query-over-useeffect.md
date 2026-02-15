# ADR-0001: React Query over useEffect for Data Fetching

## Status

**Accepted**

## Context

Data fetching is a core concern in any web application. The traditional React approach uses `useEffect` with `useState` to manage async data:

```typescript
const [data, setData] = useState(null);
const [loading, setLoading] = useState(true);
const [error, setError] = useState(null);

useEffect(() => {
  fetch('/api/data')
    .then(res => res.json())
    .then(setData)
    .catch(setError)
    .finally(() => setLoading(false));
}, []);
```

This approach has several problems:
- **Race conditions**: Rapid state changes can cause responses to arrive out of order
- **No caching**: Every mount triggers a new request
- **No deduplication**: Multiple components requesting same data = multiple requests
- **Strict mode issues**: React 18+ double-fires effects, causing duplicate requests
- **No background refetching**: Data gets stale without user awareness
- **Boilerplate**: Loading, error, and data states must be managed manually

## Decision

Use **TanStack Query (React Query) v5** for all server state management. Components never use `useEffect` for data fetching.

```typescript
const { data, isLoading, error } = useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
});
```

## Consequences

### Positive

- **Automatic caching**: Data is cached and reused across components
- **Request deduplication**: Multiple components requesting same data = one request
- **Background refetching**: Stale data is refreshed automatically
- **Race condition handling**: Built-in request cancellation and ordering
- **Optimistic updates**: Easy to implement immediate UI feedback
- **DevTools**: Excellent debugging experience
- **Reduced boilerplate**: No manual loading/error state management

### Negative

- **Learning curve**: Team must learn React Query patterns
- **Bundle size**: Adds ~12KB to bundle (acceptable trade-off)
- **Abstraction overhead**: Simple fetches might feel over-engineered

### Neutral

- Query keys become a new concept to manage (solved with query key factory)
- Cache invalidation strategy must be defined

## Alternatives Considered

### SWR

Similar to React Query but with fewer features for mutations and cache management. React Query's mutation handling and devtools are superior.

### Apollo Client

Designed for GraphQL. We use REST/PostgREST (Supabase), so Apollo would add unnecessary complexity.

### Custom Hook

Building our own solution would replicate React Query's features poorly and require ongoing maintenance.

## References

- [TanStack Query Documentation](https://tanstack.com/query/latest)
- [Why You Want React Query](https://tkdodo.eu/blog/why-you-want-react-query)
- [ADR-0007: Query Key Factory](./0007-query-key-factory.md)
