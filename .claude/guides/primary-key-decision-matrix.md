# Primary Key Decision Matrix

Fast decision tree for choosing between INTEGER and UUID primary keys.

## Quick Decision Algorithm

```
INPUT: Table requirements
OUTPUT: PK type recommendation

1. CHECK: Is table a lookup/reference table?
   └─ YES → RETURN: INTEGER (regardless of API exposure)
   └─ NO → CONTINUE

2. CHECK: Will PK be exposed in external API?
   ├─ NO → RETURN: INTEGER
   └─ YES → CONTINUE

3. CHECK: Does API return private/sensitive data?
   ├─ NO → RETURN: INTEGER
   └─ YES → CONTINUE

4. CHECK: Is system distributed?
   ├─ YES → RETURN: UUID
   └─ NO → CONTINUE

5. CHECK: Requires data merging across DBs?
   ├─ YES → RETURN: UUID
   └─ NO → CONSIDER: Hybrid approach
```

## Comparison Matrix

| Factor | INTEGER | UUID |
|--------|---------|------|
| **Size** | 4-8 bytes | 16 bytes |
| **Performance** | ✓ High | ○ Medium |
| **Indexing** | ✓ Efficient | ○ Fragmentation risk |
| **Readability** | ✓ Easy | ✗ Difficult |
| **Security** | ✗ Predictable | ✓ Non-predictable |
| **Distributed** | ✗ Coordination needed | ✓ Independent |
| **Ordering** | ✓ Implicit | ○ Version-dependent |
| **Debugging** | ✓ Easy | ✗ Hard |

## Use Case Examples

### Use INTEGER

```yaml
- Internal employee database
- Performance-critical analytics
- Lookup tables (OrderStatus, ResourceType, Country)
- Internal-only microservices
- High-throughput logging tables
- Audit/history tables
- Configuration tables
```

### Use UUID

```yaml
- User table with PII (email, address)
- Order table in e-commerce API
- Payment transaction records
- Distributed microservices with independent ID generation
- Multi-tenant systems with data isolation
- Data sync across multiple databases
- Public-facing resource identifiers
```

### Use Hybrid

```yaml
Structure:
  - Id: INTEGER PRIMARY KEY (internal use)
  - PublicId: UUID UNIQUE NOT NULL (external API)
  - Index on PublicId for API queries

When:
  - Need both internal performance and external security
  - API returns different identifier than internal queries use
  - Gradual migration from INTEGER to UUID exposure

Example:
  API GET /orders/{publicId}
  Internal JOIN operations use Id (integer)
```

## UUID Version Selection

### When Using UUID

```
DEFAULT: UUID v4
- Random generation
- Maximum uniqueness
- Non-sequential (security)

USE v7 IF:
- Write-heavy workload
- Index performance critical
- Partial ordering desired
- Reduces fragmentation

AVOID v1:
- Contains MAC address (privacy concern)
- Use v7 instead for sequential benefits
```

## Implementation Patterns

### PostgreSQL - INTEGER

```sql
CREATE TABLE Order (
  Id SERIAL PRIMARY KEY,
  -- other columns
  CreatedOn TIMESTAMP NOT NULL,
  CreatedBy VARCHAR(255) NOT NULL
);
```

### PostgreSQL - UUID

```sql
CREATE TABLE Order (
  Id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  -- other columns
  CreatedOn TIMESTAMP NOT NULL,
  CreatedBy VARCHAR(255) NOT NULL
);
```

### PostgreSQL - Hybrid

```sql
CREATE TABLE Order (
  Id SERIAL PRIMARY KEY,
  PublicId UUID UNIQUE NOT NULL DEFAULT gen_random_uuid(),
  -- other columns
  CreatedOn TIMESTAMP NOT NULL,
  CreatedBy VARCHAR(255) NOT NULL
);

CREATE INDEX idx_order_publicid ON Order(PublicId);
```

## EF Core Conventions

