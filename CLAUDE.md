# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

<!-- bunzl-forge:section:project-info -->
## Project Overview

**Project Name:** R3 Redistribution API

**Description:** B2B product catalog and lead generation API for R3 Redistribution

This is a .NET 8 API project following Bunzl development standards and best practices.
<!-- bunzl-forge:end:project-info -->

<!-- bunzl-forge:section:openapi-reference -->
## API Specification

**OpenAPI Spec:** [docs/openapi.yaml](docs/openapi.yaml)

Claude should reference the OpenAPI specification for:
- Endpoint definitions and routes
- Request/response schemas
- Data types and validation rules
- Error response formats
<!-- bunzl-forge:end:openapi-reference -->

<!-- bunzl-forge:section:getting-started -->
## Getting Started Workflow

**For new projects**, follow this workflow:

1. **Initialize project structure**: `/initialize-project`
   - Sets up user configurations
   - Creates `.forge-backend` folder
   - Generates project documentation

2. **Add API specification**: Place your `openapi.yaml` in the project root

3. **Implement API**: `/implement-api`
   - Automatically detects if database schema exists
   - Guides through database design if needed
   - Generates full API layer (controllers, services, repositories, tests)

4. **Validate**: `dotnet build && dotnet test`

## API Evolution Workflow

**When API spec is updated**, use the automated workflow for implementing changes:

1. **Compare specifications**: `/compare-api-specs baseline.yaml updated.yaml --plan`
2. **Review diff report** and approve implementation plan
3. **Implement changes**: `/implement-spec-changes implementation-plan.md`
4. **Validate**: `dotnet build && dotnet test`
5. **Deploy** with appropriate versioning strategy

**Full Workflow Guide**: [.claude/workflows/api-spec-evolution.md](.claude/workflows/api-spec-evolution.md)

**Quick Reference**: [.claude/guides/api-spec-evolution-guide.md](.claude/guides/api-spec-evolution-guide.md)

## Contract Testing Workflow

**Validate implementation matches design-first spec:**

1. **Run contract test**: `/contract-test`
   - Generates code-first API spec from implementation (Swashbuckle)
   - Compares against design-first spec (docs/openapi.yaml)
   - Reports discrepancies between contract and implementation

2. **Fix violations**: Address missing endpoints, schema mismatches, type differences

3. **Re-validate**: Run `/contract-test` until all violations resolved

**Quick Reference**: [.claude/guides/contract-testing-guide.md](.claude/guides/contract-testing-guide.md)
<!-- bunzl-forge:end:getting-started -->

<!-- bunzl-forge:section:dev-commands -->
## Development Commands

```bash
# Build
dotnet build

# Run tests
dotnet test

# Format code
dotnet format

# Run with watch
dotnet watch run

# Generate migrations
dotnet ef migrations add <MigrationName>

# Apply migrations
dotnet ef database update
```
<!-- bunzl-forge:end:dev-commands -->

<!-- bunzl-forge:section:architecture -->
## Architecture

### Default Pattern: Service-Repository Hybrid

Use a **Service-Repository Hybrid pattern** for simple backend operations with Entity Framework Core.

```
Controller → Service → Repository → DbContext
```

**Evolve this pattern when:**
- Entities become complex → Add dedicated Repository layer
- Read/write patterns diverge → Add CQRS pattern
- Business rules become complex → Add Specification pattern
- Performance is critical → Consider Dapper over EF Core

### Layer Responsibilities

| Layer | Responsibility |
|-------|---------------|
| Controller | HTTP request/response, validation, routing |
| Service | Business logic, orchestration |
| Repository | Data access, EF Core queries |
<!-- bunzl-forge:end:architecture -->

<!-- bunzl-forge:section:database-design -->
## Database Design

**Quick Reference:** See [.claude/guides/database-design-rules.md](.claude/guides/database-design-rules.md) and [.claude/guides/primary-key-decision-matrix.md](.claude/guides/primary-key-decision-matrix.md) for fast lookups.

### Table Naming
- Concise and descriptive (2-3 words ideal)
- No "Bunzl" prefix by default
- Use casing consistent with existing tables or database provider conventions

### Required Audit Columns
Every table must include:
| Column | Type | Nullable | Notes |
|--------|------|----------|-------|
| CreatedOn | datetime (UTC) | No | Set on insert |
| CreatedBy | string | No | Set on insert |
| UpdatedOn | datetime (UTC) | Yes | Set on update only |
| UpdatedBy | string | Yes | Set on update only |

