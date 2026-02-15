# Unit Testing with Vitest

Patterns and utilities for unit testing with Vitest.

## Basic Test Structure

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

describe('myFunction', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should do something', () => {
    const result = myFunction('input');
    expect(result).toBe('expected');
  });

  it('should handle edge case', () => {
    expect(() => myFunction(null)).toThrow('Invalid input');
  });
});
```

---

## Assertions

### Common Matchers

```typescript
// Equality
expect(value).toBe(expected);         // Strict equality (===)
expect(value).toEqual(expected);      // Deep equality
expect(value).toStrictEqual(expected); // Deep + type equality

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();
expect(value).toBeDefined();

// Numbers
expect(num).toBeGreaterThan(3);
expect(num).toBeLessThan(10);
expect(num).toBeCloseTo(0.3, 5);

// Strings
expect(str).toMatch(/pattern/);
expect(str).toContain('substring');

// Arrays
expect(arr).toContain(item);
expect(arr).toHaveLength(3);

// Objects
expect(obj).toHaveProperty('key');
expect(obj).toMatchObject({ key: 'value' });

// Errors
expect(() => fn()).toThrow();
expect(() => fn()).toThrow('message');
expect(() => fn()).toThrowError(CustomError);
```

### Async Assertions

```typescript
// Promises
await expect(promise).resolves.toBe('value');
await expect(promise).rejects.toThrow('error');

// Async functions
it('handles async', async () => {
  const result = await asyncFunction();
  expect(result).toBe('value');
});
```

---

## Mocking

### Mock Functions

```typescript
// Create mock
const mockFn = vi.fn();
mockFn('arg1', 'arg2');

// Assertions
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
expect(mockFn).toHaveBeenCalledTimes(1);

// Return values
const mockFn = vi.fn()
  .mockReturnValue('default')
  .mockReturnValueOnce('first')
  .mockResolvedValue('async');

// Implementation
const mockFn = vi.fn((x) => x * 2);
```

### Module Mocking

```typescript
// Mock entire module
vi.mock('@/services/api/user.service', () => ({
  userService: {
    getById: vi.fn(),
    create: vi.fn(),
  },
}));

// Mock with actual implementation
vi.mock('@/lib/utils', async () => {
  const actual = await vi.importActual('@/lib/utils');
  return {
    ...actual,
    expensiveOperation: vi.fn(),
  };
});

// Access mocked module
import { userService } from '@/services/api/user.service';
vi.mocked(userService.getById).mockResolvedValue({ id: '1', name: 'Test' });
```

### Spying

```typescript
// Spy on method
const spy = vi.spyOn(object, 'method');
object.method('arg');
expect(spy).toHaveBeenCalledWith('arg');

// Spy and mock
vi.spyOn(console, 'error').mockImplementation(() => {});
```

---

## Hoisted Mocks

For mocks that need to work with stateful tests:

```typescript
// vi.hoisted runs before imports
const { mockState, setMockState } = vi.hoisted(() => {
  const state = { value: 'initial' };
  return {
    mockState: state,
    setMockState: (v: string) => { state.value = v; },
  };
});

vi.mock('@/contexts/AuthContext', () => ({
  useAuth: vi.fn(() => mockState),
}));

it('test with different state', () => {
  setMockState('authenticated');
  // Now useAuth returns 'authenticated'
});
```

---

## Test Utilities

### Timers

```typescript
beforeEach(() => {
  vi.useFakeTimers();
});

afterEach(() => {
  vi.useRealTimers();
});

it('handles timeout', () => {
  const callback = vi.fn();
  setTimeout(callback, 1000);

  vi.advanceTimersByTime(1000);
  expect(callback).toHaveBeenCalled();
});
```

### Environment Variables

```typescript
it('uses env variable', () => {
  vi.stubEnv('API_URL', 'https://test.api.com');
  expect(process.env.API_URL).toBe('https://test.api.com');
});
```

---

## Testing Pure Functions

```typescript
// src/lib/formatters.ts
export function formatCurrency(amount: number): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(amount);
}

// src/lib/__tests__/formatters.test.ts
describe('formatCurrency', () => {
  it('formats positive amounts', () => {
    expect(formatCurrency(100)).toBe('$100.00');
    expect(formatCurrency(1234.56)).toBe('$1,234.56');
  });

  it('formats zero', () => {
    expect(formatCurrency(0)).toBe('$0.00');
  });

  it('formats negative amounts', () => {
    expect(formatCurrency(-50)).toBe('-$50.00');
  });
});
```

---

## Testing Services

```typescript
// src/services/api/__tests__/profile.service.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { profileService } from '../profile.service';
import { supabase } from '@/integrations/supabase/client';

vi.mock('@/integrations/supabase/client', () => ({
  supabase: {
    from: vi.fn(() => ({
      select: vi.fn().mockReturnThis(),
      eq: vi.fn().mockReturnThis(),
      single: vi.fn(),
    })),
  },
}));

describe('profileService', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  describe('getById', () => {
    it('returns profile when found', async () => {
      const mockProfile = { id: '1', name: 'Test' };
      vi.mocked(supabase.from).mockReturnValue({
        select: vi.fn().mockReturnThis(),
        eq: vi.fn().mockReturnThis(),
        single: vi.fn().mockResolvedValue({ data: mockProfile, error: null }),
      } as any);

      const result = await profileService.getById('1');
      expect(result).toEqual(mockProfile);
    });

    it('throws when not found', async () => {
      vi.mocked(supabase.from).mockReturnValue({
        select: vi.fn().mockReturnThis(),
        eq: vi.fn().mockReturnThis(),
        single: vi.fn().mockResolvedValue({
          data: null,
          error: { message: 'Not found' },
        }),
      } as any);

      await expect(profileService.getById('1')).rejects.toThrow();
    });
  });
});
```

---

## Best Practices

1. **One assertion focus per test** - Test one behavior
2. **Clear test names** - Describe what should happen
3. **Arrange-Act-Assert** - Structure tests clearly
4. **Don't test implementation** - Test behavior
5. **Use descriptive builders** - Make test data clear
6. **Clear mocks in beforeEach** - Prevent test pollution
