# Component Testing with React Testing Library

Patterns for testing React components.

## Philosophy

React Testing Library encourages testing components the way users interact with them:
- Query by accessible roles and text
- Avoid testing implementation details
- Use user events, not direct DOM manipulation

---

## Basic Component Test

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ProfileCard } from '../ProfileCard';

describe('ProfileCard', () => {
  const mockProfile = {
    id: '1',
    name: 'Jane Doe',
    bio: 'Software developer',
    isPremium: false,
  };

  it('renders profile information', () => {
    render(<ProfileCard profile={mockProfile} />);

    expect(screen.getByText('Jane Doe')).toBeInTheDocument();
    expect(screen.getByText('Software developer')).toBeInTheDocument();
  });

  it('shows premium badge for premium profiles', () => {
    render(<ProfileCard profile={{ ...mockProfile, isPremium: true }} />);

    expect(screen.getByText('Premium')).toBeInTheDocument();
  });

  it('calls onFavorite when favorite button clicked', async () => {
    const user = userEvent.setup();
    const onFavorite = vi.fn();

    render(<ProfileCard profile={mockProfile} onFavorite={onFavorite} />);

    await user.click(screen.getByRole('button', { name: /favorite/i }));

    expect(onFavorite).toHaveBeenCalledWith('1');
  });
});
```

---

## Querying Elements

### Priority Order (Recommended)

1. **Accessible queries** (how users/assistive tech find elements)
   - `getByRole` - buttons, links, headings, etc.
   - `getByLabelText` - form inputs
   - `getByPlaceholderText` - inputs without labels
   - `getByText` - non-interactive content
   - `getByDisplayValue` - input current values

2. **Semantic queries**
   - `getByAltText` - images
   - `getByTitle` - elements with title attribute

3. **Test IDs** (last resort)
   - `getByTestId` - when no other option

### Query Variants

| Variant | No Match | 1 Match | Multiple |
|---------|----------|---------|----------|
| `getBy` | Throw | Return | Throw |
| `queryBy` | null | Return | Throw |
| `findBy` | Throw (async) | Return | Throw |
| `getAllBy` | Throw | Array | Array |
| `queryAllBy` | [] | Array | Array |
| `findAllBy` | Throw (async) | Array | Array |

### Examples

```typescript
// By role (preferred)
screen.getByRole('button', { name: /submit/i });
screen.getByRole('heading', { level: 1 });
screen.getByRole('textbox', { name: /email/i });
screen.getByRole('link', { name: /home/i });

// By label (for form inputs)
screen.getByLabelText('Email');
screen.getByLabelText(/password/i);

// By text content
screen.getByText('Welcome');
screen.getByText(/loading/i);

// By test ID (last resort)
screen.getByTestId('custom-element');

// Async queries (for elements that appear after async operations)
await screen.findByText('Data loaded');
await screen.findByRole('alert');
```

---

## User Interactions

Always use `userEvent` over `fireEvent`:

```typescript
import userEvent from '@testing-library/user-event';

describe('Form', () => {
  it('handles form submission', async () => {
    const user = userEvent.setup();
    const onSubmit = vi.fn();

    render(<ContactForm onSubmit={onSubmit} />);

    // Type in inputs
    await user.type(screen.getByLabelText('Name'), 'John Doe');
    await user.type(screen.getByLabelText('Email'), 'john@example.com');

    // Select from dropdown
    await user.selectOptions(screen.getByLabelText('Category'), 'support');

    // Check checkbox
    await user.click(screen.getByLabelText('Subscribe to newsletter'));

    // Submit form
    await user.click(screen.getByRole('button', { name: /submit/i }));

    expect(onSubmit).toHaveBeenCalledWith({
      name: 'John Doe',
      email: 'john@example.com',
      category: 'support',
      subscribe: true,
    });
  });
});
```

### Common User Actions

```typescript
const user = userEvent.setup();

// Click
await user.click(element);
await user.dblClick(element);
await user.tripleClick(element);

// Type
await user.type(input, 'text');
await user.clear(input);

// Keyboard
await user.keyboard('{Enter}');
await user.keyboard('{Tab}');
await user.keyboard('{Escape}');

// Select
await user.selectOptions(select, 'option-value');
await user.selectOptions(select, ['opt1', 'opt2']); // multi-select

// Hover
await user.hover(element);
await user.unhover(element);

