---
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
argument-hint: [api-spec-path (optional)]
description: Implement full API from specification
---

## Context

**Locate API Specification:**
1. If $ARGUMENTS provided: Use specified path
2. Otherwise, search project root for:
   - `openapi.yaml`
   - `openapi.yml`
   - `swagger.yaml`
   - `swagger.yml`
   - `api-spec.yaml`
   - Any `*.openapi.yaml` files
3. If multiple found: Present list and ask user to choose
4. If none found: Error with helpful message to create one

**Load and analyze:**
- OpenAPI/Swagger specification (from above)
- Existing project structure
- Framework conventions
- Database schema (if exists)

## Task: Context-Aware API Implementation

This command intelligently handles both scenarios:
- **New project (no schema)**: Full workflow from schema design to API layer
- **Existing schema**: Direct API layer implementation

### Phase 1: Schema Detection and Setup

**Check for existing database schema:**
1. Search for entity classes (e.g., `*.cs` files with DbSet references, EF Core annotations)
2. Search for DbContext class
3. Search for SQL/DDL files in /docs or /database folders

**If NO schema found:**
1. **Derive entities** from API spec (similar to `/derive-entities`)
   - Identify resources from API paths
   - Infer relationships from nested routes
   - Present entity summary to user

2. **Guide database design:**
   - Apply database design rules from [database-design-rules.md](.claude/guides/database-design-rules.md)
   - Use [primary-key-decision-matrix.md](.claude/guides/primary-key-decision-matrix.md) for PK type decisions
   - Ask clarifying questions for:
     - Primary key types (UUID vs integer) - use decision matrix
     - Soft delete requirements
     - Special indexes needed
     - Complex relationships

3. **Generate schema artifacts:**
   - Create entity classes with EF Core annotations
   - Generate DbContext with proper configuration
   - Include all required audit columns (CreatedOn, CreatedBy, UpdatedOn, UpdatedBy)
   - Add IsActive for soft deletes where needed
   - Set up relationships and constraints

**If schema EXISTS:**
- Skip directly to Phase 2
- Log: "Detected existing schema, proceeding with API layer implementation"

### Phase 2: API Layer Implementation

Implement full API from specification:

1. **Generate all controllers**
   - Use `[Route("/")]` base route
   - Each action defines its own full route
   - Include ProducesResponseType attributes (200, 201, 400, 404, 409, 500)
   - POST operations return 201 with no body
   - Use primary constructors for DI

2. **Create DTOs**
   - Request DTOs with data annotations for validation
   - Response DTOs for API responses
   - Separate files for each DTO

3. **Generate AutoMapper profiles**
   - Entity → Response mappings
   - Request → Entity mappings
   - Separate profile classes

4. **Generate repository layer**
   - Interface and implementation for each entity
   - Repository returns entities
   - Keep repositories thin (CRUD only)
   - Use query extension methods for common patterns

5. **Generate service layer**
   - Business logic orchestration
   - Exception handling (throw NotFoundException, AlreadyExistsException)
   - Services call repositories

6. **Set up error handling**
   - Verify Bunzl.APIMiddleware is configured
   - Check exception handlers in Program.cs
   - Validate middleware pipeline order

7. **Generate unit tests**
   - Service tests with mocked repositories
   - AutoMapper profile tests
   - Use xUnit + Moq
   - Arrange-Act-Assert pattern

### Phase 3: Validation and Summary

1. **Run validation:**
   - `dotnet build` (must succeed)
   - `dotnet test` (all tests must pass)
   - `dotnet format` (code formatting)

2. **Provide summary:**
   - List all created files organized by layer
   - Schema artifacts (if created)
   - Controllers, DTOs, Services, Repositories
   - Tests generated
   - Manual steps needed (e.g., connection string setup)

### Important Notes

- Always throw exceptions from services, NOT controllers
- Keep controllers thin - no business logic
- Keep repositories thin - CRUD only, no business logic
- All business logic belongs in service layer
- Mock repositories in service tests, never use in-memory DbContext
- Follow [code-examples-quick-reference.md](.claude/guides/code-examples-quick-reference.md) for patterns
