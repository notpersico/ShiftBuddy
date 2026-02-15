# Adding a Feature

This guide walks through adding a complete feature module to the codebase.

## Overview

A feature module typically includes:
- Database schema (migration)
- Zod validation schemas
- Service layer
- React Query hooks
- UI components
- Tests

---

## Example: Adding a "Comments" Feature

### Step 1: Database Migration

Create the database table:

```bash
npx supabase migration new create_comments_table
```

```sql
-- supabase/migrations/YYYYMMDDHHMMSS_create_comments_table.sql

-- Create table
CREATE TABLE comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
  author_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
  body TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(post_id, author_id) -- One comment per user per post
);

-- Enable RLS
ALTER TABLE comments ENABLE ROW LEVEL SECURITY;

-- Policies
CREATE POLICY "Anyone can read comments"
  ON comments FOR SELECT
  USING (true);

CREATE POLICY "Authenticated users can create comments"
  ON comments FOR INSERT
  TO authenticated
  WITH CHECK (auth.uid() = author_id);

CREATE POLICY "Users can update own comments"
  ON comments FOR UPDATE
  USING (auth.uid() = author_id);

CREATE POLICY "Users can delete own comments"
  ON comments FOR DELETE
  USING (auth.uid() = author_id);

-- Index for performance
CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_comments_author_id ON comments(author_id);
```

Apply and generate types:

```bash
npx supabase db reset
npx supabase gen types typescript --local > src/integrations/supabase/types.ts
```

### Step 2: Zod Schema

```typescript
// src/schemas/comment.schema.ts
import { z } from 'zod';

export const commentSchema = z.object({
  id: z.string().uuid(),
  post_id: z.string().uuid(),
  author_id: z.string().uuid(),
  rating: z.number().int().min(1).max(5),
  body: z.string().max(1000).optional(),
  created_at: z.string().datetime(),
  updated_at: z.string().datetime(),
});

export type Comment = z.infer<typeof commentSchema>;

export const createCommentSchema = z.object({
  post_id: z.string().uuid(),
  rating: z.number().int().min(1).max(5),
  body: z.string().max(1000).optional(),
});

export type CreateCommentInput = z.infer<typeof createCommentSchema>;

export const updateCommentSchema = z.object({
  rating: z.number().int().min(1).max(5).optional(),
  body: z.string().max(1000).optional(),
});

export type UpdateCommentInput = z.infer<typeof updateCommentSchema>;
```

### Step 3: Service Layer

```typescript
// src/services/api/comment.service.ts
import { supabase } from '@/integrations/supabase/client';
import type { Comment, CreateCommentInput, UpdateCommentInput } from '@/schemas/comment.schema';

export const commentService = {
  async getByPost(postId: string): Promise<Comment[]> {
    const { data, error } = await supabase
      .from('comments')
      .select('*')
      .eq('post_id', postId)
      .order('created_at', { ascending: false });

    if (error) throw error;
    return data;
  },

  async getPostStats(postId: string) {
    const { data, error } = await supabase
      .from('comments')
      .select('rating')
      .eq('post_id', postId);

    if (error) throw error;

    const total = data.length;
    const average = total > 0
      ? data.reduce((sum, r) => sum + r.rating, 0) / total
      : 0;

    return { total, average: Math.round(average * 10) / 10 };
  },

  async create(input: CreateCommentInput): Promise<Comment> {
    const { data, error } = await supabase
      .from('comments')
      .insert(input)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  async update(id: string, input: UpdateCommentInput): Promise<Comment> {
    const { data, error } = await supabase
      .from('comments')
      .update({ ...input, updated_at: new Date().toISOString() })
      .eq('id', id)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  async delete(id: string): Promise<void> {
    const { error } = await supabase
      .from('comments')
      .delete()
      .eq('id', id);

    if (error) throw error;
  },
};
```

### Step 4: Query Keys

```typescript
// Add to src/hooks/api/query-keys.ts
export const queryKeys = {
  // ... existing keys

  comments: (() => {
    const base = createKeyFactory(['comments'] as const);
    return {
      all: base.all,
      byPost: (postId: string) => base.extend('post', postId),
      stats: (postId: string) => base.extend('stats', postId),
    };
  })(),
};
```

### Step 5: React Query Hooks

