# Commit Best Practices

Quick reference for writing clean, meaningful git commits.

## Prefix Tags

Every commit message MUST start with a prefix tag:

```
APPLY:
- feat:     New feature or capability
- fix:      Bug fix
- refactor: Code restructure without behavior change
- test:     Adding or updating tests
- docs:     Documentation only changes
- chore:    Build process, tooling, config (no production code)
- style:    Formatting, whitespace (no logic change)
- perf:     Performance improvement
- ci:       CI/CD pipeline changes
- revert:   Reverting a previous commit

EXAMPLES:
✓ feat: add customer search endpoint
✓ fix: null reference on empty cart
✓ refactor: extract order validation to service
✓ test: add unit tests for ProductService.GetById
✗ "fixed stuff"
✗ "updates"
✗ "wip"
```

## Message Rules

```
APPLY:
- Keep subject line under 72 characters
- Use imperative mood: "add", "fix", "update" (not "added", "fixed")
- Be specific — describe WHAT changed and WHY (not HOW)
- No period at end of subject line

EXAMPLES:
✓ feat: add pagination to product list endpoint
✓ fix: prevent duplicate order submission on retry
✗ feat: made some changes to the product controller and also fixed a bug
✗ fix: fixed it
```

## Commit Size Rules

```
APPLY:
- One logical change per commit
- If you need "and" to describe it, split it into two commits
- Avoid batching unrelated changes
- Prefer small, frequent commits over large infrequent ones

EXAMPLES:
✓ feat: add CreateOrder endpoint
✓ test: add unit tests for OrderService.Create
✗ feat: add CreateOrder endpoint and fix existing tests and update README
```

## Grouping Rule

```
DO NOT group unrelated changes in a single commit:
✗ fix: resolve null ref + update nuget packages + rename variable
✓ fix: resolve null reference in OrderRepository.GetById
✓ chore: update NuGet packages to latest stable
✓ refactor: rename customerId to externalCustomerId
```

## Full Example

```
feat: add product search by category

Supports filtering by one or more category IDs via query params.
Returns paginated results ordered by relevance score.
```
