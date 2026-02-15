# React 19 Best Practices

React 19 introduces significant changes including the React Compiler for automatic memoization, new hooks for form handling, and simplified APIs.

## Table of Contents

1. [React Compiler (No-Memo Workflow)](#1-react-compiler-no-memo-workflow)
2. [Form Actions with useActionState](#2-form-actions-with-useactionstate)
3. [The use() API](#3-the-use-api)
4. [useOptimistic Hook](#4-useoptimistic-hook)
5. [useFormStatus Hook](#5-useformstatus-hook)
6. [Suspense Patterns](#6-suspense-patterns)
7. [Ref Improvements](#7-ref-improvements)
8. [When to Use What](#8-when-to-use-what)

---

## 1. React Compiler (No-Memo Workflow)

The React Compiler automatically optimizes components, eliminating manual memoization in most cases.

### What It Does

- **Automatic memoization** of component outputs
- **Stabilizes function references** across renders
- **Caches expensive calculations** automatically
- **Analyzes dependency changes** without manual arrays

### Before vs After

```typescript
// Before (React 18) - manual optimization
const MemoizedComponent = React.memo(({ data }) => {
  const processed = useMemo(() => expensiveCalc(data), [data]);
  const handleClick = useCallback(() => doSomething(data), [data]);
  return <div onClick={handleClick}>{processed}</div>;
});

// After (React 19) - compiler handles it
function Component({ data }: { data: Data }) {
  const processed = expensiveCalc(data);
  const handleClick = () => doSomething(data);
  return <div onClick={handleClick}>{processed}</div>;
}
```

### When Manual Optimization Is Still Needed

Only add `useMemo`/`useCallback`/`React.memo` when:

1. **Profiler shows a specific bottleneck**
2. **Third-party library requires stable references**
3. **Complex dependency chains** the compiler cannot analyze

### Rules for Compiler Compatibility

```typescript
// DO: Pure render logic
function GoodComponent({ items }: Props) {
  const sorted = items.toSorted((a, b) => a.name.localeCompare(b.name));
  return <List items={sorted} />;
}

// DON'T: Side effects during render
function BadComponent({ items }: Props) {
  items.sort((a, b) => a.name.localeCompare(b.name)); // Mutates prop!
  trackAnalytics(); // Side effect during render!
  return <List items={items} />;
}
```

---

## 2. Form Actions with useActionState

Replace manual `onSubmit` handlers with declarative form actions.

### Syntax

```typescript
const [state, formAction, isPending] = useActionState(fn, initialState);
```

### Complete Example

```typescript
interface FormState {
  message: string;
  success: boolean;
  errors?: Record<string, string>;
}

async function updateProfile(
  prevState: FormState,
  formData: FormData
): Promise<FormState> {
  const username = formData.get('username') as string;

  if (!username || username.length < 3) {
    return {
      message: 'Username must be at least 3 characters',
      success: false,
      errors: { username: 'Too short' },
    };
  }

  const { error } = await supabase
    .from('profiles')
    .update({ username })
    .eq('id', userId);

  if (error) {
    return { message: error.message, success: false };
  }

  return { message: 'Profile updated!', success: true };
}

function ProfileForm() {
  const [state, formAction, isPending] = useActionState(updateProfile, {
    message: '',
    success: false,
  });

  return (
    <form action={formAction}>
      <input
        name="username"
        disabled={isPending}
        aria-invalid={!!state.errors?.username}
      />
      {state.errors?.username && (
        <p className="text-destructive">{state.errors.username}</p>
      )}
      <button type="submit" disabled={isPending}>
        {isPending ? 'Saving...' : 'Save'}
      </button>
      {state.message && <p>{state.message}</p>}
    </form>
  );
}
```

### Benefits Over Traditional Handlers

| Traditional `onSubmit` | `useActionState` |
|------------------------|------------------|
| Manual `useState` for loading | Built-in `isPending` |
| Manual error state management | State includes errors |
| No progressive enhancement | Works without JS |

---

## 3. The use() API

The `use()` hook reads resources (Promises or Context) during render. Unlike other hooks, it can be called conditionally.

### Reading Promises

```typescript
import { use, Suspense } from 'react';

function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise); // Suspends until resolved
  return <h1>{user.name}</h1>;
}

function Page() {
  const userPromise = fetchUser(); // Create outside render

  return (
    <Suspense fallback={<Loading />}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

### Reading Context Conditionally

```typescript
function ConditionalThemedButton({ useTheme }: { useTheme: boolean }) {
  if (useTheme) {
    const theme = use(ThemeContext); // Can be conditional!
    return <button className={theme.buttonClass}>Themed</button>;
  }
  return <button>Default</button>;
}
```

---

## 4. useOptimistic Hook

Show immediate UI feedback while async actions complete.

```typescript
function LikeButton({ post }: { post: Post }) {
  const [optimisticPost, addOptimisticLike] = useOptimistic(
    post,
    (currentPost, newLikeState: boolean) => ({
      ...currentPost,
      isLiked: newLikeState,
      likes: currentPost.likes + (newLikeState ? 1 : -1),
    })
  );

  async function handleLike() {
    const newLikeState = !optimisticPost.isLiked;
    addOptimisticLike(newLikeState); // Update UI immediately
    await toggleLike(post.id, newLikeState); // API call
  }

  return (
    <button onClick={handleLike}>
      {optimisticPost.isLiked ? 'Unlike' : 'Like'} ({optimisticPost.likes})
    </button>
  );
}
```

---

## 5. useFormStatus Hook

Access form submission status from child components.

```typescript
import { useFormStatus } from 'react-dom';

function SubmitButton({ children }: { children: React.ReactNode }) {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : children}
    </button>
  );
}

// Usage - must be inside a form
function ContactForm() {
  return (
    <form action={submitContact}>
      <input name="email" type="email" />
      <SubmitButton>Send Message</SubmitButton>
    </form>
  );
}
```

**Important**: `useFormStatus` must be called from a component rendered inside the `<form>`.

---

## 6. Suspense Patterns

### Data Fetching Pattern

```typescript
function ProfilePage({ userId }: { userId: string }) {
  return (
    <div>
      <h1>Profile</h1>
      <Suspense fallback={<ProfileSkeleton />}>
        <ProfileDetails userId={userId} />
      </Suspense>
      <Suspense fallback={<PostsSkeleton />}>
        <UserPosts userId={userId} />
      </Suspense>
    </div>
  );
}
```

### With Error Boundaries

```typescript
function RobustDataSection() {
  return (
    <ErrorBoundary fallback={<ErrorMessage />}>
      <Suspense fallback={<Loading />}>
        <DataComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

## 7. Ref Improvements

### Ref as a Prop (No More forwardRef)

```typescript
// Before (React 18)
const Input = forwardRef<HTMLInputElement, InputProps>((props, ref) => {
  return <input ref={ref} {...props} />;
});

// After (React 19)
function Input({ ref, ...props }: InputProps & { ref?: React.Ref<HTMLInputElement> }) {
  return <input ref={ref} {...props} />;
}
```

### Ref Callback Cleanup

```typescript
function VideoPlayer({ src }: { src: string }) {
  return (
    <video
      ref={(node) => {
        if (!node) return;

        const player = new VideoLibrary(node);
        player.load(src);

        // Cleanup when unmounted or ref changes
        return () => player.destroy();
      }}
    />
  );
}
```

---

## 8. When to Use What

### Form Handling

| Scenario | Solution |
|----------|----------|
| Simple form, need pending state | `useActionState` |
| Form status in child component | `useFormStatus` |
| Optimistic UI | `useOptimistic` + `useActionState` |
| Complex multi-step form | Traditional `useState` + handlers |

### Data Fetching

| Scenario | Solution |
|----------|----------|
| With React Query (this codebase) | `useQuery` hooks |
| Promise passed as prop | `use()` + `Suspense` |
| Conditional data needs | `use()` (can be conditional) |

### State Management

| Scenario | Solution |
|----------|----------|
| Form submission state | `useActionState` |
| Optimistic updates | `useOptimistic` |
| Server cache | React Query |
| Local UI state | `useState` |
