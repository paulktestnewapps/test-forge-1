# Code Examples Quick Reference

Fast lookup guide for Claude Code referencing best practice examples from real production .NET projects.

## When to Use This Guide

```
USE CASE: Need to generate or reference production-quality code
TRIGGER: Creating services, repositories, controllers, or project structure
ACTION: Reference appropriate example pattern below
```

## Example Categories

### 1. Base Service Pattern
**Location**: [docs/examples/BASE-ENTITY-SERVICE-EXAMPLE.md](../../docs/examples/BASE-ENTITY-SERVICE-EXAMPLE.md)

**Use When**:
- Creating new service classes
- Implementing CRUD operations
- Need to reduce code repetition across services
- Working with auditable entities or soft deletes

**Key Pattern**:
```csharp
// Abstract base with common CRUD operations
public abstract class BaseEntityService<TEntity, TKey> where TEntity : class, IEntity<TKey>
{
    // Common operations: GetById, Create, Update, Delete
    // Automatic audit field handling
    // Soft delete support
    // Protected GetBaseQuery() for derived classes
}

// Concrete service inherits and adds specific methods
public class CategoryService : BaseEntityService<Category, int>
{
    // Category-specific methods only
}
```

**Key Features**:
- Generic base class reduces boilerplate
- Automatic audit column population (CreatedOn, CreatedBy, UpdatedOn, UpdatedBy)
- Soft delete support via ISoftDeletable interface
- Protected GetBaseQuery() applies common filters (e.g., !IsDeleted)

---

### 2. PATCH Operations
**Location**: [docs/examples/PATCH-OPERATION-EXAMPLE.md](../../docs/examples/PATCH-OPERATION-EXAMPLE.md)

**Use When**:
- Implementing HTTP PATCH endpoints
- Need to update only specific fields
- Avoiding full entity replacement

**Key Pattern**:
```csharp
// Service method
public async Task<Product> PatchProductAsync(int id, ProductPatchDto patchDto)
{
    var product = await _context.Products.FindAsync(id);
    if (product == null) throw new NotFoundException();

    var patchRequest = new PatchRequest<Product>()
        .SetIfNotNull(p => p.Name, patchDto.Name)
        .SetIfHasValue(p => p.Price, patchDto.Price)
        .SetIfNotNull(p => p.Description, patchDto.Description);

    patchRequest.ApplyTo(product);
    // Set audit fields, save
}
```

**Key Features**:
- Expression trees for type-safe property updates
- `SetIfNotNull` for reference types
- `SetIfHasValue` for nullable value types
- Only modified fields are updated
- Fluent API for readable chaining

**Implementation Note**:
- Use PatchRequest<T> helper class (see example for full code)
- Handles both nullable and non-nullable properties correctly

---

### 3. Query Extensions
**Location**: [docs/examples/QUERY-EXTENSION-EXAMPLE.md](../../docs/examples/QUERY-EXTENSION-EXAMPLE.md)

**Use When**:
- Building composable query filters
- Need reusable query logic across services
- Want to keep repository/service methods clean

**Key Pattern**:
```csharp
public static class ProductQueryExtensions
{
    public static IQueryable<Product> ActiveOnly(this IQueryable<Product> query)
        => query.Where(p => p.IsActive && !p.IsDeleted);

    public static IQueryable<Product> InCategory(this IQueryable<Product> query, int categoryId)
        => query.Where(p => p.CategoryId == categoryId);

    public static IQueryable<Product> WithPriceRange(this IQueryable<Product> query, decimal? min, decimal? max)
    {
        if (min.HasValue) query = query.Where(p => p.Price >= min.Value);
        if (max.HasValue) query = query.Where(p => p.Price <= max.Value);
        return query;
    }

    public static IQueryable<Product> IncludeBasicData(this IQueryable<Product> query)
        => query.Include(p => p.Category).Include(p => p.Supplier);
}

// Usage
var products = await Context.Products
    .ActiveOnly()
    .InCategory(categoryId)
    .WithPriceRange(10m, 100m)
    .IncludeBasicData()
    .ToListAsync();
```

