# ADR-0004: Light Mapper Services

## Status

**Accepted**

## Context

The service layer sits between React Query hooks and Supabase. A common pattern is to use heavy mappers that transform database entities to domain models:

```typescript
// Heavy mapper approach
class UserMapper {
  toDomain(row: UserRow): User {
    return new User({
      id: new UserId(row.id),
      email: new Email(row.email),
      name: new Name(row.first_name, row.last_name),
      // ... many transformations
    });
  }
}
```

This adds complexity:
- Domain classes with validation
- Mapper classes for each entity
- Multiple representations of the same data

## Decision

Use **light mapper services** that perform minimal transformation. Supabase returns typed data that can be used directly in most cases.

```typescript
// Light mapper approach
export const userService = {
  async getById(id: string): Promise<User> {
    const { data, error } = await supabase
      .from('users')
      .select('*')
      .eq('id', id)
      .single();

    if (error) throw error;

    // Light transformation only when needed
    return {
      ...data,
      fullName: `${data.first_name} ${data.last_name}`,
    };
  },
};
```

## Consequences

### Positive

- **Simplicity**: Less code to write and maintain
- **Performance**: No unnecessary object creation
- **Type safety**: Supabase types flow through
- **Debugging**: Data matches what's in the database

### Negative

- **Database coupling**: UI is somewhat coupled to DB schema
- **Validation deferred**: Validation happens at boundaries, not in domain
- **Less domain modeling**: No rich domain objects

### Neutral

- Computed properties added inline as needed
- Zod schemas validate at API boundaries

## When to Add Heavier Mapping

- Complex business logic requiring domain methods
- Aggregating data from multiple tables
- Significant shape difference between DB and UI needs

## Example: When Light is Right

```typescript
// Database returns exactly what UI needs
const { data: profile } = await supabase
  .from('profiles')
  .select('id, display_name, avatar_url, bio')
  .eq('id', id)
  .single();

// Use directly in component
<ProfileCard profile={profile} />
```

## Example: When Mapping is Needed

```typescript
// Need to combine/compute
const { data: profile } = await supabase
  .from('profiles')
  .select(`
    *,
    photos:profile_photos(url, order),
    services:profile_services(name, price)
  `)
  .eq('id', id)
  .single();

// Map to cleaner shape
return {
  ...profile,
  primaryPhoto: profile.photos?.find(p => p.order === 0)?.url,
  serviceCount: profile.services?.length ?? 0,
};
```

## References

- Service layer files live in `src/services/`
- [ADR-0003: Result Pattern](./0003-result-pattern-errors.md)
