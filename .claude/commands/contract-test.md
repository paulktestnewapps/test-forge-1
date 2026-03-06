# Contract Test Command

## Command

```bash
/contract-test [options]
```

## Purpose

Perform contract testing to validate that API implementation matches the design-first OpenAPI specification. This command:
1. Generates a code-first API spec from implementation (using Swashbuckle)
2. Compares against the design-first spec (docs/openapi.yaml)
3. Reports any discrepancies between contract and implementation

## Usage

### Basic Usage

```bash
# Run contract test with defaults
/contract-test

# Specify custom design spec location
/contract-test --design-spec=specs/api.yaml

# Output report to custom location
/contract-test --output=reports/my-contract-test.md
```

### Advanced Options

```bash
# Strict mode - fail on any difference
/contract-test --strict

# Treat warnings as failures
/contract-test --fail-on-warn

# Include internal endpoints in comparison
/contract-test --include-internal

# Generate JSON diff output
/contract-test --format=json

# Verbose output with all matching endpoints
/contract-test --verbose
```

## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `--design-spec` | path | docs/openapi.yaml | Path to design-first OpenAPI spec |
| `--output` | path | reports/contract-test-report.md | Report output location |
| `--strict` | flag | false | Fail on any difference |
| `--fail-on-warn` | flag | false | Treat warnings as failures |
| `--include-internal` | flag | false | Include /health, /metrics endpoints |
| `--format` | string | markdown | Output format (markdown, json) |
| `--verbose` | flag | false | Show all comparisons including matches |

## Prerequisites

1. **Swashbuckle configured** in your .NET project:
   ```csharp
   // Program.cs
   builder.Services.AddSwaggerGen();
   app.UseSwagger();
   ```

2. **Design-first spec exists** at `docs/openapi.yaml` (or specified path)

3. **Project builds successfully**:
   ```bash
   dotnet build
   ```

## What It Does

### Step 1: Build Project

```
Building project...
✓ Build succeeded (2.3s)
```

Compiles the .NET project to ensure implementation is current.

### Step 2: Generate Code-First Spec

```
Generating OpenAPI spec from implementation...
✓ Generated spec: 18 endpoints, 14 schemas
```

Uses Swashbuckle CLI or runtime to extract OpenAPI spec from code.

### Step 3: Load Design Spec

```
Loading design-first spec...
✓ Loaded docs/openapi.yaml: 20 endpoints, 12 schemas
```

Parses the design-first OpenAPI specification.

### Step 4: Compare Specifications

```
Comparing specifications...
  Endpoints: 18/20 match
  Schemas: 12/14 match
  Parameters: 45/45 match
```

Performs detailed comparison of:
- Endpoint paths and methods
- Request/response schemas
- Parameters and validation rules
- Authentication requirements

### Step 5: Generate Report

```
Generating contract test report...
✓ Report: reports/contract-test-report.md
```

Creates a detailed report with findings and recommendations.

## Output

### Summary (Console)

```
Contract Test Results
━━━━━━━━━━━━━━━━━━━━

Status: FAIL

┌─────────────┬─────────┬────────────┬─────────┐
│ Category    │ Matches │ Mismatches │ Missing │
├─────────────┼─────────┼────────────┼─────────┤
│ Endpoints   │ 17      │ 1          │ 2       │
│ Schemas     │ 11      │ 1          │ 0       │
│ Parameters  │ 45      │ 0          │ 0       │
└─────────────┴─────────┴────────────┴─────────┘

Violations (3):
  ❌ Missing endpoint: POST /orders/{id}/cancel
  ❌ Missing endpoint: DELETE /orders/{id}
  ❌ Type mismatch: OrderResponse.status (expected: string enum, actual: int)

Warnings (1):
  ⚠️  Extra endpoint: GET /health (not in design spec)

Report: reports/contract-test-report.md
```

### Full Report (Markdown)

The generated report includes:

1. **Executive Summary**: Pass/fail status with metrics
2. **Contract Violations**: Missing or mismatched implementations
3. **Warnings**: Extra implementations, minor differences
4. **Matching Items**: Successfully validated endpoints/schemas
5. **Recommendations**: How to fix each violation

## Examples

### Example 1: All Tests Pass

