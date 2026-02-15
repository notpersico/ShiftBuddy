# Adding a Route

This guide walks through adding a new route to the application.

## Overview

Routes in this codebase are managed via:
1. Route type definition (`src/types/router.types.ts`)
2. Page component (`src/components/`)
3. Router switch (`src/App.tsx`)
4. Protection arrays (public vs protected)

---

## Step 1: Add Route Type

Add the new route name to the `AppRoute` union type:

```typescript
// src/types/router.types.ts
export type AppRoute =
  | 'home'
  | 'login'
  | 'profile'
  // Add your new route
  | 'settings';
```

## Step 2: Create Page Component

Create the page component with lazy loading support:

```typescript
// src/components/pages/SettingsPage.tsx
import { useAuth } from '@/contexts/AuthContext';

export function SettingsPage() {
  const { user } = useAuth();

  return (
    <div className="container mx-auto p-4">
      <h1 className="text-2xl font-bold">Settings</h1>
      <p>Welcome, {user?.email}</p>
      {/* Page content */}
    </div>
  );
}

// Default export for lazy loading
export default SettingsPage;
```

## Step 3: Add to Router Switch

Add the lazy import and switch case in `App.tsx`:

```typescript
// src/App.tsx

// Add lazy import at top
const SettingsPage = lazy(() => import('@/components/pages/SettingsPage'));

// In the switch statement inside AppContent
function AppContent() {
  const { currentRoute } = useRouter();

  switch (currentRoute) {
    // ... existing cases
    case 'settings':
      return <SettingsPage />;
    default:
      return <NotFoundPage />;
  }
}
```

## Step 4: Add to Protection Array

Add to the appropriate array based on whether the route requires authentication:

```typescript
// src/App.tsx

// For routes requiring authentication
const protectedRoutes: AppRoute[] = [
  'home',
  'profile',
  'settings', // Add here if protected
];

// For public routes (no auth required)
const publicRoutes: AppRoute[] = [
  'login',
  'register',
  // 'settings', // Add here if public
];
```

## Step 5: Add Navigation (Optional)

Add links/buttons to navigate to the new route:

```typescript
// In a navbar or menu component
import { useRouter } from '@/contexts/RouterContext';

function Navbar() {
  const { navigate, currentRoute } = useRouter();

  return (
    <nav>
      <button
        onClick={() => navigate('settings')}
        className={cn(currentRoute === 'settings' && 'active')}
      >
        Settings
      </button>
    </nav>
  );
}
```

---

## Route with Parameters

For routes that need URL parameters:

### Navigate with Parameters

```typescript
// Pass parameters when navigating
navigate('item-detail', { id: item.id });
```

### Access Parameters in Page

```typescript
function ItemDetailPage() {
  const { params } = useRouter();
  const itemId = params.id;

  const { data: item } = useItemDetail(itemId);

  return <ItemProfile item={item} />;
}
```

---

## Role-Based Routes

For routes restricted to specific roles:

```typescript
function AdminPage() {
  const { role } = useAuth();

  if (role !== 'admin') {
    return <AccessDenied />;
  }

  return <AdminDashboard />;
}
```

Or use a wrapper component:

```typescript
<ProtectedRoute requiredRole="admin">
  <AdminPage />
</ProtectedRoute>
```

---

## Checklist

- [ ] Route type added to `AppRoute` union
- [ ] Page component created with default export
- [ ] Lazy import added in `App.tsx`
- [ ] Switch case added in `App.tsx`
- [ ] Added to protectedRoutes or publicRoutes
- [ ] Navigation links added where needed
- [ ] TypeScript compiles without errors
- [ ] Route accessible in browser
