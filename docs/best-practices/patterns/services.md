# Service Patterns

Patterns for the service layer.

## Basic Service Structure

```typescript
// src/services/api/profile.service.ts
import { supabase } from '@/integrations/supabase/client';
import type { Profile, CreateProfileInput, UpdateProfileInput } from '@/types';

export const profileService = {
  async getById(id: string): Promise<Profile> {
    const { data, error } = await supabase
      .from('profiles')
      .select('*')
      .eq('id', id)
      .single();

    if (error) throw error;
    return data;
  },

  async search(filters?: ProfileFilters): Promise<Profile[]> {
    let query = supabase.from('profiles').select('*').eq('is_active', true);

    if (filters?.location) {
      query = query.ilike('location', `%${filters.location}%`);
    }

    const { data, error } = await query;
    if (error) throw error;
    return data;
  },

  async create(input: CreateProfileInput): Promise<Profile> {
    const { data, error } = await supabase
      .from('profiles')
      .insert(input)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  async update(id: string, input: UpdateProfileInput): Promise<Profile> {
    const { data, error } = await supabase
      .from('profiles')
      .update(input)
      .eq('id', id)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  async delete(id: string): Promise<void> {
    const { error } = await supabase.from('profiles').delete().eq('id', id);
    if (error) throw error;
  },
};
```

---

## Complex Queries

```typescript
async function getProviderWithRelations(id: string) {
  const { data, error } = await supabase
    .from('provider_profiles')
    .select(`
      *,
      photos:provider_photos(id, url, order),
      services:provider_services(id, name, price),
      reviews:reviews(id, rating, comment, created_at)
    `)
    .eq('id', id)
    .single();

  if (error) throw error;
  return data;
}
```

---

## Light Transformation

Transform data only when necessary:

```typescript
async function getProviderWithStats(id: string) {
  const { data, error } = await supabase
    .from('provider_profiles')
    .select(`
      *,
      photos:provider_photos(url, order),
      reviews:reviews(rating)
    `)
    .eq('id', id)
    .single();

  if (error) throw error;

  // Light transformation for computed values
  return {
    ...data,
    primaryPhoto: data.photos?.find((p) => p.order === 0)?.url ?? null,
    averageRating:
      data.reviews?.length > 0
        ? data.reviews.reduce((sum, r) => sum + r.rating, 0) / data.reviews.length
        : null,
    reviewCount: data.reviews?.length ?? 0,
  };
}
```

---

## RPC Calls

```typescript
async function getNearbyProviders(lat: number, lng: number, radiusKm = 50) {
  const { data, error } = await supabase.rpc('get_nearby_providers', {
    p_lat: lat,
    p_lng: lng,
    p_radius_km: radiusKm,
  });

  if (error) throw error;
  return data;
}
```

---

## Storage Operations

```typescript
export const storageService = {
  async uploadPhoto(file: File, profileId: string): Promise<string> {
    const fileName = `${profileId}/${Date.now()}-${file.name}`;

    const { error: uploadError } = await supabase.storage
      .from('user-photos')
      .upload(fileName, file);

    if (uploadError) throw uploadError;

    const { data } = supabase.storage.from('provider-photos').getPublicUrl(fileName);

    return data.publicUrl;
  },

  async deletePhoto(path: string): Promise<void> {
    const { error } = await supabase.storage.from('provider-photos').remove([path]);
    if (error) throw error;
  },
};
```

---

## Error Handling

Services throw errors for hooks to handle:

```typescript
async function updateProfile(id: string, input: UpdateProfileInput) {
  const { data, error } = await supabase
    .from('profiles')
    .update(input)
    .eq('id', id)
    .select()
    .single();

  if (error) {
    // Log for debugging
    console.error('Failed to update profile:', error.message);
    // Throw for hook/component to handle
    throw error;
  }

  return data;
}
```

---

## Batch Operations

```typescript
async function updateMultiplePhotos(updates: { id: string; order: number }[]) {
  // Use Promise.all for parallel updates
  const results = await Promise.all(
    updates.map(({ id, order }) =>
      supabase.from('photos').update({ order }).eq('id', id)
    )
  );

  // Check for errors
  const errors = results.filter((r) => r.error);
  if (errors.length > 0) {
    throw new Error(`Failed to update ${errors.length} photos`);
  }
}
```
