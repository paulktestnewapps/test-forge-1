---
allowed-tools: Read, Edit, Write, Glob, Grep
argument-hint: [path-to-ddl-file]
description: Generate EF Core entities and DbContext from PostgreSQL DDL
---

## Context

Parse the PostgreSQL DDL script at: $ARGUMENTS

## Task

Generate EF Core code (entities, DbContext, and configurations) from the DDL.

### 1. Parse DDL
Extract from CREATE TABLE statements:
- Table names → Entity class names
- Columns → Properties
- Data types → C# types
- Constraints → Fluent API configurations
- Foreign keys → Relationships

### 2. Type Mapping

| PostgreSQL Type | C# Type |
|-----------------|---------|
| UUID | Guid |
| VARCHAR(n) | string |
| TEXT | string |
| INTEGER | int |
| BIGINT | long |
| SMALLINT | short |
| NUMERIC(p,s) | decimal |
| REAL | float |
| DOUBLE PRECISION | double |
| BOOLEAN | bool |
| TIMESTAMPTZ | DateTime |
| TIMESTAMP | DateTime |
| DATE | DateOnly |
| BYTEA | byte[] |

### 3. Naming Conversion
- Tables: `snake_case` plural → `PascalCase` singular (class)
- Columns: `snake_case` → `PascalCase`
- DbSet: `PascalCase` plural

Examples:
- `users` → `User` class, `Users` DbSet
- `order_items` → `OrderItem` class, `OrderItems` DbSet
- `user_id` → `UserId` property

### 4. Generate Files

#### Entity Classes
```csharp
// Entities/User.cs
namespace MyProject.Entities;

public class User
{
    public Guid Id { get; set; }
    public string Email { get; set; } = null!;
    public string Name { get; set; } = null!;
    public DateTime CreatedOn { get; set; }
    public string CreatedBy { get; set; } = null!;
    public DateTime? UpdatedOn { get; set; }
    public string? UpdatedBy { get; set; }
    public bool IsActive { get; set; }

    // Navigation properties
    public ICollection<Order> Orders { get; set; } = new List<Order>();
}
```

#### Entity Configurations
```csharp
// Configurations/UserConfiguration.cs
namespace MyProject.Configurations;

public class UserConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.ToTable("users");
        builder.HasKey(e => e.Id);

        builder.Property(e => e.Id)
            .HasDefaultValueSql("gen_random_uuid()");

        builder.Property(e => e.Email)
            .HasMaxLength(255)
            .IsRequired();

        // ... other configurations

        builder.HasIndex(e => e.Email)
            .IsUnique();
    }
}
```

#### DbContext
```csharp
// Data/AppDbContext.cs
namespace MyProject.Data;

public class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
{
    public DbSet<User> Users => Set<User>();
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
    }
}
```

### 5. Relationship Mapping

From FK constraints, generate:
- FK property on child entity
- Navigation property on child (required)
- Collection navigation on parent
- Fluent API configuration

```csharp
// Order entity
public Guid UserId { get; set; }
public User User { get; set; } = null!;

// User entity
public ICollection<Order> Orders { get; set; } = new List<Order>();

// OrderConfiguration
builder.HasOne(e => e.User)
    .WithMany(e => e.Orders)
    .HasForeignKey(e => e.UserId);
```

### File Structure
```
src/
├── Entities/
│   ├── User.cs
│   ├── Order.cs
│   └── OrderItem.cs
├── Configurations/
│   ├── UserConfiguration.cs
│   ├── OrderConfiguration.cs
│   └── OrderItemConfiguration.cs
└── Data/
    └── AppDbContext.cs
```

### Conventions
- Use primary constructors for DbContext
- Use `IEntityTypeConfiguration<T>` for all configurations
- Initialize collections to empty lists
- Use `null!` for required navigation properties
- Use nullable reference types throughout