### Soft Deletes
- Include `IsActive` (bool, not-null) for tables with API DELETE operations
- Avoid `DateOnly` types; use `DateTime` instead

### Primary Key Selection

**Default: Incremental integers** for most internal/performance-sensitive systems.

**Use UUIDs only when:**
- Primary key exposed via APIs with private/sensitive data (e.g., User PII, Orders)
- Distributed systems requiring independent ID generation
- Data merging/sync across multiple databases

**Do NOT use UUIDs for:**
- Lookup/reference tables (e.g., OrderStatus, ResourceType)
- Internal-only tables not exposed via APIs

**Hybrid approach:** Use integer PK internally + separate UUID column for external APIs.

### Normalization
- Use join tables for many-to-many relationships
- Avoid comma-separated values in columns
- Create separate tables for status/type lookups (e.g., OrderStatus with Id + Value)
- Limit tables to ~10 value columns; more may indicate need for optimization
<!-- bunzl-forge:end:database-design -->

<!-- bunzl-forge:section:code-conventions -->
## Code Conventions

**Quick Reference:** See [.claude/guides/code-examples-quick-reference.md](.claude/guides/code-examples-quick-reference.md) for production code patterns.

**Bunzl API Middleware:** See [.claude/guides/bunzl-api-middleware-quick-reference.md](.claude/guides/bunzl-api-middleware-quick-reference.md) for error handling and authentication.

### C# Style

- **Always use primary constructors** (IDE0290) for all classes
- **Use coalesce expression** for null checks (IDE0270)
- Use class inheritance to DRY up repeating code
- Prefer projection to DTOs over returning entities directly

### Controllers

- Name after the resource: `CustomerController`, `OrderController`
- Use `[ApiController]` attribute for automatic model validation
- Use `[Route("/")]` as base route; each action defines its own route
- POST operations return `201 Created` with no body: `return StatusCode(201);`
- Include `[ProducesResponseType]` attributes for all actions (including 400, 401, 404, 409, 500)
- Use data annotations in request body classes for validation
- Inject services via constructor (primary constructor)
- **Keep controllers thin** - no business logic, no exception throwing (let services handle it)

```csharp
[ApiController]
[Route("/")]
[Produces("application/json")]
public class CustomersController(ICustomerService customerService) : ControllerBase
{
    [HttpGet("customers/{id}")]
    [ProducesResponseType(typeof(CustomerResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<CustomerResponse>> GetById(Guid id)
    {
        var customer = await customerService.GetByIdAsync(id);
        return Ok(customer);
    }

    [HttpPost("customers")]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> Create([FromBody] CreateCustomerRequest request)
    {
        await customerService.CreateAsync(request);
        return StatusCode(201);
    }
}
```

### Repository Layer

- Interacts directly with DbContext
- Contains all EF Core queries
- Injected into service layer via DI
- **Never unit tested directly** - repositories are mocked in service tests
- Use query extension methods for common patterns
- **Keep repositories "thin"** - minimal to no business logic

### Services

- Use base service class to reduce repetition
- **Throw exceptions from services, NOT controllers** (Bunzl.APIMiddleware handles formatting)
- Throw `NotFoundException` (from Bunzl.APIMiddleware.NotFound) for 404 responses
- Throw `AlreadyExistsException` (from Bunzl.APIMiddleware.BadRequest) for duplicate constraint violations
- Extract common query logic to query extension methods

### DTOs and Mapping

- Use **AutoMapper** for object-object mapping
- Mappings and DTOs should be in different classes/files
- Use DTO projections to reduce data transfer
- Only include fields needed for the projection
- PATCH operations should use expression trees
<!-- bunzl-forge:end:code-conventions -->

<!-- bunzl-forge:section:libraries -->
## Libraries

| Library | Purpose |
|---------|---------|
| **BunzlEcom.API.Middleware** | **REQUIRED: Exception handling, authentication, standardized error responses** |
| AutoMapper | Object-object mapping |
| Swashbuckle | API documentation, Swagger UI |
| Entity Framework Core | ORM |
| Microsoft.Identity.Web | Azure AD JWT bearer authentication (v3.0.0) |
| xUnit | Testing framework |
| Moq | Mocking in unit tests |
<!-- bunzl-forge:end:libraries -->

<!-- bunzl-forge:section:testing -->
## Testing

### Mandatory Testing Rule

**ALWAYS add or modify unit tests when code is added or modified.** This is not optional:
- New service methods require corresponding unit tests
- Modified service methods require updated tests
- New business logic in any layer requires test coverage
- Bug fixes should include a regression test

### Task Completion Criteria