```typescript
// src/hooks/api/useComments.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { queryKeys } from './query-keys';
import { commentService } from '@/services/api/comment.service';
import type { CreateCommentInput, UpdateCommentInput } from '@/schemas/comment.schema';

export function usePostComments(postId: string) {
  return useQuery({
    queryKey: queryKeys.comments.byPost(postId),
    queryFn: () => commentService.getByPost(postId),
    enabled: !!postId,
  });
}

export function usePostCommentStats(postId: string) {
  return useQuery({
    queryKey: queryKeys.comments.stats(postId),
    queryFn: () => commentService.getPostStats(postId),
    enabled: !!postId,
    staleTime: 5 * 60 * 1000,
  });
}

export function useCreateComment() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: commentService.create,
    onSuccess: (data) => {
      queryClient.invalidateQueries({
        queryKey: queryKeys.comments.byPost(data.post_id),
      });
      queryClient.invalidateQueries({
        queryKey: queryKeys.comments.stats(data.post_id),
      });
    },
  });
}

export function useUpdateComment(postId: string) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, input }: { id: string; input: UpdateCommentInput }) =>
      commentService.update(id, input),
    onSuccess: () => {
      queryClient.invalidateQueries({
        queryKey: queryKeys.comments.byPost(postId),
      });
      queryClient.invalidateQueries({
        queryKey: queryKeys.comments.stats(postId),
      });
    },
  });
}

export function useDeleteComment(postId: string) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: commentService.delete,
    onSuccess: () => {
      queryClient.invalidateQueries({
        queryKey: queryKeys.comments.byPost(postId),
      });
      queryClient.invalidateQueries({
        queryKey: queryKeys.comments.stats(postId),
      });
    },
  });
}
```

### Step 6: UI Components

```typescript
// src/components/comments/CommentCard.tsx
import { Card, CardContent } from '@/components/ui/card';
import { StarRating } from '@/components/ui/star-rating';
import type { Comment } from '@/schemas/comment.schema';

interface Props {
  comment: Comment;
  onEdit?: () => void;
  onDelete?: () => void;
  isOwn?: boolean;
}

export function CommentCard({ comment, onEdit, onDelete, isOwn }: Props) {
  return (
    <Card>
      <CardContent className="p-4">
        <div className="flex justify-between">
          <StarRating value={comment.rating} readonly />
          {isOwn && (
            <div className="space-x-2">
              <Button variant="ghost" size="sm" onClick={onEdit}>
                Edit
              </Button>
              <Button variant="ghost" size="sm" onClick={onDelete}>
                Delete
              </Button>
            </div>
          )}
        </div>
        {comment.body && <p className="mt-2">{comment.body}</p>}
        <p className="text-sm text-muted-foreground mt-2">
          {new Date(comment.created_at).toLocaleDateString()}
        </p>
      </CardContent>
    </Card>
  );
}
```

```typescript
// src/components/comments/CommentList.tsx
import { usePostComments } from '@/hooks/api/useComments';
import { CommentCard } from './CommentCard';
import { useAuth } from '@/contexts/AuthContext';

interface Props {
  postId: string;
}

export function CommentList({ postId }: Props) {
  const { user } = useAuth();
  const { data: comments, isLoading } = usePostComments(postId);

  if (isLoading) return <CommentListSkeleton />;
  if (!comments?.length) return <EmptyState message="No comments yet" />;

  return (
    <div className="space-y-4">
      {comments.map((comment) => (
        <CommentCard
          key={comment.id}
          comment={comment}
          isOwn={comment.author_id === user?.id}
        />
      ))}
    </div>
  );
}
```

### Step 7: Tests

```typescript
// src/hooks/api/__tests__/useComments.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { renderHook, waitFor } from '@testing-library/react';
import { usePostComments, useCreateComment } from '../useComments';
import { commentService } from '@/services/api/comment.service';
import { createWrapper } from '@/test/utils';

vi.mock('@/services/api/comment.service');

describe('usePostComments', () => {
  beforeEach(() => vi.clearAllMocks());

  it('should fetch comments for a post', async () => {
    const mockComments = [
      { id: '1', rating: 5, body: 'Great post!' },
    ];
    vi.mocked(commentService.getByPost).mockResolvedValue(mockComments);

    const { result } = renderHook(
      () => usePostComments('post-123'),
      { wrapper: createWrapper() }
    );

    await waitFor(() => expect(result.current.isSuccess).toBe(true));
    expect(result.current.data).toEqual(mockComments);
  });
});
```

---

## Feature Checklist

- [ ] Database migration created
- [ ] RLS policies added
- [ ] Types generated
- [ ] Zod schemas defined
- [ ] Service layer implemented
- [ ] Query keys added
- [ ] React Query hooks created
- [ ] UI components built
- [ ] Tests written
- [ ] `pnpm typecheck && pnpm lint && pnpm test:run` passes
