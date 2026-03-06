# Claude Code Guides

This directory contains Claude-optimized reference guides for fast decision-making and code generation.

## Purpose

These guides are structured for efficient parsing by Claude Code with:
- Decision trees and flowcharts
- Quick lookup tables
- Pattern matching syntax
- Validation checklists
- Code templates

## Available Guides

### [database-design-rules.md](database-design-rules.md)
Fast reference for database schema design decisions.

**Use when:**
- Creating new tables
- Deciding on column types
- Adding audit columns
- Implementing soft deletes
- Validating schema design

**Key sections:**
- Table naming rules
- Required audit columns
- Soft delete patterns
- Column type rules
- Normalization patterns
- Quick validation checklist

### [primary-key-decision-matrix.md](primary-key-decision-matrix.md)
Decision tree for choosing between INTEGER and UUID primary keys.

**Use when:**
- Creating new entities
- Designing API endpoints
- Evaluating security requirements
- Optimizing performance

**Key sections:**
- Quick decision algorithm
- Comparison matrix
- Use case examples
- UUID version selection
- Hybrid approach patterns
- Implementation templates
- Migration strategies

### [devops-quick-reference.md](devops-quick-reference.md)
Fast lookup for Azure DevOps pipeline configuration and troubleshooting.

**Use when:**
- Setting up CI/CD pipelines
- Configuring NuGet feeds
- Troubleshooting build failures
- Creating Dockerfiles
- Managing secrets
- Deploying to environments

**Key sections:**
- Required files checklist
- Variable reference syntax
- Secret variable mapping
- NuGet feed setup
- Pipeline structure templates
- Dockerfile templates
- Troubleshooting guide
- Pre-deployment checklist

### [code-examples-quick-reference.md](code-examples-quick-reference.md)
Fast reference for production-quality code patterns and project structures.

**Use when:**
- Creating services with CRUD operations
- Implementing PATCH endpoints
- Building query filters
- Setting up project structure
- Configuring Program.cs/Startup
- Working with private NuGet feeds

**Key sections:**
- Base Entity Service pattern
- PATCH operation implementation
- Query extension methods
- Simple vs Clean Architecture structures
- Program.cs configuration order
- NuGet configuration patterns
- Quick decision tree

### [bunzl-api-middleware-quick-reference.md](bunzl-api-middleware-quick-reference.md)
Fast reference for Bunzl.APIMiddleware - company-wide standard for error handling and authentication.

**Use when:**
- Creating new .NET API projects
- Configuring exception handling
- Setting up authentication with Azure AD
- Implementing protected endpoints
- Throwing exceptions from services

**Key sections:**
- Installation and setup
- Program.cs configuration order (CRITICAL)
- Exception usage patterns (service vs controller)
- Available exceptions (404, 409, 400, 401, 500)
- Authentication and token claims
- Swagger configuration with auth
- Common mistakes to avoid

**CRITICAL**: This middleware MUST be included in EVERY .NET API project.

## Relationship to Human Documentation

These guides complement the human-readable documentation in [docs/guides/](../../docs/guides/):

| Claude Guide | Human Documentation | Purpose |
|--------------|-------------------|---------|
| database-design-rules.md | database_design_guide.md | Quick decision trees vs detailed explanations |
| primary-key-decision-matrix.md | pk_type_decision_guide.md | Fast lookups vs comprehensive rationale |
| devops-quick-reference.md | azure_devops_guide.md | Troubleshooting focus vs step-by-step setup |
| code-examples-quick-reference.md | docs/examples/*.md | Fast pattern lookup vs full code examples |
| bunzl-api-middleware-quick-reference.md | bunzl_api_middleware_guide.md | Fast implementation vs detailed usage guide |

**When to use which:**
- **Claude guides** - During code generation, fast decisions, pattern matching
- **Human docs** - Learning, onboarding, understanding context, policy reference

## Format Conventions

All Claude guides follow these conventions:

### Decision Trees
```
START
  ├─ Condition?
  │  ├─ YES → Action
  │  └─ NO → Next condition
```

### Quick Lookups
```
WHEN: Triggering condition
USE: Recommended action
EXAMPLE: Code sample
```

### Validation Checklists
```
VALIDATION:
☐ Item to check
☐ Another item
☐ Final item
```

### Anti-Patterns
```
✗ BAD: What not to do
✓ GOOD: What to do instead
```

### Code Templates
```language
// Commented template code
// With clear placeholders
```

## Maintenance

When updating these guides:

1. Keep decision trees simple (max 4 levels deep)
2. Use consistent formatting for fast parsing
3. Include code examples for common patterns
4. Update both Claude guide and human docs if policy changes
5. Test with actual Claude Code workflows
6. Keep validation checklists comprehensive but concise

## Adding New Guides

New guides should follow this template structure:

```markdown
# Guide Title

Quick reference for [purpose].

## Quick Decision Algorithm
[Decision tree or flowchart]

## Pattern Templates
[Common code patterns with examples]

## Validation Checklist
[Items to verify before completion]

## Troubleshooting
[Common issues and solutions]

## Examples
[Real-world use cases]
```
