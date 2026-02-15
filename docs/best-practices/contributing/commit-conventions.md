# Commit Conventions

Git commit message format and best practices.

## Conventional Commits

This project follows the [Conventional Commits](https://www.conventionalcommits.org/) specification.

### Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Types

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature | `feat(auth): add magic link login` |
| `fix` | Bug fix | `fix(search): correct filter reset behavior` |
| `docs` | Documentation | `docs: update API documentation` |
| `style` | Formatting (no code change) | `style: format with prettier` |
| `refactor` | Code refactoring | `refactor(hooks): extract query logic` |
| `test` | Adding/updating tests | `test(profile): add service tests` |
| `chore` | Maintenance tasks | `chore: update dependencies` |
| `perf` | Performance improvement | `perf(search): optimize filter query` |

### Scopes (Optional)

Common scopes:

| Scope | Description |
|-------|-------------|
| `auth` | Authentication related |
| `ui` | UI components |
| `api` | API/service layer |
| `db` | Database migrations |
| `hooks` | React hooks |
| `config` | Configuration changes |
| `deps` | Dependency updates |

---

## Examples

### Feature

```
feat(auth): add magic link login

- Add MagicLinkForm component
- Integrate with Supabase Auth
- Add email validation
```

### Bug Fix

```
fix(auth): handle session expiry correctly

Session was not being refreshed on token expiry.
Now properly refreshes token before API calls.

Closes #123
```

### Refactoring

```
refactor(services): use Result pattern for error handling

Replace throw statements with Result<T, E> pattern
for better type-safe error handling.
```

### Breaking Change

```
feat(api)!: change profile response structure

BREAKING CHANGE: Profile response now uses camelCase keys.
Update all consumers to use new key names.
```

---

## Best Practices

### Description

- Use imperative mood: "add feature" not "added feature"
- Keep under 72 characters
- Don't end with a period
- Be specific about what changed

### Body

- Explain **what** and **why**, not **how**
- Wrap at 72 characters
- Use bullet points for multiple changes

### Footer

- Reference issues: `Closes #123`, `Fixes #456`
- Note breaking changes: `BREAKING CHANGE: description`

---

## Git Workflow

### Branch Naming

```
feature/short-description
fix/issue-description
refactor/what-is-changing
```

### Commit Frequency

- Commit logical units of work
- Each commit should pass all tests
- Don't commit broken code

---

## Pre-commit Validation

Before committing, always run:

```bash
pnpm typecheck && pnpm lint && pnpm test:run
```

This ensures:
- TypeScript compiles without errors
- Code passes linting rules
- All tests pass
