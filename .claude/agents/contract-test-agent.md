# Contract Test Agent

## Purpose

Specialized agent for performing contract testing between API implementation and design-first OpenAPI specification. Validates that the code correctly implements the API contract.

## When to Use

- User runs `/contract-test` command
- Before deployment to validate contract compliance
- After implementing endpoints to verify spec alignment
- During code review to detect contract drift
- As part of API evolution workflow

## Capabilities

### Code-First Spec Generation

1. **Locate project assembly**
   - Find .csproj file
   - Determine output DLL path
   - Verify Swashbuckle configuration

2. **Generate OpenAPI from code**
   - Use Swashbuckle CLI tool
   - Extract spec from running application
   - Parse controller attributes and DTOs

3. **Validate generated spec**
   - Ensure valid OpenAPI format
   - Check for generation warnings

### Specification Comparison

**Endpoint Analysis:**
- Path matching (exact and pattern)
- HTTP method verification
- Operation ID comparison
- Tag grouping validation

**Schema Analysis:**
- Property name matching
- Type compatibility checking
- Required field verification
- Enum value comparison
- Nested schema validation

**Parameter Analysis:**
- Query parameter matching
- Path parameter validation
- Header parameter verification
- Request body schema comparison

**Response Analysis:**
- Status code coverage
- Response schema matching
- Error response validation

### Discrepancy Classification

**Contract Violations (FAIL):**
- Missing implemented endpoints
- Missing required request fields
- Response schema mismatches
- Authentication requirement differences
- Breaking type changes

**Warnings (REVIEW):**
- Extra endpoints not in design spec
- Extra optional response fields
- Documentation differences
- Deprecation status differences

**Info (OK):**
- Internal endpoints (health, metrics)
- Swagger metadata differences
- Example value differences

## Workflow Steps

### Step 1: Preparation

```
1. Locate design-first spec (docs/openapi.yaml)
2. Find .NET project file
3. Check Swashbuckle configuration
4. Verify project builds successfully
```

### Step 2: Generate Code-First Spec

```
1. Build the project (dotnet build)
2. Locate output assembly
3. Run Swashbuckle CLI or equivalent
4. Save generated spec to temp location
5. Validate generated spec format
```

### Step 3: Parse Both Specs

```
1. Load design-first spec (YAML/JSON)
2. Load code-first spec (JSON)
3. Normalize paths and schemas
4. Build comparison maps
```

### Step 4: Compare Endpoints

```
For each endpoint in design spec:
  1. Find matching path in code spec
  2. Compare HTTP methods
  3. Compare parameters
  4. Compare request bodies
  5. Compare responses
  6. Record discrepancies

For each endpoint in code spec NOT in design:
  1. Record as "extra implementation"
  2. Classify (internal vs public)
```

### Step 5: Compare Schemas

```
For each schema in design spec:
  1. Find matching schema in code spec
  2. Compare properties
  3. Compare required arrays
  4. Compare nested references
  5. Record discrepancies
```

### Step 6: Generate Report

```
1. Summarize findings
2. List violations by severity
3. Provide fix recommendations
4. Output markdown report
```

## Output Format

### Contract Test Report Structure

```markdown
# Contract Test Report

**Generated**: {{timestamp}}
**Design Spec**: docs/openapi.yaml
**Code Spec**: Generated from assembly
**Result**: PASS | FAIL | WARN

## Executive Summary

- Total Endpoints: 18 design / 17 implemented
- Total Schemas: 12 design / 14 implemented
- Contract Violations: 3
- Warnings: 2

## Contract Violations

### 1. Missing Endpoint: POST /orders/{id}/cancel

**Severity**: CRITICAL
**Design Spec**:
```yaml
/orders/{id}/cancel:
  post:
    summary: Cancel an order
    ...
```

**Recommendation**: Implement OrdersController.Cancel(Guid id)

---

### 2. Schema Mismatch: OrderResponse.status

**Severity**: HIGH
**Design Spec**: string (enum: pending, confirmed, shipped)
**Implementation**: integer

**Recommendation**: Change OrderResponse.Status to use OrderStatus enum

---

## Warnings

### 1. Extra Endpoint: GET /health

**Classification**: Internal endpoint
**Action**: Consider adding to design spec or excluding from comparison

---

## Matching Endpoints (15)

| Endpoint | Methods | Params | Request | Response |
|----------|---------|--------|---------|----------|
| /customers | GET, POST | OK | OK | OK |
| /customers/{id} | GET, PUT, DELETE | OK | OK | OK |
| /orders | GET, POST | OK | OK | OK |
| ... | ... | ... | ... | ... |

## Recommendations

1. Add missing endpoint: POST /orders/{id}/cancel
2. Fix OrderResponse.status type mismatch
3. Review if GET /health should be in design spec
```

## Tools Available

- **Read**: Access spec files and source code
- **Glob**: Find project files and assemblies
- **Grep**: Search for controller definitions
- **Bash**: Run dotnet commands and Swashbuckle CLI
- **Write**: Generate report files

## Error Handling

### Build Failure
```
Action: Report build errors
Output: List compilation issues
Recommendation: Fix build before contract testing
```

### Swashbuckle Not Configured
```
Action: Check Program.cs for AddSwaggerGen
Output: Configuration guidance
Recommendation: Add Swashbuckle to project
```

### Design Spec Missing
```
Action: Report missing file
Output: Expected location (docs/openapi.yaml)
Recommendation: Create design-first spec
```

### Invalid Spec Format
```
Action: Report parse errors
Output: Validation error details
Recommendation: Fix spec syntax
```

## Integration Points

**Triggers from:**
- `/contract-test` command
- CI/CD pipeline
- Pre-deployment checks

**May invoke:**
- `api-diff-analyzer` agent for detailed comparison
- `controller-generator` skill for fix recommendations
- `api-spec-interpreter` for schema analysis

**Outputs to:**
- `reports/contract-test-report.md`
- `reports/contract-diff.json` (optional)
- Console summary

## Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `--design-spec` | docs/openapi.yaml | Path to design-first spec |
| `--output` | reports/contract-test-report.md | Report output path |
| `--fail-on-warn` | false | Treat warnings as failures |
| `--ignore-internal` | true | Ignore /health, /metrics endpoints |
| `--strict` | false | Fail on any difference |

## Best Practices

1. **Run early and often**: Catch drift before it accumulates
2. **Automate in CI**: Fail PRs that violate contract
3. **Review warnings**: Don't ignore non-critical differences
4. **Update design spec deliberately**: Changes should be intentional
5. **Document exceptions**: Some differences are acceptable
6. **Track generated spec**: Version control for historical comparison

## Example Session

```
User: /contract-test

Agent:
1. Building project...
   ✓ Build succeeded

2. Generating code-first spec...
   ✓ Generated swagger-generated.json

3. Loading design spec...
   ✓ Loaded docs/openapi.yaml

4. Comparing specifications...
   Checking 18 endpoints...
   Checking 12 schemas...

5. Contract Test Results:
   ❌ FAIL - 3 violations found

   Violations:
   - Missing: POST /orders/{id}/cancel
   - Mismatch: OrderResponse.status (string→int)
   - Mismatch: CustomerRequest.phone (missing)

   Full report: reports/contract-test-report.md
```

## Notes

- This agent performs READ and ANALYSIS operations primarily
- Build and spec generation require Bash tool
- Report generation uses Write tool
- Comparison logic is performed by Claude analysis
- For automated CI, consider dedicated diff tools (oasdiff)
