# Schema Patterns

Zod validation schema patterns.

## Basic Schema

```typescript
import { z } from 'zod';

// Entity schema (from database)
export const profileSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(100),
  email: z.string().email(),
  bio: z.string().max(500).optional(),
  is_active: z.boolean().default(true),
  created_at: z.string().datetime(),
  updated_at: z.string().datetime(),
});

// Infer type from schema
export type Profile = z.infer<typeof profileSchema>;
```

---

## Create/Update Schemas

```typescript
// For create operations (no id, timestamps)
export const createProfileSchema = profileSchema.omit({
  id: true,
  created_at: true,
  updated_at: true,
});

export type CreateProfileInput = z.infer<typeof createProfileSchema>;

// For update operations (all fields optional except id)
export const updateProfileSchema = createProfileSchema.partial();

export type UpdateProfileInput = z.infer<typeof updateProfileSchema>;
```

---

## Enum Schema

```typescript
export const roleSchema = z.enum(['client', 'provider', 'admin']);
export type Role = z.infer<typeof roleSchema>;

// With descriptions
export const statusSchema = z.enum(['pending', 'approved', 'rejected']);
export type Status = z.infer<typeof statusSchema>;
```

---

## Nested Schema

```typescript
const photoSchema = z.object({
  id: z.string().uuid(),
  url: z.string().url(),
  order: z.number().int().min(0),
});

const serviceSchema = z.object({
  id: z.string().uuid(),
  name: z.string(),
  price: z.number().positive(),
});

export const providerProfileSchema = z.object({
  id: z.string().uuid(),
  name: z.string(),
  photos: z.array(photoSchema),
  services: z.array(serviceSchema),
});

export type ProviderProfile = z.infer<typeof providerProfileSchema>;
```

---

## Validation with Custom Messages

```typescript
export const loginSchema = z.object({
  email: z.string({ required_error: 'Email is required' }).email('Invalid email address'),
  password: z
    .string({ required_error: 'Password is required' })
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain an uppercase letter')
    .regex(/[0-9]/, 'Password must contain a number'),
});
```

---

## Refinements

```typescript
// Cross-field validation
export const priceRangeSchema = z
  .object({
    minPrice: z.number().positive().optional(),
    maxPrice: z.number().positive().optional(),
  })
  .refine(
    (data) => !data.minPrice || !data.maxPrice || data.minPrice <= data.maxPrice,
    { message: 'Minimum price must be less than maximum', path: ['minPrice'] }
  );

// Transform during validation
export const dateSchema = z
  .string()
  .datetime()
  .transform((str) => new Date(str));
```

---

## Union Types

```typescript
// Discriminated union
const successResponseSchema = z.object({
  ok: z.literal(true),
  data: z.unknown(),
});

const errorResponseSchema = z.object({
  ok: z.literal(false),
  error: z.string(),
});

export const apiResponseSchema = z.discriminatedUnion('ok', [
  successResponseSchema,
  errorResponseSchema,
]);
```

---

## Form Validation

```typescript
// In component
const schema = z.object({
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Invalid email'),
});

type FormData = z.infer<typeof schema>;

function ContactForm() {
  const [errors, setErrors] = useState<Record<string, string>>({});

  const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    const data = Object.fromEntries(formData);

    const result = schema.safeParse(data);

    if (!result.success) {
      const fieldErrors: Record<string, string> = {};
      result.error.issues.forEach((issue) => {
        const path = issue.path[0] as string;
        fieldErrors[path] = issue.message;
      });
      setErrors(fieldErrors);
      return;
    }

    // Valid data
    onSubmit(result.data);
  };
}
```

---

## API Response Validation

```typescript
async function fetchProfile(id: string): Promise<Profile> {
  const response = await fetch(`/api/profiles/${id}`);
  const data = await response.json();

  // Validate response matches expected shape
  return profileSchema.parse(data);
}

// Safe version
async function safeFetchProfile(id: string): Promise<Profile | null> {
  const response = await fetch(`/api/profiles/${id}`);
  const data = await response.json();

  const result = profileSchema.safeParse(data);
  return result.success ? result.data : null;
}
```
