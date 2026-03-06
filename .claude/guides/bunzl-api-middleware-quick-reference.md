# Bunzl API Middleware Quick Reference

Fast reference for Claude Code when implementing Bunzl.APIMiddleware in .NET APIs.

## Purpose

**BunzlEcom.API.Middleware** is a company-wide standard NuGet package that provides:
- Consistent error handling across all APIs
- Authentication with Azure AD
- Standard HTTP error responses (400, 401, 404, 409, 500)
- Reduced boilerplate code

**CRITICAL: This middleware MUST be included in every .NET API project.**

## Quick Decision Algorithm

```
CREATING .NET API PROJECT?
  └─ YES → Install Bunzl.APIMiddleware package
      ├─ Configure exception handlers in Program.cs
      ├─ Configure authentication (if needed)
      ├─ Throw exceptions from SERVICE layer (not controllers)
      └─ Use [Authorize] attribute on protected endpoints
```

## Installation

```bash
# 1. Add Bunzl NuGet source (if not already configured)
dotnet nuget add source "https://bunzlweb.pkgs.visualstudio.com/BunzlEcom/_packaging/BunzlEcomArtifacts/nuget/v3/index.json" --name "BunzlEcomArtifacts"

# If authentication fails, use interactive mode
dotnet nuget add source "https://bunzlweb.pkgs.visualstudio.com/BunzlEcom/_packaging/BunzlEcomArtifacts/nuget/v3/index.json" --name "BunzlEcomArtifacts" --interactive

# 2. Install the middleware package
dotnet add package BunzlEcom.API.Middleware

# 3. Install Microsoft.Identity.Web (for authentication)
dotnet add package Microsoft.Identity.Web --version 3.0.0
```

## Program.cs Configuration

**CRITICAL ORDER**: Follow this exact sequence in Program.cs

### 1. Register Exception Handlers (SPECIFIC → GENERAL)

```csharp
using BunzlAPIMiddleware.BadRequest;
using BunzlAPIMiddleware.NotFound;
using BunzlAPIMiddleware.GlobalException;
using BunzlAPIMiddleware.Authentication;

var builder = WebApplication.CreateBuilder(args);

// Exception handlers MUST be registered in this order
builder.Services.AddExceptionHandler<AlreadyExistsExceptionHandler>();
builder.Services.AddExceptionHandler<NotFoundExceptionHandler>();
builder.Services.AddExceptionHandler<GlobalExceptionHandler>(); // LAST = catch-all
builder.Services.AddProblemDetails();
```

**WHY ORDER MATTERS**:
- Most specific handlers first
- GlobalExceptionHandler catches ALL unhandled exceptions
- If GlobalExceptionHandler is first, specific handlers never execute

### 2. Configure Authentication (If Needed)

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.Identity.Web;

// Add authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

// Configure custom authentication failure responses
builder.Services.Configure<JwtBearerOptions>(JwtBearerDefaults.AuthenticationScheme, options =>
{
    options.Events = new JwtBearerEvents
    {
        OnTokenValidated = context =>
        {
            return NotAuthenticatedHandler.CheckClaims(context, ["preferred_username"]);
        },
        OnChallenge = context =>
        {
            return NotAuthenticatedHandler.HandleResponse(context);
        }
    };
});

builder.Services.AddAuthorization();
```

### 3. Middleware Pipeline Order

```csharp
var app = builder.Build();

// Middleware MUST be in this order
app.UseSwagger();
app.UseSwaggerUI();
app.UseHttpsRedirection();

// CRITICAL: Authentication BEFORE Authorization
app.UseAuthentication();
app.UseAuthorization();

// Exception handler AFTER auth but BEFORE MapControllers
app.UseExceptionHandler();

app.MapControllers();
app.Run();
```

### 4. appsettings.json Configuration

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "common",
    "ClientId": "1687bb26-98cb-4904-9a68-9c4b6974b679"
  }
}
```

## Exception Usage Patterns

### ✓ CORRECT: Throw from Service Layer