// Upload
await user.upload(fileInput, file);
```

---

## Testing Async Behavior

### Wait for Elements

```typescript
it('shows data after loading', async () => {
  render(<DataList />);

  // Element appears after async operation
  await waitFor(() => {
    expect(screen.getByText('Item 1')).toBeInTheDocument();
  });
});

// Or use findBy (combines getBy + waitFor)
it('shows data after loading', async () => {
  render(<DataList />);

  expect(await screen.findByText('Item 1')).toBeInTheDocument();
});
```

### Wait for Removal

```typescript
it('hides loading indicator', async () => {
  render(<DataList />);

  await waitForElementToBeRemoved(() => screen.queryByText('Loading...'));

  expect(screen.getByText('Data loaded')).toBeInTheDocument();
});
```

### Wait for Condition

```typescript
it('button becomes enabled', async () => {
  render(<Form />);

  const submitButton = screen.getByRole('button', { name: /submit/i });
  expect(submitButton).toBeDisabled();

  // Fill form
  await userEvent.type(screen.getByLabelText('Name'), 'John');

  await waitFor(() => {
    expect(submitButton).toBeEnabled();
  });
});
```

---

## Testing with Context/Providers

### Custom Render Function

```typescript
// src/test/utils.tsx
import { render, RenderOptions } from '@testing-library/react';
import { QueryClientProvider } from '@tanstack/react-query';
import { AuthProvider } from '@/contexts/AuthContext';
import { RouterProvider } from '@/contexts/RouterContext';

interface CustomRenderOptions extends RenderOptions {
  initialRoute?: string;
  authenticated?: boolean;
}

function customRender(
  ui: React.ReactElement,
  options: CustomRenderOptions = {}
) {
  const { initialRoute, authenticated, ...renderOptions } = options;

  const queryClient = createTestQueryClient();

  const Wrapper = ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      <AuthProvider initialUser={authenticated ? mockUser : null}>
        <RouterProvider initialRoute={initialRoute}>
          {children}
        </RouterProvider>
      </AuthProvider>
    </QueryClientProvider>
  );

  return render(ui, { wrapper: Wrapper, ...renderOptions });
}

export { customRender as render };
```

### Usage

```typescript
import { render, screen } from '@/test/utils';

it('shows user name when authenticated', () => {
  render(<Header />, { authenticated: true });

  expect(screen.getByText('John Doe')).toBeInTheDocument();
});
```

---

## Testing Forms

```typescript
describe('LoginForm', () => {
  it('validates required fields', async () => {
    const user = userEvent.setup();
    render(<LoginForm />);

    // Submit empty form
    await user.click(screen.getByRole('button', { name: /login/i }));

    expect(screen.getByText('Email is required')).toBeInTheDocument();
    expect(screen.getByText('Password is required')).toBeInTheDocument();
  });

  it('validates email format', async () => {
    const user = userEvent.setup();
    render(<LoginForm />);

    await user.type(screen.getByLabelText('Email'), 'invalid-email');
    await user.click(screen.getByRole('button', { name: /login/i }));

    expect(screen.getByText('Invalid email address')).toBeInTheDocument();
  });

  it('submits valid form', async () => {
    const user = userEvent.setup();
    const onSubmit = vi.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText('Email'), 'test@example.com');
    await user.type(screen.getByLabelText('Password'), 'password123');
    await user.click(screen.getByRole('button', { name: /login/i }));

    expect(onSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123',
    });
  });
});
```

---

## Accessibility Testing

```typescript
it('is accessible', async () => {
  const { container } = render(<ProfileCard profile={mockProfile} />);

  // Check for accessibility violations (requires jest-axe or similar)
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

it('has proper ARIA attributes', () => {
  render(<Dialog open={true} />);

  const dialog = screen.getByRole('dialog');
  expect(dialog).toHaveAttribute('aria-modal', 'true');
  expect(dialog).toHaveAttribute('aria-labelledby');
});

it('supports keyboard navigation', async () => {
  const user = userEvent.setup();
  render(<Menu />);

  await user.keyboard('{Tab}');
  expect(screen.getByRole('menuitem', { name: 'Home' })).toHaveFocus();

  await user.keyboard('{ArrowDown}');
  expect(screen.getByRole('menuitem', { name: 'About' })).toHaveFocus();
});
```

---

## Debugging

```typescript
// Print current DOM
screen.debug();

// Print specific element
screen.debug(screen.getByRole('button'));

// Log accessible tree
import { logRoles } from '@testing-library/react';
const { container } = render(<MyComponent />);
logRoles(container);
```
