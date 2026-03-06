---
name: error-handler
description: Implement consistent error handling using Bunzl.APIMiddleware with NotFoundException and AlreadyExistsException. Use when setting up API error responses or handling exceptions.
allowed-tools: Read, Edit, Write, Glob, Grep
---

# Error Handler

## Purpose

Implements consistent error handling using **BunzlEcom.API.Middleware** for non-success responses.

**CRITICAL: This middleware MUST be used in EVERY .NET API project.**

## When to Use

- Setting up new .NET API projects
- Setting up exception handling in services
- Configuring Program.cs for error handling
- Implementing consistent error responses
- Adding new error scenarios

## Installation

**REQUIRED for every .NET API project:**

```bash
dotnet add package BunzlEcom.API.Middleware
dotnet add package Microsoft.Identity.Web --version 3.0.0
```

## Program.cs Configuration

**CRITICAL ORDER - Exception handlers must be registered in this exact order:**

```csharp
using BunzlAPIMiddleware.BadRequest;
using BunzlAPIMiddleware.NotFound;
using BunzlAPIMiddleware.GlobalException;
using BunzlAPIMiddleware.Authentication;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.Identity.Web;

var builder = WebApplication.CreateBuilder(args);

// 1. Register exception handlers (SPECIFIC → GENERAL)
builder.Services.AddExceptionHandler<AlreadyExistsExceptionHandler>();
builder.Services.AddExceptionHandler<NotFoundExceptionHandler>();
builder.Services.AddExceptionHandler<GlobalExceptionHandler>(); // LAST = catch-all
builder.Services.AddProblemDetails();

// 2. Authentication (if needed)
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

builder.Services.Configure<JwtBearerOptions>(JwtBearerDefaults.AuthenticationScheme, options =>
{
    options.Events = new JwtBearerEvents
    {
        OnTokenValidated = context => NotAuthenticatedHandler.CheckClaims(context, ["preferred_username"]),
        OnChallenge = context => NotAuthenticatedHandler.HandleResponse(context)
    };
});

builder.Services.AddAuthorization();

var app = builder.Build();

// 3. Middleware pipeline order (CRITICAL)
app.UseAuthentication();    // Before authorization
app.UseAuthorization();     // Before exception handler
app.UseExceptionHandler();  // After auth
app.MapControllers();
app.Run();
```

## Project-Specific Conventions

### Standard Exceptions

| Exception | Namespace | HTTP Status | Usage |
|-----------|-----------|-------------|-------|
| `NotFoundException` | BunzlAPIMiddleware.NotFound | 404 | Resource not found |
| `AlreadyExistsException` | BunzlAPIMiddleware.BadRequest | 409 | Duplicate constraint violation |
| `GlobalExceptionHandler` | BunzlAPIMiddleware.GlobalException | 500 | Catches all unhandled exceptions |
| Model Validation | Automatic | 400 | Use data annotations on request DTOs |

## Usage in Services

### NotFoundException

Throw when a requested resource doesn't exist:

```csharp
public async Task<CustomerDetailResponse> GetByIdAsync(Guid id, CancellationToken ct)
{
    var customer = await _repository.GetByIdAsync(id, ct)
        ?? throw new NotFoundException($"Customer with ID '{id}' was not found.");

    return customer;
}

public async Task DeleteAsync(Guid id, CancellationToken ct)
{
    var exists = await _repository.ExistsAsync(id, ct);
    if (!exists)
    {
        throw new NotFoundException($"Customer with ID '{id}' was not found.");
    }

    await _repository.DeleteAsync(id, ct);
}
```

### AlreadyExistsException

Throw when an operation violates a uniqueness constraint:

```csharp
public async Task CreateAsync(CreateCustomerRequest request, CancellationToken ct)
{
    if (await _repository.ExistsByEmailAsync(request.Email, ct))
    {
        throw new AlreadyExistsException($"Customer with email '{request.Email}' already exists.");
    }

    var customer = _mapper.Map<Customer>(request);
    await _repository.AddAsync(customer, ct);
}

public async Task UpdateAsync(Guid id, UpdateCustomerRequest request, CancellationToken ct)
{
    var customer = await GetEntityByIdAsync(id, ct);

    // Check if email is being changed to one that already exists
    if (customer.Email != request.Email &&
        await _repository.ExistsByEmailAsync(request.Email, ct))
    {
        throw new AlreadyExistsException($"Customer with email '{request.Email}' already exists.");
    }

    _mapper.Map(request, customer);
    await _repository.UpdateAsync(customer, ct);
}
```

## Controller Behavior

Controllers should **NOT** catch exceptions. Let Bunzl.APIMiddleware handle them:

```csharp
// CORRECT: Let exceptions propagate
[HttpGet("customers/{id:guid}")]
[ProducesResponseType(typeof(CustomerDetailResponse), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<ActionResult<CustomerDetailResponse>> GetById(Guid id, CancellationToken ct)
{
    var customer = await customerService.GetByIdAsync(id, ct);
    return Ok(customer);
}

// INCORRECT: Don't catch exceptions in controllers
[HttpGet("customers/{id:guid}")]
public async Task<ActionResult<CustomerDetailResponse>> GetById(Guid id, CancellationToken ct)
{
    try
    {
        var customer = await customerService.GetByIdAsync(id, ct);
        return Ok(customer);
    }
    catch (NotFoundException)
    {
        return NotFound(); // DON'T DO THIS - let middleware handle it
    }
}
```

## Input Validation

Use data annotations in request DTOs for input validation:

```csharp
public record CreateCustomerRequest(
    [Required]
    [StringLength(100, MinimumLength = 1)]
    string Name,

    [Required]
    [EmailAddress]
    string Email,

    [Phone]
    string? PhoneNumber
);
```

The `[ApiController]` attribute enables automatic model validation, returning 400 Bad Request for invalid input.

## Custom Exception Classes (If Needed)

If Bunzl.APIMiddleware doesn't provide a needed exception:

```csharp
namespace MyApi.Exceptions;

/// <summary>
/// Thrown when a business rule is violated
/// </summary>
public class BusinessRuleException : Exception
{
    public string RuleCode { get; }

    public BusinessRuleException(string ruleCode, string message)
        : base(message)
    {
        RuleCode = ruleCode;
    }
}

// Usage
throw new BusinessRuleException("ORDER_001", "Cannot cancel an order that has already shipped.");
```

## Service Layer Pattern

```csharp
namespace MyApi.Services;

public class CustomerService(
    ICustomerRepository repository,
    IMapper mapper,
    ILogger<CustomerService> logger) : ICustomerService
{
    public async Task<CustomerDetailResponse> GetByIdAsync(Guid id, CancellationToken ct)
    {
        logger.LogDebug("Getting customer {CustomerId}", id);

        return await repository.GetByIdAsync(id, ct)
            ?? throw new NotFoundException($"Customer with ID '{id}' was not found.");
    }

    public async Task CreateAsync(CreateCustomerRequest request, CancellationToken ct)
    {
        logger.LogDebug("Creating customer with email {Email}", request.Email);

        if (await repository.ExistsByEmailAsync(request.Email, ct))
        {
            throw new AlreadyExistsException($"Customer with email '{request.Email}' already exists.");
        }

        var customer = mapper.Map<Customer>(request);
        await repository.AddAsync(customer, ct);

        logger.LogInformation("Created customer {CustomerId}", customer.Id);
    }

    public async Task DeleteAsync(Guid id, CancellationToken ct)
    {
        logger.LogDebug("Deleting customer {CustomerId}", id);

        if (!await repository.ExistsAsync(id, ct))
        {
            throw new NotFoundException($"Customer with ID '{id}' was not found.");
        }

        await repository.DeleteAsync(id, ct);

        logger.LogInformation("Deleted customer {CustomerId}", id);
    }
}
```

## ProducesResponseType Attributes

Always include response type attributes for documentation:

```csharp
[HttpGet("customers/{id:guid}")]
[ProducesResponseType(typeof(CustomerDetailResponse), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<ActionResult<CustomerDetailResponse>> GetById(Guid id, CancellationToken ct)

[HttpPost("customers")]
[ProducesResponseType(StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
[ProducesResponseType(StatusCodes.Status409Conflict)]
public async Task<IActionResult> Create([FromBody] CreateCustomerRequest request, CancellationToken ct)
```

## Best Practices

1. **Install BunzlEcom.API.Middleware in EVERY .NET API project**
2. **Configure exception handlers in EXACT ORDER** - AlreadyExists → NotFound → Global
3. **Throw from service layer** - NEVER from controllers or repositories
4. **Use specific exceptions** - NotFoundException vs generic Exception
5. **Include context in messages** - ID, email, resource name
6. **Log before throwing** - For debugging
7. **Let middleware handle responses** - Don't catch in controllers
8. **Include all [ProducesResponseType] attributes** - 200, 201, 204, 400, 401, 404, 409, 500
9. **Use data annotations for validation** - [Required], [EmailAddress], [StringLength], etc.

## Required Imports

**In Program.cs:**
```csharp
using BunzlAPIMiddleware.BadRequest;
using BunzlAPIMiddleware.NotFound;
using BunzlAPIMiddleware.GlobalException;
using BunzlAPIMiddleware.Authentication;
```

**In Service classes:**
```csharp
using BunzlAPIMiddleware.NotFound;
using BunzlAPIMiddleware.BadRequest;
```

## Reference Documentation

**Quick Reference:** [Bunzl API Middleware Quick Reference](../../guides/bunzl-api-middleware-quick-reference.md)

**Human Documentation:** [Bunzl API Middleware Guide](../../../docs/guides/bunzl_api_middleware_guide.md)

**Code Examples:** [Usage Examples](../../../docs/examples/BUNZL-API-MIDDLEWARE-USAGE-EXAMPLES.md)

**Related:**
- [Program.cs Example](../../../docs/examples/DOTNET-STARTUP-FILE-EXAMPLE.md)
- [CLAUDE.md](../../../CLAUDE.md)