**Key Features**:
- Composable, chainable filters
- Reusable across services
- Include/eager loading patterns
- Conditional filtering support

**Naming Convention**:
- Boolean filters: `ActiveOnly()`, `InStock()`
- Conditional filters: `WithPriceRange()`, `WithRatingAbove()`
- Eager loading: `IncludeBasicData()`, `IncludeFullData()`

---

### 4. Project Structure (Simple)
**Location**: [docs/examples/PROJECT-STRUCTURE.md](../../docs/examples/PROJECT-STRUCTURE.md)

**Use When**:
- Starting a new .NET API project
- Project has simple/moderate complexity
- Single API with tests

**Structure**:
```
/
├── src/
│   ├── PrdPal.Api/              # Main API project
│   │   ├── PrdPal.Api.csproj
│   │   ├── Program.cs
│   │   ├── appsettings.json
│   │   └── Properties/
│   │       └── launchSettings.json
│   └── PrdPal.Api.Tests/        # Test project
│       └── PrdPal.Api.Tests.csproj
├── infrastructure/              # Bicep/IaC files
│   ├── main.bicep
│   └── parameters.json
├── .devops/                     # CI/CD
│   └── pipeline.yaml
├── Dockerfile
├── nuget.config
└── README.md
```

**Key Principles**:
- Keep all source under `src/`
- Separate infrastructure and pipelines
- Configuration files at root
- Exclude secrets (parameters.json) from git

---

### 5. Project Structure (Clean Architecture)
**Location**: [docs/examples/PROJECT-STRUCTURE-FOR-LARGER-PROJECT.md](../../docs/examples/PROJECT-STRUCTURE-FOR-LARGER-PROJECT.md)

**Use When**:
- Large or complex projects
- Need clear separation of concerns
- Using CQRS or Clean Architecture
- Multiple teams working on project

**Structure**:
```
OrderHistory.sln
├── src/
│   ├── OrderHistory.Api/         # Presentation
│   ├── OrderHistory.Application/ # Business logic (Commands/Queries)
│   ├── OrderHistory.Domain/      # Core entities, interfaces
│   └── OrderHistory.Infrastructure/ # Data access, external services
├── tests/
│   └── OrderHistory.Test/
├── scripts/
│   └── sql/
├── docs/
│   └── api-spec.json
├── .devops/
├── Dockerfile
└── nuget.config
```

**Layer Dependencies** (CRITICAL):
```
Api → Infrastructure → Application → Domain
Test → All layers

Domain: NO dependencies (core business objects)
Application: depends on Domain only
Infrastructure: depends on Application (implements interfaces)
Api: depends on Infrastructure + Application
```

**Key Folders**:
- `Domain/Entities/` - Core business objects
- `Domain/Repositories/` - Repository interfaces (NOT implementations)
- `Domain/Specifications/` - Query specifications
- `Application/Commands/` - Write operations (CQS)
- `Application/Queries/` - Read operations (CQS)
- `Infrastructure/Data/` - DbContext, migrations
- `Infrastructure/Repositories/` - Repository implementations
- `Infrastructure/Bicep/` - IaC templates

---

### 6. Program.cs / Startup Configuration
**Location**: [docs/examples/DOTNET-STARTUP-FILE-EXAMPLE.md](../../docs/examples/DOTNET-STARTUP-FILE-EXAMPLE.md)

**Use When**:
- Configuring DI container
- Setting up middleware pipeline
- Configuring authentication/authorization
- Registering services

