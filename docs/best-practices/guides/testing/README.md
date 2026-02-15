# Testing Guide

This section covers the testing strategy and patterns used in the codebase.

## Contents

| Guide | Description |
|-------|-------------|
| [Unit Testing](./unit-testing.md) | Vitest patterns and utilities |
| [Hook Testing](./hook-testing.md) | Testing React Query hooks |
| [Component Testing](./component-testing.md) | React Testing Library patterns |
| [MSW Handlers](./msw-handlers.md) | Network mocking with MSW v2 |

---

## Testing Stack

| Tool | Purpose |
|------|---------|
| Vitest | Test runner |
| React Testing Library | Component/hook testing |
| MSW v2 | Network mocking |
| @testing-library/user-event | User interaction simulation |

---

## Quick Commands

```bash
pnpm test          # Watch mode
pnpm test:run      # Single run
pnpm test:coverage # With coverage report
```

---

## Test File Location

Tests are co-located with source files in `__tests__/` folders:

```
src/
├── hooks/
│   └── api/
│       ├── useProfiles.ts
│       └── __tests__/
│           └── useProfiles.test.ts
├── components/
│   └── ProfileCard.tsx
│   └── __tests__/
│       └── ProfileCard.test.tsx
```

---

## Testing Philosophy

### What to Test

| Type | Test For |
|------|----------|
| Hooks | Data fetching, state changes, side effects |
| Components | Rendering, user interactions, accessibility |
| Services | Business logic, data transformation |
| Utils | Pure function behavior |

### What NOT to Test

- Implementation details (internal state, private methods)
- Third-party library behavior
- TypeScript types (compiler checks these)
- Styling/CSS (unless functionally relevant)

---

## Test Setup

The global setup is in `src/test/setup.ts`:

```typescript
import { afterAll, afterEach, beforeAll } from 'vitest';
import { cleanup } from '@testing-library/react';
import { server } from './msw/server';

// MSW server lifecycle
beforeAll(() => server.listen({ onUnhandledRequest: 'warn' }));
afterEach(() => {
  server.resetHandlers();
  cleanup();
});
afterAll(() => server.close());
```

---

## Common Test Patterns

### Testing Loading States

```typescript
it('shows loading state', () => {
  render(<ProfileList />);
  expect(screen.getByTestId('skeleton')).toBeInTheDocument();
});
```

### Testing Success States

```typescript
it('displays data when loaded', async () => {
  render(<ProfileList />, { wrapper: createWrapper() });

  await waitFor(() => {
    expect(screen.getByText('John Doe')).toBeInTheDocument();
  });
});
```

### Testing Error States

```typescript
it('shows error message on failure', async () => {
  server.use(
    http.get('*/rest/v1/profiles*', () => HttpResponse.error())
  );

  render(<ProfileList />, { wrapper: createWrapper() });

  await waitFor(() => {
    expect(screen.getByRole('alert')).toBeInTheDocument();
  });
});
```

### Testing Empty States

```typescript
it('shows empty state when no data', async () => {
  server.use(
    http.get('*/rest/v1/profiles*', () => HttpResponse.json([]))
  );

  render(<ProfileList />, { wrapper: createWrapper() });

  await waitFor(() => {
    expect(screen.getByText('No profiles found')).toBeInTheDocument();
  });
});
```

---

## Test Data Builders

Use builders for consistent test data:

```typescript
// src/test/builders/profile-builder.ts
export const aProfile = () => new ProfileBuilder();

class ProfileBuilder {
  private profile: Partial<Profile> = {
    id: 'test-id',
    name: 'Test User',
    email: 'test@example.com',
  };

  withName(name: string) {
    this.profile.name = name;
    return this;
  }

  isPremium() {
    this.profile.isPremium = true;
    return this;
  }

  build(): Profile {
    return this.profile as Profile;
  }
}

// Usage
const profile = aProfile().withName('Jane').isPremium().build();
```

---

## Running Specific Tests

```bash
# Run tests in specific file
pnpm test useProfiles.test.ts

# Run tests matching pattern
pnpm test -t "should fetch"

# Run with coverage
pnpm test:coverage
```
