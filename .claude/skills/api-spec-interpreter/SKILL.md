---
name: api-spec-interpreter
description: Parse and analyze OpenAPI/Swagger specifications to extract entities, relationships, and API patterns. Use when deriving database schemas or understanding API structure from specifications.
allowed-tools: Read, Glob, Grep
---

# API Spec Interpreter

## Purpose

Analyzes OpenAPI/Swagger specifications to extract entities, relationships, and patterns for schema design and implementation planning.

## When to Use

- Deriving database entities from API specs
- Understanding API structure before implementation
- Mapping API schemas to database types
- Identifying relationships from endpoint patterns

## Supported Formats

- OpenAPI 3.0/3.1 (YAML/JSON)
- Swagger 2.0 (YAML/JSON)

## Analysis Process

### 1. Resource Identification

Extract resources from paths:

```yaml
# API Spec
paths:
  /customers:
    get: ...
    post: ...
  /customers/{id}:
    get: ...
  /customers/{customerId}/orders:
    get: ...
```

**Entities identified:**
- Customer
- Order (child of Customer)

### 2. Schema Extraction

Map API schemas to entity properties:

```yaml
# API Spec
components:
  schemas:
    Customer:
      type: object
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
          maxLength: 100
        email:
          type: string
          format: email
        createdAt:
          type: string
          format: date-time
```

**Entity mapping:**
- `id` (uuid) → `Id` (Guid, PK)
- `name` (string, maxLength: 100) → `Name` (string, max 100)
- `email` (string, email) → `Email` (string, email format)
- `createdAt` (date-time) → `CreatedOn` (DateTime)

### 3. Relationship Detection

Identify relationships from:

1. **Nested routes**: `/customers/{customerId}/orders` → Customer 1:N Order
2. **$ref references**: `$ref: '#/components/schemas/Customer'` → Foreign key
3. **Array properties**: `orders: array of Order` → 1:N relationship

### 4. Type Mapping

| OpenAPI Type | OpenAPI Format | C# Type | SQL Type (Postgres) |
|--------------|----------------|---------|---------------------|
| string | - | string | VARCHAR |
| string | uuid | Guid | UUID |
| string | date-time | DateTime | TIMESTAMP |
| string | date | DateOnly | DATE |
| string | email | string | VARCHAR |
| integer | int32 | int | INTEGER |
| integer | int64 | long | BIGINT |
| number | float | float | REAL |
| number | double | double | DOUBLE PRECISION |
| boolean | - | bool | BOOLEAN |
| array | - | List<T> | (join table) |

## Output Format

### Entity Summary

```markdown
## Entities

### Customer
| Property | Type | Required | Constraints |
|----------|------|----------|-------------|
| Id | Guid | Yes | PK |
| Name | string | Yes | MaxLength(100) |
| Email | string | Yes | Email format |
| CreatedOn | DateTime | Yes | UTC |

### Order
| Property | Type | Required | Constraints |
|----------|------|----------|-------------|
| Id | Guid | Yes | PK |
| CustomerId | Guid | Yes | FK → Customer |
| Total | decimal | Yes | |
| Status | OrderStatus | Yes | Enum |

## Relationships

- Customer 1:N Order (via CustomerId)
```

### Schema Coverage Analysis

```markdown
## API Coverage

| Endpoint | Entities Used | Notes |
|----------|---------------|-------|
| GET /customers | Customer | List projection |
| GET /customers/{id} | Customer | Full entity |
| POST /customers | Customer | Create DTO |
| GET /customers/{id}/orders | Order, Customer | Nested route |
```

## Validation Rules

Extract validation from spec:

```yaml
properties:
  name:
    type: string
    minLength: 1
    maxLength: 100
  email:
    type: string
    format: email
  age:
    type: integer
    minimum: 0
    maximum: 150
```

**C# Validation:**
```csharp
[Required]
[StringLength(100, MinimumLength = 1)]
public string Name { get; set; }

[Required]
[EmailAddress]
public string Email { get; set; }

[Range(0, 150)]
public int Age { get; set; }
```

## Distinguishing Persisted vs Computed

| Pattern | Classification | Example |
|---------|----------------|---------|
| Direct property | Persisted | `customer.name` |
| Calculated field | Computed | `order.total` (sum of items) |
| Aggregation | Computed | `customer.orderCount` |
| Joined data | Projection | `order.customerName` |

## Checklist

- [ ] All paths analyzed for resources
- [ ] Schemas mapped to entities
- [ ] Relationships identified
- [ ] Types mapped correctly
- [ ] Validation rules extracted
- [ ] Computed vs persisted distinguished
- [ ] Missing audit columns noted
