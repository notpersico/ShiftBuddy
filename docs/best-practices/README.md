# Tech Stack Best Practices

A comprehensive collection of best practices, architectural patterns, and development guides for building modern React + TypeScript + Supabase applications.

## Tech Stack Overview

| Technology | Purpose |
|------------|---------|
| React 19 | UI framework |
| TypeScript 5.9+ | Type safety |
| Vite + SWC | Build tooling |
| Tailwind CSS v4 | Styling |
| shadcn/ui | UI component primitives |
| Supabase | Backend (auth, database, storage, edge functions) |
| TanStack Query v5 | Server state management |
| TanStack Router | Type-safe routing |
| Jotai | Client state management |
| Vitest + RTL + MSW | Testing |

---

## Sections

### [Architecture](./architecture/)

Core architectural decisions and principles.

- [Design Principles](./architecture/design-principles.md) -- SOLID, composition, unidirectional data flow
- [Layer Separation](./architecture/layer-separation.md) -- Components, hooks, services, data layer
- [State Management](./architecture/state-management.md) -- The state ladder: server, URL, shared, local
- [Data Flow](./architecture/data-flow.md) -- Query, mutation, form, real-time, and error flows

### [Patterns](./patterns/)

Reusable code patterns and conventions.

- [Components](./patterns/components.md) -- CVA variants, composition, controlled/uncontrolled
- [Hooks](./patterns/hooks.md) -- Query hooks, mutation hooks, Suspense, infinite queries
- [Services](./patterns/services.md) -- Supabase queries, transformations, batch operations
- [Contexts](./patterns/contexts.md) -- React Context patterns, split contexts, provider composition
- [Testing](./patterns/testing.md) -- Test structure, builders, MSW handlers
- [Schemas](./patterns/schemas.md) -- Zod validation, refinements, form validation

### [Guides](./guides/)

Step-by-step development guides.

#### [Best Practices](./guides/best-practices/)

- [React 19](./guides/best-practices/react-19.md) -- Compiler, form actions, use(), Suspense, refs
- [TanStack Query](./guides/best-practices/tanstack-query.md) -- v5 patterns, cache invalidation, prefetching
- [Supabase](./guides/best-practices/supabase.md) -- Client queries, RPC, edge functions, RLS
- [TypeScript](./guides/best-practices/typescript.md) -- Strict mode, type guards, utility types
- [Anti-Patterns](./guides/best-practices/anti-patterns.md) -- Common mistakes and their fixes

#### [Testing](./guides/testing/)

- [Unit Testing](./guides/testing/unit-testing.md) -- Vitest patterns and utilities
- [Hook Testing](./guides/testing/hook-testing.md) -- Testing React Query hooks
- [Component Testing](./guides/testing/component-testing.md) -- React Testing Library patterns
- [MSW Handlers](./guides/testing/msw-handlers.md) -- Network mocking with MSW v2

#### [Development](./guides/development/)

- [Adding a Route](./guides/development/adding-a-route.md) -- TanStack Router route setup
- [Adding an API Hook](./guides/development/adding-an-api-hook.md) -- Schema, service, hook pattern
- [Adding a Component](./guides/development/adding-a-component.md) -- shadcn/ui and custom components
- [Adding a Feature](./guides/development/adding-a-feature.md) -- End-to-end feature module guide
- [Adding an Edge Function](./guides/development/adding-edge-function.md) -- Supabase edge functions
- [Database Migrations](./guides/development/database-migrations.md) -- Supabase CLI migration workflow

### [Contributing](./contributing/)

Team standards and workflow.

- [Code Style](./contributing/code-style.md) -- TypeScript, Prettier, ESLint, naming conventions
- [Commit Conventions](./contributing/commit-conventions.md) -- Conventional Commits format
- [Pull Request Process](./contributing/pull-request-process.md) -- PR creation, review, and merging

### [ADR](./adr/)

Architecture Decision Records documenting key technical choices.

- [ADR-0001](./adr/0001-react-query-over-useeffect.md) -- React Query over useEffect
- [ADR-0003](./adr/0003-result-pattern-errors.md) -- Result pattern for error handling
- [ADR-0004](./adr/0004-light-mapper-services.md) -- Light mapper services
- [ADR-0005](./adr/0005-react-19-compiler.md) -- React 19 with Compiler
- [ADR-0007](./adr/0007-query-key-factory.md) -- Query key factory pattern
- [ADR-0008](./adr/0008-tanstack-router.md) -- TanStack Router
- [ADR-0009](./adr/0009-jotai-state-management.md) -- Jotai state management