**Configuration Order** (CRITICAL):
```csharp
var builder = WebApplication.CreateBuilder(args);

// 1. CONFIGURATION
if (!builder.Environment.IsDevelopment())
{
    builder.Configuration.AddAzureAppConfiguration(/* ... */);
}

// 2. OPTIONS BINDING
builder.Services.Configure<DatabaseOptions>(builder.Configuration.GetSection("PrdPalApi:Database"));

// 3. DBCONTEXT
builder.Services.AddDbContext<PrdPalDbContext>(/* ... */);

// 4. CONTROLLERS + JSON OPTIONS
builder.Services.AddControllers()
    .AddJsonOptions(options => {
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
        options.JsonSerializerOptions.DefaultIgnoreCondition =
            JsonIgnoreCondition.WhenWritingDefault | JsonIgnoreCondition.WhenWritingNull;
    });

// 5. HTTP CLIENTS
builder.Services.AddHttpClient<IService, Service>();

// 6. SWAGGER
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(/* ... */);

// 7. EXCEPTION HANDLERS (order matters!)
builder.Services.AddExceptionHandler<AlreadyExistsExceptionHandler>();
builder.Services.AddExceptionHandler<NotFoundExceptionHandler>();
builder.Services.AddExceptionHandler<GlobalExceptionHandler>(); // Last = catch-all

// 8. AUTHENTICATION + AUTHORIZATION
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));
builder.Services.AddAuthorization();

// 9. CUSTOM SERVICES (Singleton → Scoped → Transient)
builder.Services.AddSingleton<IBlobContainerAdapter>(/* ... */);
builder.Services.AddScoped<IRepository, Repository>();
builder.Services.AddScoped<IService, Service>();
builder.Services.AddAutoMapper(typeof(Program).Assembly);

var app = builder.Build();

// MIDDLEWARE PIPELINE ORDER (CRITICAL):
app.UseSwagger();
app.UseSwaggerUI(/* ... */);
app.UseHttpsRedirection();
app.UseCors(corsPolicy);           // Before authentication!
app.UseAuthentication();           // Before authorization!
app.UseAuthorization();
app.UseExceptionHandler();         // Near the end
app.MapControllers();
app.Run();
```

**Key Patterns**:
- Azure App Configuration only in non-dev environments
- Options pattern for configuration binding
- CORS policy only in Development
- Exception handlers in order: specific → general
- Middleware pipeline order is critical

**Authentication Configuration**:
```csharp
builder.Services.Configure<JwtBearerOptions>(JwtBearerDefaults.AuthenticationScheme, options =>
{
    options.Events = new JwtBearerEvents
    {
        OnTokenValidated = context =>
            NotAuthenticatedHandler.CheckClaims(context, ["preferred_username"]),
        OnChallenge = context =>
            NotAuthenticatedHandler.HandleResponse(context)
    };
});
```

**Swagger Security**:
```csharp
builder.Services.AddSwaggerGen(options => {
    options.AddSecurityDefinition("http", new OpenApiSecurityScheme
    {
        Type = SecuritySchemeType.Http,
        Scheme = "bearer",
        BearerFormat = "JWT",
        In = ParameterLocation.Header,
    });
    options.AddSecurityRequirement(/* ... */);
});
```

---

### 7. NuGet Configuration
**Location**: [docs/examples/NUGET-CONFIG-EXAMPLE.md](../../docs/examples/NUGET-CONFIG-EXAMPLE.md)

**Use When**:
- Using private NuGet feeds (Azure Artifacts)
- Need authenticated package sources
- Setting up CI/CD pipelines

**Pattern**:
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
    <add key="BunzlEcomArtifacts" value="https://bunzlweb.pkgs.visualstudio.com/BunzlEcom/_packaging/BunzlEcomArtifacts/nuget/v3/index.json" />
  </packageSources>

  <packageSourceCredentials>
    <BunzlEcomArtifacts>
      <add key="Username" value="az" />
      <add key="ClearTextPassword" value="%AZURE_ARTIFACTS_PAT%" />
    </BunzlEcomArtifacts>
  </packageSourceCredentials>
