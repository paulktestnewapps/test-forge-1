# Database Design Rules

Quick reference for database schema design decisions.

## Table Naming Rules

```
APPLY:
- Concise and descriptive (2-3 words ideal)
- No "Bunzl" prefix by default (only if duplicate exists in same DB)
- Use casing consistent with existing tables or DB provider conventions

EXAMPLES:
✓ Order, OrderItem, CustomerAddress
✗ BunzlOrder, OrderTbl, tbl_orders
```

## Required Audit Columns

Every table MUST include these columns:

| Column | Type | Nullable | Set When | Value |
|--------|------|----------|----------|-------|
| `CreatedOn` | datetime (UTC) | NO | Insert only | Current UTC time |
| `CreatedBy` | string | NO | Insert only | User identifier |
| `UpdatedOn` | datetime (UTC) | YES | Update only | Current UTC time |
| `UpdatedBy` | string | YES | Update only | User identifier |

## Soft Delete Column

```
WHEN: Table exposed by API with DELETE operations
ADD: IsActive column (bool, not-null)
DEFAULT: true
ON DELETE: Set to false instead of physical deletion
```

## Column Type Rules

```
AVOID: DateOnly types
USE: DateTime instead
REASON: Broader compatibility and consistency
```

## Primary Key Selection

### Decision Tree

```
START
  ├─ Is PK exposed in API that returns private/sensitive data?
  │  ├─ YES → Is it a lookup/reference table (e.g., OrderStatus)?
  │  │  ├─ YES → Use INTEGER
  │  │  └─ NO → Use UUID
  │  └─ NO → Is system distributed requiring independent ID generation?
  │     ├─ YES → Use UUID
  │     └─ NO → Use INTEGER (default)
```

### INTEGER (Default)

```
USE WHEN:
- Internal systems
- Performance-critical applications
- Lookup/reference tables (OrderStatus, ResourceType)
- Tables not exposing sensitive data via API

ADVANTAGES:
- High performance (4-8 bytes)
- Easy debugging
- Implicit ordering
```

### UUID

```
USE WHEN:
- PK exposed via API with private data (User, Order)
- Distributed systems
- Data merging across multiple databases

NEVER USE FOR:
- Lookup/reference tables
- Internal-only tables

ADVANTAGES:
- Non-predictable (security)
- Globally unique
- Independent generation

DISADVANTAGES:
- Larger size (16 bytes)
- Harder to debug
- Potential index fragmentation
```

### Hybrid Approach

```
WHEN: Need both security and performance
IMPLEMENTATION:
1. Use INTEGER as PK for internal operations
2. Add separate UUID column for API exposure
3. Index the UUID column if frequently queried

EXAMPLE:
  CREATE TABLE Order (
    Id INT PRIMARY KEY AUTO_INCREMENT,
    PublicId UUID UNIQUE NOT NULL,
    ...
  );

  API returns PublicId, never Id
```

## Normalization Rules

### Many-to-Many Relationships

```
PATTERN: Create join table
AVOID: Comma-separated values in columns

EXAMPLE:
✗ BAD:
  Product.CategoryIds = "1,3,7"

✓ GOOD:
  ProductCategory table
    ProductId (FK)
    CategoryId (FK)
```

### Lookup Tables

```
PATTERN: Separate table for static values
AVOID: Hard-coded strings in value columns

EXAMPLE:
✗ BAD:
  Order.Status = "pending" | "paid" | "shipped"

✓ GOOD:
  OrderStatus table (Id, Value)
  Order.StatusId references OrderStatus.Id
```

## Scale Considerations

### Column Count Warning

```
IF: Table has > 10 value columns
THEN: Consider optimization
OPTIONS:
  - Split into related tables
  - Vertical partitioning
  - Reconsider design
```

### Record Volume Analysis

```
BEFORE FINALIZING:
1. Estimate records per operation
2. Calculate total eventual records
3. Consider query patterns
4. Plan indexing strategy
5. Assess impact on related tables
```

## Quick Validation Checklist

```
TABLE DESIGN COMPLETE WHEN:
☐ Table name is concise (2-3 words)
☐ All 4 audit columns present (CreatedOn, CreatedBy, UpdatedOn, UpdatedBy)
☐ IsActive column added if DELETE operations exist
☐ PK type chosen based on decision tree (INTEGER default, UUID if exposed)
☐ No comma-separated values (normalized with join tables)
☐ Lookup values in separate tables with FKs
☐ Column count ≤ 10 (or justification documented)
☐ DateTime used instead of DateOnly
☐ Indexes planned for FK and query columns
```

## Anti-Patterns to Avoid

```
✗ Comma-separated lists in columns
✗ Hard-coded status/type strings
✗ Missing audit columns
✗ DateOnly column types
✗ UUID for lookup tables
✗ INTEGER PKs exposed in sensitive APIs
✗ Nullable CreatedOn/CreatedBy
✗ Tables with > 15 columns without justification
```
