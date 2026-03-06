# Contract Testing Quick Reference

Fast lookup guide for API contract testing workflow.

## Quick Start

```bash
# Run contract test
/contract-test

# With custom spec location
/contract-test --design-spec=specs/api.yaml

# Strict mode (any difference = fail)
/contract-test --strict
```

## What Is Contract Testing?

```
┌─────────────────────┐      ┌─────────────────────┐
│  Design-First Spec  │      │   Implementation    │
│  (docs/openapi.yaml)│      │   (Controllers/DTOs)│
└──────────┬──────────┘      └──────────┬──────────┘
           │                            │
           │                            ▼
           │                  ┌─────────────────────┐
           │                  │  Code-First Spec    │
           │                  │  (Swashbuckle gen)  │
           │                  └──────────┬──────────┘
           │                             │
           ▼                             ▼
         ┌───────────────────────────────────┐
         │         COMPARISON                │
         │  Design Spec vs Code Spec         │
         └───────────────┬───────────────────┘
                         │
                         ▼
         ┌───────────────────────────────────┐
         │      Contract Test Report         │
         │  • Violations (must fix)          │
         │  • Warnings (review)              │
         │  • Matches (validated)            │
         └───────────────────────────────────┘
```

## Prerequisites Checklist

- [ ] Swashbuckle.AspNetCore installed
- [ ] `builder.Services.AddSwaggerGen()` in Program.cs
- [ ] Design-first spec at `docs/openapi.yaml`
- [ ] Project builds successfully (`dotnet build`)

## Violation Categories

| Category | Severity | Action Required |
|----------|----------|-----------------|
| Missing Endpoint | CRITICAL | Implement endpoint |
| Missing Required Field | CRITICAL | Add field to DTO |
| Type Mismatch | HIGH | Fix property type |
| Extra Optional Field | LOW | Review if intentional |
| Extra Endpoint | INFO | Add to spec or ignore |

## Common Violations & Fixes

### Missing Endpoint

**Problem**: Design spec has endpoint, implementation doesn't.

```yaml
# Design spec
/orders/{id}/cancel:
  post:
    summary: Cancel order
```

**Fix**: Add to controller
```csharp
[HttpPost("orders/{id}/cancel")]
public async Task<IActionResult> Cancel(Guid id)
{
    await _orderService.CancelAsync(id);
    return StatusCode(201);
}
```

### Type Mismatch

**Problem**: Property types don't match.

```
Design: status (string enum: pending, confirmed)
Implementation: Status (int)
```

**Fix**: Use enum type
```csharp
public OrderStatus Status { get; set; }

public enum OrderStatus { Pending, Confirmed, Shipped }
```

### Missing Required Field

**Problem**: Design spec requires field, DTO missing it.

```yaml
# Design spec
CreateOrderRequest:
  required:
    - customerId
    - items
```

**Fix**: Add field to DTO
```csharp
public class CreateOrderRequest
{
    [Required]
    public Guid CustomerId { get; set; }

    [Required]
    public List<OrderItemRequest> Items { get; set; }
}
```

### Validation Mismatch

**Problem**: Validation rules differ.

```yaml
# Design spec
name:
  type: string
  minLength: 1
  maxLength: 100
```

**Fix**: Add validation attributes
```csharp
[Required]
[StringLength(100, MinimumLength = 1)]
public string Name { get; set; }
```

## Command Options

| Option | Description |
|--------|-------------|
| `--design-spec=<path>` | Custom spec location |
| `--output=<path>` | Report output path |
| `--strict` | Any difference = fail |
| `--fail-on-warn` | Warnings cause failure |
| `--include-internal` | Include /health, etc. |
| `--verbose` | Show all comparisons |

## Workflow Integration

### Development Workflow

```
1. Update docs/openapi.yaml (design changes)
2. Implement changes (/implement-endpoint)
3. Run /contract-test
4. Fix violations
5. Commit when PASS
```

### CI/CD Integration

```yaml
# GitHub Actions
- name: Contract Test
  run: |
    dotnet build
    swagger tofile --output generated.json ./bin/Debug/net8.0/MyApi.dll v1
    oasdiff diff docs/openapi.yaml generated.json
```

## Swashbuckle Setup

### Minimal Configuration

```csharp
// Program.cs
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "My API",
        Version = "v1"
    });
});

app.UseSwagger();
```

### With XML Comments

```csharp
builder.Services.AddSwaggerGen(c =>
{
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    c.IncludeXmlComments(xmlPath);
});
```

### CLI Tool Installation

```bash
# Install Swashbuckle CLI
dotnet tool install --global Swashbuckle.AspNetCore.Cli

# Generate spec
swagger tofile --output swagger.json ./bin/Debug/net8.0/MyApi.dll v1
```

## Acceptable Differences

These are typically OK to ignore:

| Difference | Reason |
|------------|--------|
| GET /health | Internal endpoint |
| GET /metrics | Internal endpoint |
| Server URLs | Environment-specific |
| Contact/License info | Metadata only |
| Example values | Documentation only |

## Troubleshooting

| Error | Solution |
|-------|----------|
| Build failed | Fix compilation errors first |
| Swashbuckle not found | Install package, configure Program.cs |
| Design spec missing | Create docs/openapi.yaml |
| Invalid YAML | Validate with linter |
| Assembly not found | Build project first |

## Related Commands

| Command | Use When |
|---------|----------|
| `/compare-api-specs` | Compare two spec files directly |
| `/implement-endpoint` | Add missing endpoint |
| `/regen-api-layer` | Regenerate from spec |
| `/analyze-endpoint` | Understand endpoint complexity |

## Quick Decision Matrix

```
Design spec change needed?
├── YES: Update docs/openapi.yaml first
│        Then implement, then contract test
│
└── NO: Implementation bug
        Fix code to match existing spec
```

## Best Practices

1. **Design-first**: Spec is source of truth
2. **Test early**: Run contract tests before PRs
3. **Fix immediately**: Don't let drift accumulate
4. **Review warnings**: They may reveal spec gaps
5. **Automate**: Include in CI pipeline
6. **Document exceptions**: Some differences are intentional