```csharp
// Service class
public class CustomerService : ICustomerService
{
    public async Task<Customer> GetByIdAsync(Guid id)
    {
        var customer = await _repository.GetByIdAsync(id);

        if (customer == null)
        {
            throw new NotFoundException("Customer"); // ✓ Throw here
        }

        return customer;
    }

    public async Task CreateAsync(CreateCustomerRequest request)
    {
        var existing = await _repository.FindByEmailAsync(request.Email);

        if (existing != null)
        {
            throw new AlreadyExistsException("Customer", $"Customer with email {request.Email} already exists");
        }

        // Create logic...
    }
}

// Controller class - thin layer, no exception throwing
[ApiController]
[Route("/")]
public class CustomersController(ICustomerService customerService) : ControllerBase
{
    [HttpGet("customers/{id}")]
    [ProducesResponseType(typeof(CustomerResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<CustomerResponse>> GetById(Guid id)
    {
        var customer = await customerService.GetByIdAsync(id); // Service throws if not found
        return Ok(customer); // Only called if service succeeds
    }
}
```

### ✗ INCORRECT: Throw from Controller

```csharp
// ✗ BAD - Don't do this
[HttpGet("customers/{id}")]
public async Task<ActionResult<CustomerResponse>> GetById(Guid id)
{
    var customer = await _service.GetByIdAsync(id);

    if (customer == null)
    {
        throw new NotFoundException("Customer"); // ✗ Business logic in controller
    }

    return Ok(customer);
}
```

## Available Exceptions

### 1. NotFoundException (404)

```csharp
// Overload 1: Generic message
throw new NotFoundException();
// Returns: { "status": 404, "title": "Not Found" }

// Overload 2: With resource name
throw new NotFoundException("Customer");
// Returns: { "status": 404, "title": "Not Found", "detail": "Customer not found" }
```

**When to use**: Entity lookup by ID returns null

### 2. AlreadyExistsException (409)

```csharp
// Overload 1: Pointer only
throw new AlreadyExistsException("Customer");
// Returns: { "status": 409, "title": "Already Exists", "detail": "Customer already exists" }

// Overload 2: Pointer + custom detail
throw new AlreadyExistsException("Customer", "Customer with email john@example.com already exists");
// Returns: { "status": 409, "title": "Already Exists", "detail": "Customer with email john@example.com already exists" }
```

**When to use**: Duplicate constraint violation, unique key conflict

### 3. GlobalExceptionHandler (500)

```csharp
// Any unhandled exception
throw new Exception("Unexpected error occurred");
// Automatically caught by GlobalExceptionHandler
// Returns: { "status": 500, "title": "Internal Server Error" }
// Logs full exception details for debugging
```

