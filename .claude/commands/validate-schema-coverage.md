---
allowed-tools: Read, Glob, Grep
argument-hint: [api-spec-path] [entities-path]
description: Validate that database schema covers all API requirements
---

## Context

- API Specification: $1
- Entity/Schema files: $2

## Task

Validate that the database schema fully supports the API specification:

### 1. Load Both Sources
- Parse OpenAPI spec for all endpoints and schemas
- Load entity definitions from codebase

### 2. Coverage Analysis

For each API endpoint, verify:
- Required entity exists
- All request fields can be persisted
- All response fields can be retrieved
- Relationships support nested routes

### 3. Generate Report

```yaml
coverage_summary:
  total_endpoints: N
  fully_covered: N
  partially_covered: N
  not_covered: N

fully_covered:
  - endpoint: GET /users
    entity: User
    status: ✓ All fields mapped

partially_covered:
  - endpoint: GET /orders/{id}
    entity: Order
    missing_fields:
      - customerName: "Needs Customer join"
    recommendation: "Add navigation property"

not_covered:
  - endpoint: GET /reports/dashboard
    issue: "No entity for aggregated data"
    recommendation: "This is a projection query, not entity-backed"

schema_issues:
  - entity: Order
    issue: "Missing index on CustomerId"
    impact: "Slow lookups for customer orders"

relationship_gaps:
  - api_pattern: GET /users/{id}/favorites
    expected: User → Favorite (1:N)
    actual: No relationship defined
```

### 4. Recommendations
- Missing entities to create
- Relationships to add
- Indexes to improve performance
- Fields that should be projections (not stored)