### INTEGER

```csharp
public class Order
{
    public int Id { get; set; } // Auto-detected as PK
    // ...
}
```

### UUID

```csharp
public class Order
{
    public Guid Id { get; set; } // Auto-detected as PK
    // ...
}

// In DbContext OnModelCreating:
modelBuilder.Entity<Order>()
    .Property(o => o.Id)
    .HasDefaultValueSql("gen_random_uuid()");
```

### Hybrid

```csharp
public class Order
{
    public int Id { get; set; }
    public Guid PublicId { get; set; }
    // ...
}

// In DbContext OnModelCreating:
modelBuilder.Entity<Order>()
    .Property(o => o.PublicId)
    .HasDefaultValueSql("gen_random_uuid()");

modelBuilder.Entity<Order>()
    .HasIndex(o => o.PublicId)
    .IsUnique();
```

## Security Considerations

### When INTEGER is Exposed

```
RISK: ID enumeration attacks
EXAMPLE: /api/orders/1, /api/orders/2, ...
ALLOWS: Attackers to guess valid IDs

MITIGATIONS IF MUST USE:
1. Implement authorization checks (user can only access their orders)
2. Add random token alongside ID
3. Rate limit API endpoints
4. Consider hybrid approach instead
```

### When UUID is Used

```
BENEFIT: Non-predictable identifiers
EXAMPLE: /api/orders/550e8400-e29b-41d4-a716-446655440000
PREVENTS: ID enumeration
COST: Larger URLs, harder debugging

STILL REQUIRED:
- Authorization checks (UUID ≠ security)
- Proper authentication
- Input validation
```

## Performance Implications

### INTEGER Advantages

```
INDEX SIZE: Smaller (4-8 bytes)
JOIN PERFORMANCE: Faster
FOREIGN KEY OVERHEAD: Lower
INSERTION: Sequential (no fragmentation)
QUERY SPEED: Faster comparisons
```

### UUID Overhead

```
INDEX SIZE: Larger (16 bytes)
JOIN PERFORMANCE: Slower
FOREIGN KEY OVERHEAD: Higher
INSERTION: Random (potential fragmentation with v4)
QUERY SPEED: Slower comparisons

OPTIMIZATION:
- Use UUID v7 for sequential characteristics
- Ensure proper indexing
- Monitor fragmentation
- Consider table partitioning for large tables
```

## Migration Strategies

### INTEGER → UUID

```
STRATEGY 1: Hybrid approach
1. Add PublicId UUID column
2. Generate UUIDs for existing rows
3. Update API to use PublicId
4. Keep INTEGER PK internally

STRATEGY 2: Full migration
1. Add new UUID column
2. Backfill values
3. Update all FKs
4. Switch PK
⚠️  High risk, requires downtime
```

### UUID → INTEGER

```
⚠️  NOT RECOMMENDED
- Breaks external references
- Requires API versioning
- Data migration complexity

IF REQUIRED:
1. Create new API version
2. Add INTEGER column
3. Maintain both during transition
4. Deprecate UUID version gradually
```

## Decision Flowchart Summary

```
                        START
                          |
                    [Lookup Table?]
                    /           \
                  YES            NO
                   |              |
               INTEGER      [Exposed in API?]
                            /            \
                          NO             YES
                           |              |
                       INTEGER      [Sensitive Data?]
                                    /            \
                                  NO             YES
                                   |              |
                               INTEGER      [Distributed?]
                                            /          \
                                          NO           YES
                                           |            |
                                      HYBRID/UUID      UUID
```

## Validation Checklist

```
BEFORE FINALIZING PK CHOICE:
☐ Identified table type (entity vs lookup)
☐ Determined API exposure level
☐ Assessed data sensitivity
☐ Evaluated performance requirements
☐ Considered future scaling needs
☐ Checked for distributed system requirements
☐ Documented decision rationale
☐ Reviewed with team if uncertain
```
