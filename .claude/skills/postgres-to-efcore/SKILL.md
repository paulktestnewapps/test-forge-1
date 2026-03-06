---
name: postgres-to-efcore
description: Generate EF Core entities, DbContext, and configurations from PostgreSQL DDL scripts. Use when reverse-engineering existing database schemas to .NET code.
allowed-tools: Read, Edit, Write, Glob, Grep
---

# Postgres to EF Core

## Purpose

Generates Entity Framework Core entities, DbContext, and Fluent API configurations from PostgreSQL DDL scripts.

## When to Use

- Reverse-engineering existing PostgreSQL databases
- Converting DDL to EF Core code-first models
- Migrating database schemas to .NET projects

## Type Mapping

| PostgreSQL Type | C# Type | EF Core Configuration |
|-----------------|---------|----------------------|
| SERIAL | int | `.ValueGeneratedOnAdd()` |
| BIGSERIAL | long | `.ValueGeneratedOnAdd()` |
| UUID | Guid | Default |
| VARCHAR(n) | string | `.HasMaxLength(n)` |
| TEXT | string | Default |
| INTEGER | int | Default |
| BIGINT | long | Default |
| NUMERIC(p,s) | decimal | `.HasPrecision(p, s)` |
| BOOLEAN | bool | Default |
| TIMESTAMP | DateTime | Default |
| TIMESTAMPTZ | DateTimeOffset | Default |
| DATE | DateOnly | Default |
| TIME | TimeOnly | Default |
| JSONB | string/JsonDocument | `.HasColumnType("jsonb")` |
| BYTEA | byte[] | Default |

## DDL to Entity Conversion

### Input DDL

```sql
CREATE TABLE customers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    phone VARCHAR(20),
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_on TIMESTAMP NOT NULL DEFAULT (NOW() AT TIME ZONE 'UTC'),
    created_by VARCHAR(255) NOT NULL,
    updated_on TIMESTAMP,
    updated_by VARCHAR(255)
);

CREATE INDEX idx_customers_email ON customers(email);
CREATE INDEX idx_customers_is_active ON customers(is_active) WHERE is_active = TRUE;
```

### Output Entity

```csharp
namespace MyApi.Domain.Entities;

public class Customer
{
    public Guid Id { get; set; }
    public string Name { get; set; } = null!;
    public string Email { get; set; } = null!;
    public string? Phone { get; set; }
    public bool IsActive { get; set; }
    public DateTime CreatedOn { get; set; }
    public string CreatedBy { get; set; } = null!;
    public DateTime? UpdatedOn { get; set; }
    public string? UpdatedBy { get; set; }

    // Navigation properties
    public ICollection<Order> Orders { get; set; } = new List<Order>();
}
```

### Output Configuration

```csharp
namespace MyApi.Infrastructure.Persistence.Configurations;

public class CustomerConfiguration : IEntityTypeConfiguration<Customer>
{
    public void Configure(EntityTypeBuilder<Customer> builder)
    {
        builder.ToTable("customers");

        builder.HasKey(e => e.Id);

        builder.Property(e => e.Id)
            .HasDefaultValueSql("gen_random_uuid()");

        builder.Property(e => e.Name)
            .HasMaxLength(100)
            .IsRequired();

        builder.Property(e => e.Email)
            .HasMaxLength(255)
            .IsRequired();

        builder.HasIndex(e => e.Email)
            .IsUnique();

        builder.Property(e => e.Phone)
            .HasMaxLength(20);

        builder.Property(e => e.IsActive)
            .HasDefaultValue(true);

        builder.Property(e => e.CreatedOn)
            .HasDefaultValueSql("NOW() AT TIME ZONE 'UTC'");

        builder.Property(e => e.CreatedBy)
            .HasMaxLength(255)
            .IsRequired();

        builder.Property(e => e.UpdatedBy)
            .HasMaxLength(255);

        // Filtered index
        builder.HasIndex(e => e.IsActive)
            .HasFilter("is_active = TRUE");

        // Relationships configured in child entities
    }
}
```

## Relationship Mapping

### One-to-Many (Foreign Key)

```sql
-- DDL
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    customer_id UUID NOT NULL REFERENCES customers(id),
    ...
);
```

```csharp
// Order entity
public Guid CustomerId { get; set; }
public Customer Customer { get; set; } = null!;

// Order configuration
builder.HasOne(e => e.Customer)
    .WithMany(c => c.Orders)
    .HasForeignKey(e => e.CustomerId)
    .OnDelete(DeleteBehavior.Cascade);
```

### Many-to-Many (Join Table)

```sql
-- DDL
CREATE TABLE product_categories (
    product_id UUID REFERENCES products(id),
    category_id UUID REFERENCES categories(id),
    PRIMARY KEY (product_id, category_id)
);
```

```csharp
// Product entity
public ICollection<Category> Categories { get; set; } = new List<Category>();

// Product configuration (EF Core 5+)
builder.HasMany(e => e.Categories)
    .WithMany(c => c.Products)
    .UsingEntity(j => j.ToTable("product_categories"));
```

## Naming Convention Conversion

| PostgreSQL (snake_case) | C# (PascalCase) |
|-------------------------|-----------------|
| customer_id | CustomerId |
| created_on | CreatedOn |
| is_active | IsActive |
| order_line_items | OrderLineItems |

Use `.HasColumnName()` to preserve database naming:

```csharp
builder.Property(e => e.CustomerId)
    .HasColumnName("customer_id");
```

## DbContext Template

```csharp
namespace MyApi.Infrastructure.Persistence;

public class ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
    : DbContext(options)
{
    public DbSet<Customer> Customers => Set<Customer>();
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(ApplicationDbContext).Assembly);

        // Use snake_case naming convention for PostgreSQL
        foreach (var entity in modelBuilder.Model.GetEntityTypes())
        {
            entity.SetTableName(entity.GetTableName()?.ToSnakeCase());

            foreach (var property in entity.GetProperties())
            {
                property.SetColumnName(property.GetColumnName().ToSnakeCase());
            }
        }
    }
}
```

## Audit Column Base Class

```csharp
public abstract class AuditableEntity
{
    public DateTime CreatedOn { get; set; }
    public string CreatedBy { get; set; } = null!;
    public DateTime? UpdatedOn { get; set; }
    public string? UpdatedBy { get; set; }
}

public class Customer : AuditableEntity
{
    public Guid Id { get; set; }
    public string Name { get; set; } = null!;
    // ...
}
```

## Checklist

- [ ] All tables converted to entities
- [ ] Primary keys configured
- [ ] Foreign keys mapped to navigation properties
- [ ] Indexes created
- [ ] Default values preserved
- [ ] Nullable columns marked nullable in C#
- [ ] Max lengths specified
- [ ] Audit columns using base class
- [ ] Column names mapped (snake_case → PascalCase)
- [ ] DbContext includes all DbSets
