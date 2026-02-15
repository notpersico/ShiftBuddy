# Adding an API Hook

This guide walks through adding a new data fetching feature using the Schema → Service → Hook pattern.

## Overview

Data features follow this structure:
1. **Schema** - Zod validation (`src/schemas/`)
2. **Service** - Supabase queries (`src/services/api/`)
3. **Hook** - React Query wrapper (`src/hooks/api/`)
4. **Tests** - Hook tests (`src/hooks/api/__tests__/`)

---

## Step 1: Create Schema

Define the data shape and validation with Zod:

```typescript
// src/schemas/product.schema.ts
import { z } from 'zod';

export const productSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(200),
  description: z.string().optional(),
  price: z.number().positive(),
  category: z.enum(['electronics', 'clothing', 'food']),
  created_at: z.string().datetime(),
  updated_at: z.string().datetime(),
});

export type Product = z.infer<typeof productSchema>;

// For create operations (no id, timestamps)
export const createProductSchema = productSchema.omit({
  id: true,
  created_at: true,
  updated_at: true,
});

export type CreateProductInput = z.infer<typeof createProductSchema>;

// For update operations (all fields optional)
export const updateProductSchema = createProductSchema.partial();

export type UpdateProductInput = z.infer<typeof updateProductSchema>;
```

## Step 2: Create Service

Implement the Supabase queries:

```typescript
// src/services/api/product.service.ts
import { supabase } from '@/integrations/supabase/client';
import type { Product, CreateProductInput, UpdateProductInput } from '@/schemas/product.schema';

export interface ProductFilters {
  category?: string;
  minPrice?: number;
  maxPrice?: number;
}

export const productService = {
  async getAll(filters?: ProductFilters): Promise<Product[]> {
    let query = supabase
      .from('products')
      .select('*')
      .order('created_at', { ascending: false });

    if (filters?.category) {
      query = query.eq('category', filters.category);
    }
    if (filters?.minPrice) {
      query = query.gte('price', filters.minPrice);
    }
    if (filters?.maxPrice) {
      query = query.lte('price', filters.maxPrice);
    }

    const { data, error } = await query;
    if (error) throw error;
    return data;
  },

  async getById(id: string): Promise<Product> {
    const { data, error } = await supabase
      .from('products')
      .select('*')
      .eq('id', id)
      .single();

    if (error) throw error;
    return data;
  },

  async create(input: CreateProductInput): Promise<Product> {
    const { data, error } = await supabase
      .from('products')
      .insert(input)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  async update(id: string, input: UpdateProductInput): Promise<Product> {
    const { data, error } = await supabase
      .from('products')
      .update(input)
      .eq('id', id)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  async delete(id: string): Promise<void> {
    const { error } = await supabase
      .from('products')
      .delete()
      .eq('id', id);

    if (error) throw error;
  },
};
```

## Step 3: Add Query Keys

Add keys to the query key factory:

```typescript
// src/hooks/api/query-keys.ts
export const queryKeys = {
  // ... existing keys

  products: (() => {
    const base = createKeyFactory(['products'] as const);
    return {
      all: base.all,
      lists: () => base.extend('list'),
      list: (filters: Record<string, unknown>) => base.extend('list', filters),
      details: () => base.extend('detail'),
      detail: (id: string) => base.extend('detail', id),
    };
  })(),
};
```

## Step 4: Create React Query Hooks

Wrap the service in React Query hooks:

```typescript
// src/hooks/api/useProducts.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { queryKeys } from './query-keys';
import { productService, type ProductFilters } from '@/services/api/product.service';
import type { CreateProductInput, UpdateProductInput } from '@/schemas/product.schema';

// Query: List products
export function useProducts(filters?: ProductFilters) {
  return useQuery({
    queryKey: queryKeys.products.list(filters ?? {}),
    queryFn: () => productService.getAll(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
}

// Query: Single product
export function useProduct(id: string) {
  return useQuery({
    queryKey: queryKeys.products.detail(id),
    queryFn: () => productService.getById(id),
    enabled: !!id,
  });
}

// Mutation: Create product
export function useCreateProduct() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (input: CreateProductInput) => productService.create(input),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.products.lists() });
    },
  });
}

// Mutation: Update product
export function useUpdateProduct() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, input }: { id: string; input: UpdateProductInput }) =>
      productService.update(id, input),
    onSuccess: (data) => {
      queryClient.setQueryData(queryKeys.products.detail(data.id), data);
      queryClient.invalidateQueries({ queryKey: queryKeys.products.lists() });
    },
  });
}

// Mutation: Delete product
export function useDeleteProduct() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id: string) => productService.delete(id),
    onSuccess: (_, id) => {
      queryClient.removeQueries({ queryKey: queryKeys.products.detail(id) });
      queryClient.invalidateQueries({ queryKey: queryKeys.products.lists() });
    },
  });
}
```

## Step 5: Add Tests

Create tests for the hooks:

```typescript
// src/hooks/api/__tests__/useProducts.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useProducts, useProduct, useCreateProduct } from '../useProducts';
import { productService } from '@/services/api/product.service';

// Mock the service
vi.mock('@/services/api/product.service', () => ({
  productService: {
    getAll: vi.fn(),
    getById: vi.fn(),
    create: vi.fn(),
  },
}));

function createWrapper() {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
}

describe('useProducts', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should fetch products', async () => {
    const mockProducts = [
      { id: '1', name: 'Product 1', price: 100 },
      { id: '2', name: 'Product 2', price: 200 },
    ];
    vi.mocked(productService.getAll).mockResolvedValue(mockProducts);

    const { result } = renderHook(() => useProducts(), {
      wrapper: createWrapper(),
    });

    await waitFor(() => expect(result.current.isSuccess).toBe(true));
    expect(result.current.data).toEqual(mockProducts);
  });

  it('should apply filters', async () => {
    const filters = { category: 'electronics' };
    vi.mocked(productService.getAll).mockResolvedValue([]);

    const { result } = renderHook(() => useProducts(filters), {
      wrapper: createWrapper(),
    });

    await waitFor(() => expect(result.current.isSuccess).toBe(true));
    expect(productService.getAll).toHaveBeenCalledWith(filters);
  });
});
```

---

## Usage in Components

```typescript
// src/components/ProductList.tsx
import { useProducts } from '@/hooks/api/useProducts';

function ProductList({ filters }: { filters?: ProductFilters }) {
  const { data: products, isLoading, error } = useProducts(filters);

  if (isLoading) return <ProductListSkeleton />;
  if (error) return <ErrorMessage error={error} />;
  if (!products?.length) return <EmptyState message="No products found" />;

  return (
    <div className="grid gap-4">
      {products.map((product) => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

---

## Checklist

- [ ] Zod schema created with proper types
- [ ] Service layer with Supabase queries
- [ ] Query keys added to factory
- [ ] React Query hooks (queries + mutations)
- [ ] Tests written for hooks
- [ ] TypeScript compiles without errors
- [ ] `pnpm test:run` passes
