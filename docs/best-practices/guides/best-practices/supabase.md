# Supabase Best Practices

Advanced patterns for Supabase: Client queries, RPC functions, and Edge Functions.

## Decision Framework

| Tier | Use When | Performance | Security |
|------|----------|-------------|----------|
| Client `.from()` | Simple CRUD with RLS | Fastest | RLS-enforced |
| RPC Functions | Atomic operations, aggregations | Fast (server-side) | Server validation |
| Edge Functions | External APIs, secrets, webhooks | Network-dependent | Full control |

---

## 1. Tier 1: Client Side (`.from()`)

Use for simple CRUD operations where Row Level Security is sufficient.

### When to Use

- Fetching data with straightforward filters
- User-specific data access (profiles, preferences)
- Standard insert/update/delete operations
- Real-time subscriptions

### Examples

```typescript
import { supabase } from '@/integrations/supabase/client';

// Simple select
const { data: profiles, error } = await supabase
  .from('profiles')
  .select('id, name, avatar_url')
  .eq('is_active', true);

// Insert with typed response
const { data: newProfile, error } = await supabase
  .from('profiles')
  .insert({ name: 'John', email: 'john@example.com' })
  .select()
  .single();

// Update with filters
const { error } = await supabase
  .from('profiles')
  .update({ last_seen: new Date().toISOString() })
  .eq('id', userId);
```

---

## 2. Tier 2: Database Functions (RPC)

Use for atomic operations, heavy aggregations, or bypassing complex RLS.

### When to Use

- Multiple related operations (atomic transactions)
- Complex calculations or aggregations
- Operations requiring elevated privileges
- Preventing N+1 queries

### Creating Functions

```sql
-- Simple function
CREATE OR REPLACE FUNCTION hello_world()
RETURNS TEXT
LANGUAGE SQL
AS $$
  SELECT 'hello world';
$$;

-- Function with parameters
CREATE OR REPLACE FUNCTION add_item_to_cart(
  p_user_id UUID,
  p_item_id BIGINT,
  p_quantity INT DEFAULT 1
)
RETURNS BIGINT
LANGUAGE plpgsql
SECURITY INVOKER
AS $$
DECLARE
  v_cart_item_id BIGINT;
  v_available_stock INT;
BEGIN
  -- Check stock with row lock
  SELECT stock INTO v_available_stock
  FROM products
  WHERE id = p_item_id
  FOR UPDATE;

  IF v_available_stock < p_quantity THEN
    RAISE EXCEPTION 'Insufficient stock: % available', v_available_stock;
  END IF;

  -- Deduct stock
  UPDATE products SET stock = stock - p_quantity WHERE id = p_item_id;

  -- Add to cart
  INSERT INTO cart_items (user_id, item_id, quantity)
  VALUES (p_user_id, p_item_id, p_quantity)
  RETURNING id INTO v_cart_item_id;

  RETURN v_cart_item_id;
END;
$$;
```

### Calling from Client

```typescript
// Basic RPC call
const { data, error } = await supabase.rpc('hello_world');

// RPC with parameters
const { data: cartItemId, error } = await supabase.rpc('add_item_to_cart', {
  p_user_id: userId,
  p_item_id: 123,
  p_quantity: 2,
});
```

### Security Best Practices

```sql
-- SECURITY INVOKER (default, recommended)
-- Runs with caller's permissions
CREATE FUNCTION safe_operation()
RETURNS VOID
SECURITY INVOKER
AS $$ ... $$ LANGUAGE plpgsql;

-- SECURITY DEFINER (use carefully)
-- Runs with creator's permissions - ALWAYS set search_path
CREATE FUNCTION elevated_operation()
RETURNS VOID
SECURITY DEFINER
SET search_path = ''
AS $$
BEGIN
  INSERT INTO public.audit_log (action) VALUES ('elevated');
END;
$$ LANGUAGE plpgsql;

-- Restrict function access
REVOKE EXECUTE ON FUNCTION elevated_operation FROM PUBLIC;
GRANT EXECUTE ON FUNCTION elevated_operation TO authenticated;
```