```bash
$ /contract-test

Contract Test Results
━━━━━━━━━━━━━━━━━━━━

Status: PASS ✓

All 18 endpoints match design specification.
All 12 schemas validated successfully.

No contract violations detected.

Report: reports/contract-test-report.md
```

### Example 2: Violations Detected

```bash
$ /contract-test

Contract Test Results
━━━━━━━━━━━━━━━━━━━━

Status: FAIL ✗

Violations (2):

1. Missing Endpoint: POST /orders/{id}/cancel
   Design spec defines endpoint but implementation missing.

   Recommendation:
   Add to OrdersController:
   [HttpPost("orders/{id}/cancel")]
   public async Task<IActionResult> Cancel(Guid id)

2. Schema Mismatch: CreateOrderRequest.priority
   Design: string (enum: low, medium, high)
   Implementation: integer

   Recommendation:
   Change property type to enum or string

Report: reports/contract-test-report.md
```

### Example 3: Strict Mode

```bash
$ /contract-test --strict

Contract Test Results
━━━━━━━━━━━━━━━━━━━━

Status: FAIL ✗ (strict mode)

Differences (3):
  • Extra endpoint: GET /health
  • Extra schema: HealthResponse
  • Documentation differs: GET /customers description

In strict mode, any difference is a failure.
Consider adding these to design spec or use --ignore-patterns.
```

## Integration with CI/CD

### GitHub Actions

```yaml
- name: Contract Test
  run: |
    # Run contract test command via Claude Code
    claude code "/contract-test --fail-on-warn"
```

### Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-push

echo "Running contract tests..."
claude code "/contract-test"
if [ $? -ne 0 ]; then
    echo "Contract test failed. Push aborted."
    exit 1
fi
```

## Workflow Integration

This command fits into the API development workflow:

```
Design Phase:
  1. Create/update docs/openapi.yaml (design-first)

Implementation Phase:
  2. /implement-api or /implement-endpoint
  3. Write controller and service code

Validation Phase:
  4. /contract-test ← YOU ARE HERE
  5. Fix any violations
  6. Repeat until PASS

Deployment Phase:
  7. Commit and deploy
```

## Troubleshooting

### Build Fails

```
Error: Build failed with errors
Solution: Run 'dotnet build' and fix compilation errors first
```

### Swashbuckle Not Found

```
Error: Cannot generate code-first spec - Swashbuckle not configured
Solution: Add Swashbuckle.AspNetCore package and configure in Program.cs
```

### Design Spec Not Found

```
Error: Design spec not found at docs/openapi.yaml
Solution: Create spec or use --design-spec to specify location
```

### Invalid Spec Format

```
Error: Failed to parse docs/openapi.yaml - invalid YAML
Solution: Validate spec with a linter (e.g., spectral lint docs/openapi.yaml)
```

## Behind the Scenes

When you run this command, Claude Code:

1. **Invokes contract-test-agent** to orchestrate the process
2. **Builds the project** using `dotnet build`
3. **Generates code-first spec** using Swashbuckle CLI
4. **Loads both specifications** and parses OpenAPI format
5. **Performs deep comparison** of endpoints, schemas, parameters
6. **Classifies differences** as violations, warnings, or matches
7. **Generates report** with findings and recommendations

## Best Practices

1. **Run before committing**: Catch contract drift early
2. **Include in PR checks**: Automated contract validation
3. **Fix violations immediately**: Don't let drift accumulate
4. **Update design spec deliberately**: Changes should be intentional
5. **Review warnings**: They may indicate missing spec updates
6. **Use strict mode in CI**: Ensure complete alignment

## Related Commands

```bash
# Compare two spec files directly
/compare-api-specs docs/openapi.yaml generated-spec.json

# Implement missing endpoints
/implement-endpoint POST /orders/{id}/cancel

# Regenerate API layer from spec
/regen-api-layer

# Analyze specific endpoint
/analyze-endpoint GET /customers
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | PASS - No violations |
| 1 | FAIL - Contract violations found |
| 2 | ERROR - Build or generation failed |

## Notes

- Always runs a fresh build before generating code-first spec
- Compares against design-first spec as source of truth
- Internal endpoints (/health, /metrics) are ignored by default
- Report is generated even when tests pass (for documentation)
- JSON output format available for CI/CD integration
