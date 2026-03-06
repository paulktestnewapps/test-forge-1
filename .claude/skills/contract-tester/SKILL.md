---
name: contract-tester
description: Perform contract testing by generating code-first API spec from implementation and comparing against design-first spec. Use for validating implementation matches API contract.
allowed-tools: Read, Glob, Grep, Bash, Write
---

# Contract Tester Skill

## Purpose

Validates that API implementation code matches the design-first OpenAPI specification by:
1. Generating a code-first API spec from Swashbuckle/Swagger
2. Comparing it against the design-first spec (docs/openapi.yaml)
3. Reporting discrepancies between contract and implementation

## When to Use

- Before deployment to verify API contract compliance
- After implementing new endpoints to ensure spec alignment
- During code review to catch contract drift
- As part of CI/CD pipeline validation
- When refactoring controllers or DTOs

## Prerequisites

Project must have:
- Swashbuckle.AspNetCore configured for OpenAPI generation
- A design-first spec at `docs/openapi.yaml`
- Runnable .NET application

## Contract Testing Process

### Phase 1: Generate Code-First Spec

1. **Build the project**
   ```bash
   dotnet build
   ```

2. **Generate OpenAPI spec from code**

   Option A: Using Swashbuckle CLI (preferred)
   ```bash
   # Install CLI tool if needed
   dotnet tool install --global Swashbuckle.AspNetCore.Cli

   # Generate spec from built assembly
   swagger tofile --output swagger-generated.json <path-to-dll> v1
   ```

   Option B: Using runtime endpoint
   ```bash
   # Start the application
   dotnet run &

   # Fetch the generated spec
   curl http://localhost:5000/swagger/v1/swagger.json -o swagger-generated.json
   ```

   Option C: Using MSBuild target
   ```xml
   <!-- In .csproj -->
   <Target Name="GenerateSwagger" AfterTargets="Build">
     <Exec Command="swagger tofile --output $(OutputPath)swagger.json $(OutputPath)$(AssemblyName).dll v1" />
   </Target>
   ```

### Phase 2: Compare Specifications

Compare the generated spec against the design-first spec:

#### Endpoint Comparison

| Aspect | Check |
|--------|-------|
| Paths | All design-first paths exist in implementation |
| HTTP Methods | Methods match for each path |
| Parameters | Query, path, header params match |
| Request Bodies | Schema and required fields match |
| Responses | Status codes and schemas match |
| Authentication | Security requirements match |

#### Schema Comparison

| Aspect | Check |
|--------|-------|
| Schema Names | All design schemas implemented |
| Properties | Field names and types match |
| Required Fields | Required arrays match |
| Enums | Enum values match exactly |
| Validation | Constraints match (min, max, pattern) |

### Phase 3: Generate Report

Output a structured contract test report:

```markdown
# Contract Test Report

**Design Spec**: docs/openapi.yaml
**Generated Spec**: swagger-generated.json
**Date**: {{date}}
**Status**: PASS | FAIL

## Summary

| Category | Matches | Mismatches | Missing |
|----------|---------|------------|---------|
| Endpoints | 15 | 2 | 1 |
| Schemas | 12 | 1 | 0 |
| Parameters | 28 | 0 | 0 |

## Discrepancies

### Missing Endpoints

| Design Spec Path | Method | Notes |
|------------------|--------|-------|
| /orders/{id}/cancel | POST | Not implemented |

### Endpoint Mismatches

| Path | Issue | Design Spec | Implementation |
|------|-------|-------------|----------------|
| /customers | Response schema | CustomerResponse | CustomerDto |
| /orders | Query param | pageSize (required) | pageSize (optional) |

### Schema Mismatches

| Schema | Field | Issue | Design | Implementation |
|--------|-------|-------|--------|----------------|
| OrderResponse | status | Type mismatch | string enum | integer |
| CustomerResponse | email | Missing field | present | absent |

### Extra in Implementation

| Type | Name | Notes |
|------|------|-------|
| Endpoint | GET /health | Not in design spec |
| Schema | HealthResponse | Supporting health endpoint |

## Recommendations

1. **Add missing endpoint**: POST /orders/{id}/cancel
2. **Fix schema mismatch**: Change OrderResponse.status to string enum
3. **Update design spec**: Add GET /health if intended
```

## Classification of Issues

### Breaking Contract Violations (Must Fix)

- Missing required endpoints
- Missing required fields in request/response
- Type mismatches for existing fields
- Missing required parameters
- Changed authentication requirements

### Non-Breaking Differences (Review)

- Extra endpoints in implementation
- Extra optional fields in responses
- Documentation differences
- Example differences

### Acceptable Differences (Ignore)

- Internal endpoints (health, metrics)
- Swagger UI configuration differences
- Server URL differences

## Integration with CI/CD

### GitHub Actions Example

```yaml
- name: Contract Test
  run: |
    dotnet build
    swagger tofile --output generated.json ./bin/Debug/net8.0/MyApi.dll v1
    # Custom comparison script or use oasdiff
    oasdiff diff docs/openapi.yaml generated.json --format json > contract-diff.json
```

### Azure Pipelines Example

```yaml
- script: |
    dotnet build
    dotnet tool install --global Swashbuckle.AspNetCore.Cli
    swagger tofile --output $(Build.ArtifactStagingDirectory)/swagger.json $(Build.SourcesDirectory)/src/MyApi/bin/Debug/net8.0/MyApi.dll v1
  displayName: 'Generate Swagger'

- script: |
    # Compare specs
    npx oasdiff diff docs/openapi.yaml $(Build.ArtifactStagingDirectory)/swagger.json
  displayName: 'Contract Test'
```

## Tools for Comparison

### oasdiff (Recommended)

```bash
# Install
go install github.com/tufin/oasdiff@latest

# Or use Docker
docker run --rm -t tufin/oasdiff diff design.yaml implementation.json

# Breaking changes only
oasdiff breaking docs/openapi.yaml generated.json

# Full diff
oasdiff diff docs/openapi.yaml generated.json --format text
```

### Manual Comparison (Claude)

When automated tools aren't available, Claude can:
1. Read both spec files
2. Parse endpoint definitions
3. Compare schemas field by field
4. Generate structured diff report

## Output Artifacts

1. **contract-test-report.md**: Human-readable report
2. **contract-diff.json**: Machine-readable diff (if using oasdiff)
3. **swagger-generated.json**: Code-first spec for reference

## Best Practices

1. **Run contract tests early**: Before merging PRs
2. **Automate in CI**: Fail builds on breaking violations
3. **Version control generated spec**: Track drift over time
4. **Update design spec deliberately**: Changes should be intentional
5. **Document exceptions**: Some differences may be acceptable

## Error Handling

### Build Failure
```
Error: Build failed
Solution: Fix compilation errors before contract testing
```

### Swagger Generation Failure
```
Error: Could not generate Swagger spec
Solution: Ensure Swashbuckle is configured in Program.cs
```

### Spec Parse Failure
```
Error: Invalid OpenAPI specification
Solution: Validate both specs with a linter
```

## Related Skills

- `api-spec-interpreter`: For parsing OpenAPI specs
- `api-diff-analyzer`: For detailed spec comparison
- `controller-generator`: For implementing missing endpoints
