# MSW Handlers (Network Mocking)

MSW (Mock Service Worker) intercepts network requests for testing.

## Setup

### Server Configuration

```typescript
// src/test/msw/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

### Test Setup Integration

```typescript
// src/test/setup.ts
import { beforeAll, afterEach, afterAll } from 'vitest';
import { server } from './msw/server';

beforeAll(() => server.listen({ onUnhandledRequest: 'warn' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

---

## MSW v2 Handler Syntax

### Basic GET Handler

```typescript
// src/test/msw/handlers.ts
import { http, HttpResponse } from 'msw';

const TEST_SUPABASE_URL = 'http://localhost:54321';

export const handlers = [
  // List endpoint
  http.get(`${TEST_SUPABASE_URL}/rest/v1/profiles`, () => {
    return HttpResponse.json([
      { id: '1', name: 'John Doe', email: 'john@example.com' },
      { id: '2', name: 'Jane Smith', email: 'jane@example.com' },
    ]);
  }),

  // Single item endpoint (with query params)
  http.get(`${TEST_SUPABASE_URL}/rest/v1/profiles`, ({ request }) => {
    const url = new URL(request.url);
    const idFilter = url.searchParams.get('id');

    // Handle single item query (id=eq.123)
    if (idFilter?.startsWith('eq.')) {
      const id = idFilter.replace('eq.', '');
      return HttpResponse.json({
        id,
        name: 'John Doe',
        email: 'john@example.com',
      });
    }

    // Handle list query
    return HttpResponse.json([]);
  }),
];
```

### POST Handler

```typescript
http.post(`${TEST_SUPABASE_URL}/rest/v1/profiles`, async ({ request }) => {
  const body = await request.json();

  // Validate input
  if (!body.name || !body.email) {
    return HttpResponse.json(
      { message: 'Name and email are required' },
      { status: 400 }
    );
  }

  // Return created resource
  return HttpResponse.json(
    { id: 'new-id', ...body, created_at: new Date().toISOString() },
    { status: 201 }
  );
}),
```

### PATCH Handler

```typescript
http.patch(`${TEST_SUPABASE_URL}/rest/v1/profiles`, async ({ request }) => {
  const url = new URL(request.url);
  const idFilter = url.searchParams.get('id');
  const body = await request.json();

  if (!idFilter) {
    return HttpResponse.json(
      { message: 'ID filter required' },
      { status: 400 }
    );
  }

  const id = idFilter.replace('eq.', '');

  return HttpResponse.json({
    id,
    ...body,
    updated_at: new Date().toISOString(),
  });
}),
```

### DELETE Handler

```typescript
http.delete(`${TEST_SUPABASE_URL}/rest/v1/profiles`, ({ request }) => {
  const url = new URL(request.url);
  const idFilter = url.searchParams.get('id');

  if (!idFilter) {
    return HttpResponse.json(
      { message: 'ID filter required' },
      { status: 400 }
    );
  }

  // Return empty for successful delete
  return new HttpResponse(null, { status: 204 });
}),
```

---

## Overriding Handlers in Tests

Override default handlers for specific test scenarios:

```typescript
import { http, HttpResponse } from 'msw';
import { server } from '@/test/msw/server';

describe('ProfileList', () => {
  it('shows empty state', async () => {
    // Override for this test only
    server.use(
      http.get('*/rest/v1/profiles*', () => {
        return HttpResponse.json([]);
      })
    );

    render(<ProfileList />);
    await screen.findByText('No profiles found');
  });

  it('shows error state', async () => {
    server.use(
      http.get('*/rest/v1/profiles*', () => {
        return new HttpResponse(null, { status: 500 });
      })
    );

    render(<ProfileList />);
    await screen.findByRole('alert');
  });

  it('shows network error', async () => {
    server.use(
      http.get('*/rest/v1/profiles*', () => {
        return HttpResponse.error();
      })
    );

    render(<ProfileList />);
    await screen.findByText(/network error/i);
  });
});
```

---

## Response Utilities

### JSON Response

```typescript
// Simple JSON
HttpResponse.json({ data: 'value' });

// With status code
HttpResponse.json({ error: 'Not found' }, { status: 404 });

// With headers
HttpResponse.json(data, {
  status: 200,
  headers: {
    'X-Total-Count': '100',
  },
});
```

### Error Responses

```typescript
// HTTP error (4xx, 5xx)
HttpResponse.json(
  { message: 'Unauthorized' },
  { status: 401 }
);

// Network error (connection failed)
HttpResponse.error();
```

### Delayed Response

```typescript
import { delay } from 'msw';

http.get('/api/data', async () => {
  await delay(1000); // 1 second delay
  return HttpResponse.json({ data: 'value' });
});

// Or use 'infinite' for testing loading states
http.get('/api/data', async () => {
  await delay('infinite');
  return HttpResponse.json({});
});
```

---

## Request Inspection

### URL Parameters

```typescript
http.get('/api/users/:id', ({ params }) => {
  const { id } = params;
  return HttpResponse.json({ id, name: `User ${id}` });
});
```

### Query Parameters

```typescript
http.get('/api/search', ({ request }) => {
  const url = new URL(request.url);
  const query = url.searchParams.get('q');
  const page = url.searchParams.get('page') || '1';

  return HttpResponse.json({
    results: [`Result for: ${query}`],
    page: parseInt(page),
  });
});
```

### Request Body

```typescript
http.post('/api/users', async ({ request }) => {
  const body = await request.json();
  console.log('Received:', body);

  return HttpResponse.json({ id: '123', ...body });
});
```

### Request Headers

```typescript
http.get('/api/protected', ({ request }) => {
  const authHeader = request.headers.get('Authorization');

  if (!authHeader?.startsWith('Bearer ')) {
    return HttpResponse.json(
      { message: 'Unauthorized' },
      { status: 401 }
    );
  }

  return HttpResponse.json({ data: 'protected data' });
});
```

---

## Handler Organization

### By Domain

```typescript
// src/test/msw/handlers/profile.handlers.ts
export const profileHandlers = [
  http.get('*/rest/v1/profiles*', () => { /* ... */ }),
  http.post('*/rest/v1/profiles*', () => { /* ... */ }),
];

// src/test/msw/handlers/auth.handlers.ts
export const authHandlers = [
  http.post('*/auth/v1/token', () => { /* ... */ }),
];

// src/test/msw/handlers.ts
import { profileHandlers } from './handlers/profile.handlers';
import { authHandlers } from './handlers/auth.handlers';

export const handlers = [
  ...profileHandlers,
  ...authHandlers,
];
```

### With Test Data Builders

```typescript
import { aProfile } from '@/test/builders/profile-builder';

http.get('*/rest/v1/profiles*', () => {
  return HttpResponse.json([
    aProfile().withName('John').build(),
    aProfile().withName('Jane').isPremium().build(),
  ]);
});
```

---

## Debugging

### Log Requests

```typescript
http.get('*/api/*', ({ request }) => {
  console.log('Request:', request.method, request.url);
  console.log('Headers:', Object.fromEntries(request.headers));

  return HttpResponse.json({});
});
```

### Unhandled Requests

Configure server to warn about unhandled requests:

```typescript
server.listen({ onUnhandledRequest: 'warn' });
// or 'error' to fail tests
// or 'bypass' to allow through
```
