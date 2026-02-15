# Adding an Edge Function

This guide walks through creating and deploying Supabase Edge Functions.

## Overview

Edge Functions are serverless functions that run on Supabase's edge network. Use them for:
- External API integrations (Stripe, SendGrid, etc.)
- Operations requiring secrets
- Webhook handlers
- Custom authentication flows

---

## Step 1: Create Function

```bash
# Create new function directory
npx supabase functions new my-function
```

This creates:
```
supabase/functions/my-function/
└── index.ts
```

## Step 2: Implement Function

```typescript
// supabase/functions/my-function/index.ts
import "jsr:@supabase/functions-js/edge-runtime.d.ts";

Deno.serve(async (req: Request) => {
  // Handle CORS preflight
  if (req.method === 'OPTIONS') {
    return new Response(null, {
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'POST, OPTIONS',
        'Access-Control-Allow-Headers': 'Authorization, Content-Type',
      },
    });
  }

  try {
    // Parse request body
    const { name, email } = await req.json();

    // Validate input
    if (!name || !email) {
      return new Response(
        JSON.stringify({ error: 'Name and email are required' }),
        { status: 400, headers: { 'Content-Type': 'application/json' } }
      );
    }

    // Your logic here
    const result = await processData(name, email);

    return new Response(
      JSON.stringify({ success: true, data: result }),
      {
        status: 200,
        headers: {
          'Content-Type': 'application/json',
          'Access-Control-Allow-Origin': '*',
        },
      }
    );
  } catch (error) {
    console.error('Function error:', error);
    return new Response(
      JSON.stringify({ error: error.message }),
      {
        status: 500,
        headers: { 'Content-Type': 'application/json' },
      }
    );
  }
});

async function processData(name: string, email: string) {
  // Implementation
  return { processed: true };
}
```

## Step 3: Add Environment Variables

For local development, create `.env.local`:

```bash
# supabase/.env.local
MY_API_KEY=your-api-key
EXTERNAL_SERVICE_URL=https://api.example.com
```

For production, set secrets:

```bash
npx supabase secrets set MY_API_KEY=your-api-key
npx supabase secrets list
```

## Step 4: Use Supabase Client in Function

```typescript
import { createClient } from 'npm:@supabase/supabase-js@2';

Deno.serve(async (req: Request) => {
  // For admin operations (bypasses RLS)
  const supabaseAdmin = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  );

  // For user operations (respects RLS)
  const authHeader = req.headers.get('Authorization');
  const supabaseClient = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_ANON_KEY')!,
    { global: { headers: { Authorization: authHeader! } } }
  );

  // Get authenticated user
  const { data: { user }, error: authError } = await supabaseClient.auth.getUser();
  if (authError || !user) {
    return new Response(
      JSON.stringify({ error: 'Unauthorized' }),
      { status: 401 }
    );
  }

  // Use client for RLS-protected queries
  const { data, error } = await supabaseClient
    .from('profiles')
    .select('*')
    .eq('id', user.id)
    .single();

  // ...
});
```

## Step 5: Local Development

```bash
# Start local Supabase (if not running)
npx supabase start

# Serve function locally
npx supabase functions serve my-function --env-file ./supabase/.env.local
```

Test locally:

```bash
curl -i --location --request POST 'http://localhost:54321/functions/v1/my-function' \
  --header 'Authorization: Bearer YOUR_ANON_KEY' \
  --header 'Content-Type: application/json' \
  --data '{"name":"Test","email":"test@example.com"}'
```

## Step 6: Deploy

```bash
# Deploy single function
npx supabase functions deploy my-function

# Deploy all functions
npx supabase functions deploy
```

---

## Calling from Client

```typescript
// Using supabase-js
const { data, error } = await supabase.functions.invoke('my-function', {
  body: { name: 'John', email: 'john@example.com' },
});

if (error) {
  console.error('Function error:', error);
  return;
}

console.log('Result:', data);
```

### Error Handling

```typescript
import {
  FunctionsHttpError,
  FunctionsRelayError,
  FunctionsFetchError,
} from '@supabase/supabase-js';

const { data, error } = await supabase.functions.invoke('my-function', {
  body: { name: 'John' },
});

if (error instanceof FunctionsHttpError) {
  // Function returned an error response
  const errorMessage = await error.context.json();
  console.log('Function error:', errorMessage);
} else if (error instanceof FunctionsRelayError) {
  // Relay error (network/infrastructure)
  console.log('Relay error:', error.message);
} else if (error instanceof FunctionsFetchError) {
  // Fetch failed
  console.log('Fetch error:', error.message);
}
```

---

## Common Patterns

### Webhook Handler

```typescript
Deno.serve(async (req: Request) => {
  // Verify webhook signature
  const signature = req.headers.get('x-webhook-signature');
  const body = await req.text();

  if (!verifySignature(body, signature)) {
    return new Response('Invalid signature', { status: 401 });
  }

  const payload = JSON.parse(body);

  // Process webhook
  await handleWebhook(payload);

  return new Response('OK', { status: 200 });
});
```

### External API Integration

```typescript
Deno.serve(async (req: Request) => {
  const apiKey = Deno.env.get('EXTERNAL_API_KEY');

  const response = await fetch('https://api.external.com/endpoint', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(await req.json()),
  });

  const data = await response.json();

  return new Response(JSON.stringify(data), {
    headers: { 'Content-Type': 'application/json' },
  });
});
```

### Admin Operation

```typescript
Deno.serve(async (req: Request) => {
  // Verify admin role
  const supabaseClient = createClient(/* user client */);
  const { data: { user } } = await supabaseClient.auth.getUser();

  const { data: roles } = await supabaseClient
    .from('user_roles')
    .select('role')
    .eq('user_id', user.id)
    .single();

  if (roles?.role !== 'admin') {
    return new Response('Forbidden', { status: 403 });
  }

  // Use admin client for operation
  const supabaseAdmin = createClient(/* admin client */);
  // ... admin operation
});
```

---

## Debugging

### View Logs

```bash
# Local logs appear in terminal

# Production logs
npx supabase functions logs my-function

# Or in Dashboard: Functions → my-function → Logs
```

### Common Issues

**CORS errors**:
- Add CORS headers to all responses
- Handle OPTIONS preflight requests

**Environment variables not found**:
- Check secrets are set: `npx supabase secrets list`
- Use `Deno.env.get()` not `process.env`

**Auth issues**:
- Pass Authorization header when invoking
- Use correct client (admin vs user)

---

## Checklist

- [ ] Function created with `supabase functions new`
- [ ] Input validation added
- [ ] Error handling implemented
- [ ] CORS headers configured
- [ ] Environment variables/secrets set
- [ ] Tested locally
- [ ] Deployed to production
- [ ] Client invocation working