</configuration>
```

**Key Points**:
- `<clear />` removes default sources
- Add nuget.org explicitly if needed
- Use environment variable `%AZURE_ARTIFACTS_PAT%` for credentials
- Place at solution root
- Never commit PAT values directly

**CI/CD Integration**:
- Set `AZURE_ARTIFACTS_PAT` as pipeline variable
- Use `AZURE_DEVOPS_PAT` if following existing conventions

---

## Quick Decision Tree

```
NEED TO CREATE:

├─ New Service?
│  ├─ Has CRUD operations? → Use Base Entity Service Pattern
│  └─ Custom logic only? → Standard service class
│
├─ PATCH endpoint?
│  └─ Use PatchRequest<T> pattern with expression trees
│
├─ Query logic getting complex?
│  └─ Extract to Query Extension methods
│
├─ New Project?
│  ├─ Simple API? → Use Simple Project Structure
│  └─ Complex/Large? → Use Clean Architecture Structure
│
├─ Configure Program.cs?
│  └─ Follow configuration order from example
│
└─ Private NuGet feed?
   └─ Use nuget.config pattern with PAT environment variable
```

## Integration with Commands

### When Generating Services
```bash
/generate-repository Order
# Then reference Base Entity Service pattern for service creation
```

### When Generating Controllers
```bash
/implement-endpoint orders.patch
# Reference PATCH operation example for implementation
```

### When Scaffolding Project
```bash
# For simple project
/scaffold-project --template simple

# For clean architecture
/scaffold-project --template clean-architecture
```

## Common Mistakes to Avoid

### ❌ DON'T:
- Create services without inheriting from base (if using base pattern)
- Implement PATCH with manual property-by-property updates
- Write duplicate query logic in multiple services
- Mix configuration and service registration order in Program.cs
- Commit nuget.config with hardcoded PAT values
- Use Clean Architecture for simple projects (over-engineering)

### ✅ DO:
- Use BaseEntityService for standard CRUD operations
- Use PatchRequest<T> for type-safe partial updates
- Extract common queries to extension methods
- Follow exact middleware pipeline order
- Use environment variables for NuGet credentials
- Choose project structure based on complexity

## Related Documentation

**Human-Readable Examples**:
- [docs/examples/BASE-ENTITY-SERVICE-EXAMPLE.md](../../docs/examples/BASE-ENTITY-SERVICE-EXAMPLE.md)
- [docs/examples/PATCH-OPERATION-EXAMPLE.md](../../docs/examples/PATCH-OPERATION-EXAMPLE.md)
- [docs/examples/QUERY-EXTENSION-EXAMPLE.md](../../docs/examples/QUERY-EXTENSION-EXAMPLE.md)
- [docs/examples/PROJECT-STRUCTURE.md](../../docs/examples/PROJECT-STRUCTURE.md)
- [docs/examples/PROJECT-STRUCTURE-FOR-LARGER-PROJECT.md](../../docs/examples/PROJECT-STRUCTURE-FOR-LARGER-PROJECT.md)
- [docs/examples/DOTNET-STARTUP-FILE-EXAMPLE.md](../../docs/examples/DOTNET-STARTUP-FILE-EXAMPLE.md)
- [docs/examples/NUGET-CONFIG-EXAMPLE.md](../../docs/examples/NUGET-CONFIG-EXAMPLE.md)
- [docs/examples/SOLUTION-FILE-EXAMPLE-FOR-LARGER-PROJECT.md](../../docs/examples/SOLUTION-FILE-EXAMPLE-FOR-LARGER-PROJECT.md)

**Claude Guides**:
- [Database Design Rules](database-design-rules.md)
- [DevOps Quick Reference](devops-quick-reference.md)
- [API Spec Evolution Guide](api-spec-evolution-guide.md)

**Main Project Guide**:
- [CLAUDE.md](../../CLAUDE.md)
