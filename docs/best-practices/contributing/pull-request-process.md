# Pull Request Process

Guidelines for creating and reviewing pull requests.

## Creating a PR

### Before Opening

1. **Run validation**:
   ```bash
   pnpm typecheck && pnpm lint && pnpm test:run
   ```

2. **Update documentation** if adding new features

3. **Self-review** your changes

### PR Title

Follow conventional commit format:

```
feat(scope): short description
fix(scope): short description
refactor(scope): short description
```

### PR Description

Include:

1. **What** - Brief description of changes
2. **Why** - Motivation for the change
3. **How** - High-level implementation approach
4. **Testing** - How to verify the changes

#### Template

```markdown
## Summary

Brief description of what this PR does.

## Changes

- Change 1
- Change 2
- Change 3

## Testing

- [ ] Unit tests pass
- [ ] Manual testing done
- [ ] Edge cases covered

## Screenshots (if UI changes)

[Add screenshots here]
```

---

## PR Size Guidelines

### Ideal PR Size

| Lines Changed | Size | Recommendation |
|---------------|------|----------------|
| < 200 | Small | ✅ Ideal |
| 200-400 | Medium | ✅ Good |
| 400-800 | Large | ⚠️ Consider splitting |
| > 800 | Very Large | ❌ Split into smaller PRs |

### Splitting Large PRs

If a feature is large, break it into:

1. **Infrastructure PR** - Types, schemas, services
2. **Implementation PR** - Components, hooks
3. **Polish PR** - Tests, edge cases, documentation

---

## Review Process

### As a Reviewer

1. **Understand the context** - Read PR description fully
2. **Check functionality** - Does it work as intended?
3. **Review code quality** - Follows project patterns?
4. **Verify tests** - Adequate test coverage?

### Review Checklist

- [ ] Code follows project conventions
- [ ] No `any` types or `@ts-ignore`
- [ ] Tests included for new functionality
- [ ] No console.log statements
- [ ] Import alias used (`@/`)
- [ ] Documentation updated if needed

### Feedback Types

| Prefix | Meaning |
|--------|---------|
| `nit:` | Minor, non-blocking suggestion |
| `suggestion:` | Improvement idea, not required |
| `question:` | Seeking clarification |
| (none) | Required change |

---

## Merging

### Requirements

1. All CI checks pass
2. At least one approval (if required)
3. No unresolved conversations
4. Branch is up to date with main

### Merge Strategy

Use **Squash and Merge** for:
- Clean commit history
- Easier reverting if needed
- Single commit per feature

---

## After Merging

1. **Delete feature branch** (automatic on GitHub)
2. **Verify deployment** if auto-deploying
3. **Close related issues** if not auto-linked

---

## Common Issues

### CI Failures

| Issue | Solution |
|-------|----------|
| Type errors | Run `pnpm typecheck` locally |
| Lint errors | Run `pnpm lint:fix` |
| Test failures | Run `pnpm test:run` locally |
| Build failures | Run `pnpm build` locally |

### Merge Conflicts

1. Update your branch:
   ```bash
   git checkout main
   git pull origin main
   git checkout your-branch
   git merge main
   ```

2. Resolve conflicts in IDE

3. Run validation again:
   ```bash
   pnpm typecheck && pnpm lint && pnpm test:run
   ```

4. Push updated branch
