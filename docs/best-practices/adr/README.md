# Architecture Decision Records

This directory contains Architecture Decision Records (ADRs) documenting significant architectural decisions made in this project.

## What is an ADR?

An ADR is a short document that captures an important architectural decision along with its context and consequences.

## Index

| ADR | Title | Status |
|-----|-------|--------|
| [0001](./0001-react-query-over-useeffect.md) | React Query over useEffect for Data Fetching | Accepted |
| [0003](./0003-result-pattern-errors.md) | Result Pattern for Error Handling | Accepted |
| [0004](./0004-light-mapper-services.md) | Light Mapper Services | Accepted |
| [0005](./0005-react-19-compiler.md) | React 19 with Compiler | Accepted |
| [0007](./0007-query-key-factory.md) | Query Key Factory Pattern | Accepted |
| [0008](./0008-tanstack-router.md) | TanStack Router | Accepted |
| [0009](./0009-jotai-state-management.md) | Jotai State Management | Accepted |

## Creating a New ADR

1. Copy the [template](./template.md)
2. Name it `NNNN-short-title.md` where NNNN is the next number
3. Fill in all sections
4. Add to the index above
5. Submit for review

## ADR Statuses

| Status | Description |
|--------|-------------|
| **Proposed** | Under discussion |
| **Accepted** | Approved and in effect |
| **Deprecated** | No longer applicable |
| **Superseded** | Replaced by another ADR |
