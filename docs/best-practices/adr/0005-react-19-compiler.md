# ADR-0005: React 19 with Compiler

## Status

**Accepted**

## Context

React applications traditionally require manual optimization:

```typescript
// React 18 and earlier - manual memoization
const MemoizedComponent = React.memo(({ data }) => {
  const processed = useMemo(() => expensiveCalc(data), [data]);
  const handleClick = useCallback(() => doSomething(data), [data]);
  return <div onClick={handleClick}>{processed}</div>;
});
```

This has problems:
- **Cognitive load**: Developers must decide what to memoize
- **Over-memoization**: Often adds more overhead than it saves
- **Under-memoization**: Missing optimizations cause performance issues
- **Maintenance**: Dependency arrays must be kept in sync

React 19 introduces the **React Compiler** (formerly React Forget) which automatically handles memoization.

## Decision

Adopt React 19 with the React Compiler enabled. Remove manual memoization unless profiler indicates specific need.

```typescript
// React 19 with Compiler - no manual optimization needed
function Component({ data }) {
  const processed = expensiveCalc(data);
  const handleClick = () => doSomething(data);
  return <div onClick={handleClick}>{processed}</div>;
}
```

## Consequences

### Positive

- **Simpler code**: No useMemo/useCallback/memo boilerplate
- **Fewer bugs**: No stale closure issues from wrong dependencies
- **Better performance**: Compiler optimizes more consistently than humans
- **Easier reviews**: Less to check in code reviews

### Negative

- **New technology**: Compiler is relatively new
- **Build complexity**: Requires Babel plugin
- **Debugging**: Compiled output differs from source
- **Rules enforcement**: Must follow Rules of React strictly

### Neutral

- Some edge cases may still need manual optimization
- Third-party libraries may require stable references

## Compiler Rules

The compiler expects code that follows React's rules:

```typescript
// DO: Pure render logic
function GoodComponent({ items }) {
  const sorted = items.toSorted((a, b) => a.localeCompare(b));
  return <List items={sorted} />;
}

// DON'T: Side effects during render
function BadComponent({ items }) {
  items.sort((a, b) => a.localeCompare(b)); // Mutates prop!
  trackAnalytics(); // Side effect during render!
  return <List items={items} />;
}
```

## When Manual Optimization is Still Needed

1. **Profiler shows specific bottleneck**
2. **Third-party library requires stable references**
3. **Complex dependency chains compiler cannot analyze**

## Configuration

```typescript
// vite.config.ts
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [
    react({
      babel: {
        plugins: [['babel-plugin-react-compiler']],
      },
    }),
  ],
});
```

## Amendment (2026-02-12)

The React Compiler is **not currently enabled**. The project uses **SWC** (via `@vitejs/plugin-react-swc`) instead of the Babel-based compiler plugin described above.

Current state:
- `babel-plugin-react-compiler` is not installed
- Manual memoizations (`useMemo`, `useCallback`, `React.memo`) are intentionally kept
- The codebase follows Rules of React in anticipation of future compiler adoption

The compiler may be enabled in a future iteration once the ecosystem stabilizes. Until then, manual memoization remains the optimization strategy.

## References

- [React 19 Announcement](https://react.dev/blog/2024/04/25/react-19)
- [React Compiler Documentation](https://react.dev/learn/react-compiler)
- [Rules of React](https://react.dev/reference/rules)