**When to use**: Automatically catches all unhandled exceptions (don't throw manually)

### 4. BadRequestException (400)

```csharp
// Model validation failures are automatically handled
// Use [Required], [MaxLength], etc. attributes on DTOs

public class CreateCustomerRequest
{
    [Required]
    [MaxLength(100)]
    public string Name { get; set; }

    [Required]
    [EmailAddress]
    public string Email { get; set; }
}
// Invalid requests automatically return 400 with validation details
```

**When to use**: Automatic via model validation; use data annotations on request DTOs

## Authentication Patterns

### Protected Endpoints

```csharp
[ApiController]
[Route("/")]
public class OrdersController(IOrderService orderService) : ControllerBase
{
    [HttpGet("orders/{id}")]
    [Authorize] // ✓ Require authentication
    [ProducesResponseType(typeof(OrderResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status401Unauthorized)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<OrderResponse>> GetById(Guid id)
    {
        // Extract user from token claims
        var userEmail = HttpContext.User.FindFirst("preferred_username")?.Value;

        var order = await orderService.GetByIdAsync(id, userEmail);
        return Ok(order);
    }
}
```

### Extracting Token Claims

```csharp
// Common claims to extract
var userEmail = HttpContext.User.FindFirst("preferred_username")?.Value;
var userId = HttpContext.User.FindFirst("sub")?.Value;
var userName = HttpContext.User.FindFirst("name")?.Value;

// Use in service layer via dependency injection of IHttpContextAccessor
public class AuditService(IHttpContextAccessor httpContextAccessor) : IAuditService
{
    public string GetCurrentUser()
    {
        return httpContextAccessor.HttpContext?.User
            .FindFirst("preferred_username")?.Value ?? "System";
    }
}
```

## Swagger Configuration with Auth

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });

    // Add Authorization header to Swagger UI
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Description = "JWT Authorization header using the Bearer scheme. Example: \"Authorization: Bearer {token}\"",
        Name = "Authorization",
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.ApiKey,
        Scheme = "Bearer"
    });

    options.AddSecurityRequirement(new OpenApiSecurityRequirement()
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                },
                Scheme = "oauth2",
                Name = "Bearer",
                In = ParameterLocation.Header,
            },
            new List<string>()
        }
    });

    // Include XML comments for documentation
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);
});
```

## Validation Checklist

When implementing Bunzl.APIMiddleware in a new API project:

```
☐ Installed BunzlEcom.API.Middleware package
☐ Installed Microsoft.Identity.Web package (v3.0.0)
☐ Configured nuget.config with BunzlEcomArtifacts feed
☐ Added exception handlers in CORRECT ORDER (AlreadyExists → NotFound → Global)
☐ Added authentication configuration (if needed)
☐ Added AzureAd section to appsettings.json (if using auth)
☐ Configured middleware pipeline in CORRECT ORDER (Auth → Authorization → ExceptionHandler)
☐ Used [Authorize] attribute on protected endpoints
☐ Throw exceptions from SERVICE layer (not controllers)
☐ Added [ProducesResponseType] attributes for 400/401/404/409/500 responses
☐ Configured Swagger with Bearer authentication (if using auth)
☐ Enabled XML documentation for Swagger (optional but recommended)
```

## Common Mistakes to Avoid

### ❌ DON'T:
- Throw exceptions from controller layer (keep controllers thin)
- Register GlobalExceptionHandler before specific handlers
- Use middleware pipeline without authentication before authorization
- Hardcode user information (extract from token claims)
- Return manual error responses (let middleware handle it)
- Skip [ProducesResponseType] attributes for error responses

### ✅ DO:
- Throw exceptions from service/handler layer
- Register exception handlers in order: specific → general → global
- Use authentication before authorization in pipeline
- Extract user from HttpContext.User claims
- Let middleware automatically format error responses
- Document all possible HTTP status codes with [ProducesResponseType]

## Response Examples

### 404 Not Found
```json
{
  "status": 404,
  "title": "Not Found",
  "detail": "Customer not found"
}
```

### 409 Conflict
```json
{
  "status": 409,
  "title": "Already Exists",
  "detail": "Customer with email john@example.com already exists"
}
```

### 400 Bad Request (Model Validation)
```json
{
  "status": 400,
  "title": "Bad Request",
  "errors": {
    "Name": ["The Name field is required."],
    "Email": ["The Email field is not a valid e-mail address."]
  }
}
```

### 401 Unauthorized
```json
{
  "status": 401,
  "title": "Unauthorized",
  "detail": "Invalid or missing authentication token"
}
```

### 500 Internal Server Error
```json
{
  "status": 500,
  "title": "Internal Server Error",
  "detail": "An unexpected error occurred"
}
```

## Integration with Base Service Pattern

```csharp
public abstract class BaseEntityService<TEntity, TKey> where TEntity : class, IEntity<TKey>
{
    protected readonly DbContext Context;
    protected readonly ILogger Logger;

    public virtual async Task<TEntity> GetByIdAsync(TKey id)
    {
        var entity = await Context.Set<TEntity>().FindAsync(id);

        if (entity == null)
        {
            throw new NotFoundException(typeof(TEntity).Name); // ✓ Middleware handles response
        }

        return entity;
    }

    public virtual async Task<TEntity> CreateAsync(TEntity entity)
    {
        // Check for duplicates if applicable
        if (await ExistsAsync(entity))
        {
            throw new AlreadyExistsException(typeof(TEntity).Name); // ✓ Middleware handles response
        }

        Context.Set<TEntity>().Add(entity);
        await Context.SaveChangesAsync();
        return entity;
    }

    protected abstract Task<bool> ExistsAsync(TEntity entity);
}
```

## Related Documentation

**Human-Readable Guide**: [docs/guides/bunzl_api_middleware_guide.md](../../docs/guides/bunzl_api_middleware_guide.md)

**Code Examples**: [docs/examples/BUNZL-API-MIDDLEWARE-USAGE-EXAMPLES.md](../../docs/examples/BUNZL-API-MIDDLEWARE-USAGE-EXAMPLES.md)

**Other Claude Guides**:
- [Code Examples Quick Reference](code-examples-quick-reference.md)
- [DevOps Quick Reference](devops-quick-reference.md)
- [Database Design Rules](database-design-rules.md)

**Main Project Guide**: [CLAUDE.md](../../CLAUDE.md)