---

## 3. Tier 3: Edge Functions (Deno)

Use for third-party APIs, secrets, or webhooks.

### When to Use

- External API integrations (Stripe, SendGrid)
- Operations requiring API keys/secrets
- Webhook handlers
- AI/ML inference

### Basic Edge Function

```typescript
// supabase/functions/process-payment/index.ts
import "jsr:@supabase/functions-js/edge-runtime.d.ts";

Deno.serve(async (req: Request) => {
  try {
    const { amount, currency } = await req.json();
    const stripeKey = Deno.env.get('STRIPE_SECRET_KEY');

    const response = await fetch('https://api.stripe.com/v1/charges', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${stripeKey}`,
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: new URLSearchParams({ amount: amount.toString(), currency }),
    });

    return new Response(JSON.stringify(await response.json()), {
      headers: { 'Content-Type': 'application/json' },
    });
  } catch (error) {
    return new Response(
      JSON.stringify({ error: error.message }),
      { status: 500 }
    );
  }
});
```

### Invoking from Client

```typescript
const { data, error } = await supabase.functions.invoke('process-payment', {
  body: { amount: 1000, currency: 'USD' },
});

// Error handling
import { FunctionsHttpError } from '@supabase/supabase-js';

if (error instanceof FunctionsHttpError) {
  const errorMessage = await error.context.json();
  console.log('Function error:', errorMessage);
}
```

---

## 4. TypeScript Integration

### Generating Types

```bash
# From local database
npx supabase gen types typescript --local > src/integrations/supabase/types.ts

# From remote database
npx supabase gen types typescript --project-id "$PROJECT_REF" > src/integrations/supabase/types.ts
```

### Using Generated Types

```typescript
import { createClient } from '@supabase/supabase-js';
import { Database, Tables } from '@/integrations/supabase/types';

const supabase = createClient<Database>(url, key);

type Profile = Tables<'profiles'>;

const { data } = await supabase
  .from('profiles')
  .select('*')
  .single();
// data is typed as Profile | null
```

---

## 5. Row Level Security (RLS)

### Essential Patterns

```sql
-- Enable RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- User can only access their own data
CREATE POLICY "Users can view own profile"
  ON profiles FOR SELECT
  USING (auth.uid() = user_id);

-- Public read, authenticated write
CREATE POLICY "Public profiles viewable"
  ON profiles FOR SELECT
  USING (is_public = true);

CREATE POLICY "Authenticated users can insert"
  ON profiles FOR INSERT
  TO authenticated
  WITH CHECK (auth.uid() = user_id);

-- Role-based access
CREATE POLICY "Admins have full access"
  ON profiles FOR ALL
  USING (
    EXISTS (
      SELECT 1 FROM user_roles
      WHERE user_roles.user_id = auth.uid()
      AND user_roles.role = 'admin'
    )
  );
```

### Verification

```bash
npx supabase db lint
```

---

## 6. Real-time Subscriptions

```typescript
const channel = supabase
  .channel('profiles-changes')
  .on(
    'postgres_changes',
    {
      event: '*',
      schema: 'public',
      table: 'profiles',
      filter: `id=eq.${userId}`,
    },
    (payload) => {
      console.log('Change received:', payload);
    }
  )
  .subscribe();

// Cleanup
channel.unsubscribe();
```

---

## Quick Reference: When to Use What

| Scenario | Recommended Tier |
|----------|-----------------|
| Fetch user's own data | Client `.from()` |
| Update user profile | Client `.from()` |
| Transfer money between accounts | RPC (atomic transaction) |
| Calculate complex statistics | RPC (server-side aggregation) |
| Process Stripe webhook | Edge Function |
| Send email via SendGrid | Edge Function |
| Real-time chat messages | Client with subscriptions |
| Audit logging | RPC with `SECURITY DEFINER` |