**A task is NOT complete until:**
1. All new/modified code has corresponding unit tests
2. All unit tests pass (`dotnet test` exits with code 0)
3. No skipped or ignored tests for the new functionality
4. Code formatting is correct (`dotnet format` produces no changes)

### What to Test (and What Not To)

| Layer | Test? | Reason |
|-------|-------|--------|
| Services | **Yes** | Contains business logic; mock repositories |
| AutoMapper Profiles | **Yes** | Verify mapping configurations are correct |
| Controllers | Optional | Thin layer; mostly delegates to services |
| Repositories | **No** | Thin data access; no business logic to test |
| DTOs | **No** | Data containers only; no logic to test |

**Never set up in-memory databases or DbContext for unit tests.** Mock the repository interface instead.

### Unit Testing Standards

- **Framework**: xUnit
- **Mocking**: Moq
- **Pattern**: Arrange-Act-Assert with section comments

```csharp
[Fact]
public async Task GetById_WhenCustomerExists_ReturnsCustomer()
{
    // Arrange
    var expectedCustomer = TestDataFactory.CreateCustomer();
    _mockRepository.Setup(r => r.GetByIdAsync(expectedCustomer.Id))
        .ReturnsAsync(expectedCustomer);

    // Act
    var result = await _service.GetByIdAsync(expectedCustomer.Id);

    // Assert
    Assert.NotNull(result);
    Assert.Equal(expectedCustomer.Id, result.Id);
}
```

### Testing Best Practices

- No hard-coded values; use constants or configuration
- Base assertions on expected output data
- Create shared test data classes for repeated models
- Only create helpers when used in 2+ tests AND more than 2 lines
- Mock all dependencies (repositories, external services) - never use real implementations
- Test both happy paths and error cases (e.g., `NotFoundException` scenarios)
<!-- bunzl-forge:end:testing -->

<!-- bunzl-forge:section:infrastructure -->
## Infrastructure

### Configuration

- **Azure App Configuration** for managing settings across environments
- **appsettings.json** for non-sensitive local settings
- **secrets.json** for sensitive local development data (never commit)

### Infrastructure as Code

- Use **Bicep** for Azure deployments
- Automate API publication to API Management via CI/CD

**IaC Use Cases:**
- API Management publication
- Azure resources (App Service, SQL Database, Storage)
- Azure Container App configuration
- Environment variables via App Configuration
- Monitoring (Application Insights, Log Analytics)
<!-- bunzl-forge:end:infrastructure -->

<!-- bunzl-forge:section:devops -->
## DevOps

**Quick Reference:** See [.claude/guides/devops-quick-reference.md](.claude/guides/devops-quick-reference.md) for fast lookups and troubleshooting.

### Project Structure

```
.devops/
  azure-pipelines.yaml      # Main pipeline definition
  variables.yml             # Shared variables

nuget.config                # NuGet source configuration
Dockerfile                  # Container build definition
.dockerignore               # Docker build exclusions
```

### Key Setup Requirements

1. **AZURE_DEVOPS_PAT**: Add as pipeline variable with `packaging:read/write` permissions
2. **nuget.config**: Include private NuGet feed for internal packages
3. **.dockerignore**: Exclude build artifacts to optimize Docker images
4. **Environment Approvals**: Request access for sbx, uat, prod environments

### Connecting to Private NuGet Feed

```bash
# Add source
dotnet nuget add source "https://bunzlweb.pkgs.visualstudio.com/BunzlEcom/_packaging/BunzlEcomArtifacts/nuget/v3/index.json" --name "BunzlEcomArtifacts"

# If authentication fails, use interactive mode
dotnet nuget add source "https://bunzlweb.pkgs.visualstudio.com/BunzlEcom/_packaging/BunzlEcomArtifacts/nuget/v3/index.json" --name "BunzlEcomArtifacts" --interactive
```

### Variable Reference Syntax

| Context | Syntax | Example |
|---------|--------|---------|
| YAML | `$(VAR_NAME)` | `$(AZURE_DEVOPS_PAT)` |
| Bash | `$(VAR_NAME)` | `$(AZURE_DEVOPS_PAT)` |
| PowerShell | `${env:VAR_NAME}` | `${env:AZURE_DEVOPS_PAT}` |
| Batch | `%VAR_NAME%` | `%AZURE_DEVOPS_PAT%`
<!-- bunzl-forge:end:devops -->

<!-- bunzl-forge:section:exception-handling -->
## Exception Handling

**Quick Reference:** See [.claude/guides/bunzl-api-middleware-quick-reference.md](.claude/guides/bunzl-api-middleware-quick-reference.md)

