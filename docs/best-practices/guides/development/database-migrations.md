# Database Migrations

This guide covers managing database schema with Supabase CLI migrations.

## Overview

We use Supabase CLI for all database schema management. The local database is the source of truth for development.

---

## CLI Quick Reference

| Command | Description |
|---------|-------------|
| `supabase start` | Start local Supabase |
| `supabase stop` | Stop local Supabase |
| `supabase status` | Show local service status |
| `supabase db reset` | Reset DB, run all migrations + seed |
| `supabase db diff -f <name>` | Generate migration from changes |
| `supabase migration new <name>` | Create empty migration file |
| `supabase migration list` | List applied migrations |
| `supabase db push` | Push migrations to remote |
| `supabase db pull` | Pull remote schema as migration |
| `supabase db lint` | Check for issues |
| `supabase gen types typescript --local` | Generate TypeScript types |

---

## Local Development Flow

### 1. Start Local Database

```bash
npx supabase start
```

Check status and connection details:

```bash
npx supabase status
```

### 2. Create Migration

**Option A: Manual Migration (Recommended)**

```bash
npx supabase migration new create_users_table
```

Edit the created file:

```sql
-- supabase/migrations/YYYYMMDDHHMMSS_create_users_table.sql

CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Always enable RLS
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Add policies
CREATE POLICY "Users can view own profile"
  ON users FOR SELECT
  USING (auth.uid() = id);
```

**Option B: Schema Diff (for Dashboard changes)**

1. Make changes via local Studio at `http://localhost:54323`
2. Generate migration:

```bash
npx supabase db diff -f add_avatar_column
```

### 3. Apply Migration

```bash
npx supabase db reset
```

This:
- Drops local database
- Re-runs all migrations
- Applies `seed.sql`

### 4. Generate Types

```bash
npx supabase gen types typescript --local > src/integrations/supabase/types.ts
```

---

## Migration Best Practices

### File Naming

Migrations apply in timestamp order:

```
20240101000000_create_users_table.sql
20240101000001_add_profiles_table.sql
20240102000000_add_user_avatar.sql
```

### Idempotent Migrations

Write migrations that can be re-run safely:

```sql
-- Use IF NOT EXISTS / IF EXISTS
CREATE TABLE IF NOT EXISTS users (...);
ALTER TABLE users ADD COLUMN IF NOT EXISTS avatar TEXT;
DROP INDEX IF EXISTS idx_users_email;

-- Use CREATE OR REPLACE for functions
CREATE OR REPLACE FUNCTION get_user_count()
RETURNS BIGINT AS $$
  SELECT COUNT(*) FROM users;
$$ LANGUAGE SQL STABLE;
```

### Avoiding Destructive Changes

```sql
-- DANGEROUS: Avoid in production
DROP TABLE users;
ALTER TABLE users DROP COLUMN email;
TRUNCATE users;

-- SAFER: Rename instead of drop (allows rollback)
ALTER TABLE users RENAME TO users_deprecated;
ALTER TABLE users RENAME COLUMN email TO email_old;
```

---

## Row Level Security (RLS)

### Enable RLS

```sql
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;
```

### Common Patterns

```sql
-- User owns row
CREATE POLICY "Users can CRUD own posts"
  ON posts FOR ALL
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

-- Public read
CREATE POLICY "Anyone can read published posts"
  ON posts FOR SELECT
  USING (status = 'published');

-- Authenticated users only
CREATE POLICY "Authenticated can insert"
  ON posts FOR INSERT
  TO authenticated
  WITH CHECK (auth.uid() = user_id);

-- Role-based access
CREATE POLICY "Admins have full access"
  ON posts FOR ALL
  USING (
    EXISTS (
      SELECT 1 FROM user_roles
      WHERE user_id = auth.uid()
      AND role = 'admin'
    )
  );
```

### Verify RLS

```bash
npx supabase db lint
```

Checks for missing RLS policies.

---

## Seed Data

### Location

`supabase/seed.sql` - Runs after migrations during `db reset`

### Idempotent Seeds

```sql
-- Use ON CONFLICT for idempotent seeding
INSERT INTO departments (id, name)
VALUES
  ('dept-1', 'Engineering'),
  ('dept-2', 'Marketing')
ON CONFLICT (id) DO UPDATE SET
  name = EXCLUDED.name;

-- Or delete + insert for complete refresh
DELETE FROM test_users WHERE email LIKE '%@test.com';
INSERT INTO test_users (name, email)
VALUES
  ('Test User 1', 'test1@test.com'),
  ('Test User 2', 'test2@test.com');
```

---

## Remote Database

### Link to Project

```bash
npx supabase login
npx supabase link --project-ref <project-id>
```

### Push Migrations

```bash
# Preview first
npx supabase db push --dry-run

# Apply
npx supabase db push
```

### Pull Remote Changes

```bash
npx supabase db pull
```

### Generate Types from Remote

```bash
npx supabase gen types typescript --project-id "$PROJECT_REF" > src/integrations/supabase/types.ts
```

---

## Troubleshooting

### Schema Drift

**Symptom**: `db diff` shows unexpected changes

**Solution**:
1. Don't sync remote to local blindly
2. Communicate about direct remote changes
3. Run `npx supabase db reset` to verify

### Migration Conflicts

**Symptom**: Two developers created migrations simultaneously

**Solution**:
1. Rename timestamp prefix to ensure correct order
2. Ensure dependent migrations have later timestamps
3. Test with `npx supabase db reset`

### Permission Denied

**Symptom**: `permission denied for table` errors

**Solution**:

```sql
-- Run in Supabase Dashboard SQL Editor
GRANT ALL ON ALL TABLES IN SCHEMA public TO postgres, anon, authenticated;
GRANT ALL ON ALL FUNCTIONS IN SCHEMA public TO postgres, anon, authenticated;
GRANT ALL ON ALL SEQUENCES IN SCHEMA public TO postgres, anon, authenticated;
```

### Table Ownership

**Symptom**: `must be owner of table` error

**Solution**:

```sql
ALTER TABLE your_table OWNER TO postgres;
```

---

## Daily Workflow

```bash
# Morning: Start fresh
npx supabase db reset

# Make changes via Studio or SQL
# ...

# Generate migration
npx supabase db diff -f my_changes

# Verify
npx supabase db reset
npx supabase db lint

# Generate types
npx supabase gen types typescript --local > src/integrations/supabase/types.ts

# Commit
git add supabase/migrations/ src/integrations/supabase/types.ts
git commit -m "Add my_changes migration"
```

---

## Before PR Merge

```bash
# Verify all migrations apply
npx supabase db reset

# Check for issues
npx supabase db lint

# Ensure types are current
npx supabase gen types typescript --local > src/integrations/supabase/types.ts
```

---

## Checklist

- [ ] Migration file created with descriptive name
- [ ] RLS enabled on new tables
- [ ] Appropriate indexes added
- [ ] No destructive DROP statements
- [ ] `supabase db reset` succeeds
- [ ] `supabase db lint` passes
- [ ] Types regenerated
- [ ] Tests pass with new schema