### CRITICAL: Use BunzlEcom.API.Middleware in ALL .NET API Projects

**Installation:**
```bash
dotnet add package BunzlEcom.API.Middleware
dotnet add package Microsoft.Identity.Web --version 3.0.0
```

**Program.cs Setup (CRITICAL ORDER):**
```csharp
// 1. Exception handlers (specific → general)
builder.Services.AddExceptionHandler<AlreadyExistsExceptionHandler>();
builder.Services.AddExceptionHandler<NotFoundExceptionHandler>();
builder.Services.AddExceptionHandler<GlobalExceptionHandler>(); // LAST

// 2. Authentication
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

// Middleware pipeline order
app.UseAuthentication();
app.UseAuthorization();
app.UseExceptionHandler();
app.MapControllers();
```

**Available Exceptions:**

| Exception | Namespace | HTTP Status | When to Use |
|-----------|-----------|-------------|-------------|
| `NotFoundException` | BunzlAPIMiddleware.NotFound | 404 | Entity lookup by ID returns null |
| `AlreadyExistsException` | BunzlAPIMiddleware.BadRequest | 409 | Duplicate constraint violation |
| `GlobalExceptionHandler` | BunzlAPIMiddleware.GlobalException | 500 | Catches all unhandled exceptions |
| Model Validation | Automatic | 400 | Use data annotations on request DTOs |

**CRITICAL PATTERN: Throw from Services, NOT Controllers**
```csharp
// CORRECT - Service throws
public async Task<Customer> GetByIdAsync(Guid id)
{
    var customer = await _repository.GetByIdAsync(id);
    if (customer == null)
        throw new NotFoundException("Customer"); // Middleware handles response
    return customer;
}

// WRONG - Don't throw from controllers
[HttpGet("customers/{id}")]
public async Task<ActionResult> GetById(Guid id)
{
    var customer = await _service.GetByIdAsync(id);
    if (customer == null)
        throw new NotFoundException("Customer"); // DON'T DO THIS
    return Ok(customer);
}
```
<!-- bunzl-forge:end:exception-handling -->

<!-- bunzl-forge:section:slash-commands -->
## Available Slash Commands

### Project Setup
- `/initialize-project` - **Run this FIRST** - Set up project structure, move user configs, create backend folder

### Database & Schema
- `/derive-entities [path]` - Extract entity list summary from API spec
- `/generate-efcore-from-ddl [path]` - Generate EF Core entities and DbContext from PostgreSQL DDL
- `/generate-db-schema [entity]` - Generate ORM entities and DbContext
- `/generate-sql --db postgres` - Generate SQL migration scripts
- `/generate-er-diagram` - Generate ER diagram from entities

### API Implementation
- `/generate-controller [endpoint]` - Generate controller with routes
- `/generate-dtos [entity]` - Generate request/response DTOs
- `/generate-mapper [source] [target]` - Generate AutoMapper profile
- `/generate-repository [entity]` - Generate repository interface/implementation
- `/generate-error-handler` - Generate global exception handling
- `/implement-endpoint [resource.action]` - Implement a specific endpoint
- `/implement-api [spec-path]` - **Main command** - Context-aware full API implementation
- `/regen-api-layer` - Regenerate API layer from updated entities

### API Evolution
- `/compare-api-specs [baseline] [updated]` - Compare two API specs and identify changes
- `/implement-spec-changes [plan]` - Implement changes from API spec comparison
- `/contract-test` - Validate implementation matches design-first spec (generates code-first spec and compares)

### Architecture
- `/recommend-pattern` - Analyze and recommend architectural patterns
- `/analyze-endpoint [endpoint]` - Analyze endpoint complexity and intent
- `/compare-patterns [p1] [p2]` - Compare two patterns for your codebase
- `/apply-pattern [pattern]` - Apply architectural pattern (hexagonal, clean, etc.)

### Testing
- `/generate-tests [target]` - Generate unit tests for a class or method

### DevOps
- `/scaffold-devops` - Scaffold complete DevOps configuration for .NET API
- `/generate-pipeline [platform]` - Generate CI/CD pipeline
- `/generate-iac [tool] [cloud]` - Generate Infrastructure as Code
- `/devops-review` - Review DevOps configuration
- `/deploy-architecture` - Generate deployment architecture docs
<!-- bunzl-forge:end:slash-commands -->

<!-- bunzl-forge:section:project-specific -->
## Project-Specific Configuration

<!-- Add project-specific notes and configurations here -->
<!-- This section is preserved during updates -->
<!-- bunzl-forge:end:project-specific -->
