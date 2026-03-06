# Spec Change Planner Skill

## Overview

Analyzes API specification changes and creates detailed implementation plans with task ordering, dependency management, and risk mitigation.

## Skill Type

Planning and analysis skill that bridges API diff analysis and implementation.

## When to Use

- After running API spec comparison
- Before implementing spec changes
- When updating existing APIs
- For API versioning decisions
- Planning backwards-compatible migrations

## Input Requirements

```yaml
inputs:
  baseline_spec: path/to/original-spec.yaml
  updated_spec: path/to/updated-spec.yaml
  diff_report: path/to/diff-report.md  # Optional, will generate if not provided
  project_context:
    codebase_path: path/to/codebase
    architecture_pattern: service-repository | cqrs | clean-architecture
    database: postgres | sqlserver | mysql
```

## Outputs

```yaml
outputs:
  implementation_plan:
    phases: [...]
    tasks: [...]
    dependencies: [...]
    risk_mitigation: [...]

  versioning_strategy:
    requires_new_version: boolean
    approach: none | header | url | media-type

  migration_guide:
    breaking_changes: [...]
    client_updates: [...]
    timeline: [...]
```

## Core Capabilities

### 1. Change Impact Analysis

Analyzes how spec changes affect:
- Database schema
- Entity models
- DTOs and mapping
- Service layer logic
- Controller endpoints
- Validation rules
- Authentication/authorization
- External dependencies

### 2. Task Dependency Resolution

Creates proper task ordering:
```
DEPENDENCY RULES:
1. Database schema changes BEFORE entity updates
2. Entity updates BEFORE DTO changes
3. DTO changes BEFORE service layer
4. Service layer BEFORE controllers
5. All implementation BEFORE tests
6. Tests BEFORE deployment
```

### 3. Breaking Change Mitigation

Strategies for handling breaking changes:
- API versioning (v1 → v2)
- Deprecation periods
- Adapter patterns
- Dual-write patterns
- Feature flags
- Gradual rollout

### 4. Implementation Planning

Generates:
- Ordered task list
- File-level changes
- Code generation commands
- Test requirements
- Migration scripts
- Rollback procedures

## Implementation Patterns

### Pattern 1: Non-Breaking Addition

```yaml
scenario: New optional field in existing DTO
tasks:
  1. Add column to database (nullable)
  2. Update entity model
  3. Update DTO
  4. Update AutoMapper profile
  5. Add service layer logic
  6. Update tests
  7. Update API documentation

risk: LOW
versioning: Not required
rollback: Simple (remove column)
```

### Pattern 2: Breaking Field Removal

```yaml
scenario: Remove required field from DTO
tasks:
  1. Create new API version (v2)
  2. Implement v2 endpoint (without field)
  3. Mark v1 as deprecated
  4. Update client documentation
  5. Set deprecation timeline (90 days)
  6. Monitor v1 usage
  7. Remove v1 after timeline

risk: HIGH
versioning: Required (v1 + v2)
rollback: Keep v1 running
```

### Pattern 3: New Endpoint

```yaml
scenario: Add new GET /orders/{id}/items endpoint
tasks:
  1. Analyze data model (Order → OrderItem relationship)
  2. Generate entity if needed
  3. Generate DTO (OrderItemResponse)
  4. Generate AutoMapper profile
  5. Generate repository method
  6. Generate service method
  7. Generate controller action
  8. Generate unit tests
  9. Update Swagger docs

risk: LOW
versioning: Not required
rollback: Remove endpoint
```

### Pattern 4: Endpoint Modification

```yaml
scenario: Change POST /orders request schema
tasks:
  1. Assess breaking vs non-breaking
  2. IF BREAKING:
       - Create v2 endpoint
       - Implement new schema
       - Deprecate v1
     ELSE:
       - Update existing DTO
       - Update validation
       - Update service
  3. Update tests
  4. Update documentation

risk: VARIABLE (depends on change type)
versioning: If breaking, required
rollback: Version-specific
```

## Task Template

Each task includes:

```yaml
task:
  id: TASK-001
  title: Add IsActive column to Order table
  type: database | entity | dto | service | controller | test
  priority: critical | high | medium | low
  dependencies: []
  estimated_complexity: trivial | simple | moderate | complex

  description: |
    Add IsActive boolean column to Order table for soft delete support

  files_affected:
    - path/to/migration.sql
    - path/to/Order.cs

  commands:
    - /generate-sql --table Order --add-column IsActive:boolean

  validation:
    - Database migration applies successfully
    - Entity model compiles
    - No breaking changes to existing queries

  rollback:
    - Drop column from Order table
    - Revert entity model changes
```

## Risk Mitigation Strategies

### Critical Risk Changes

```yaml
mitigation:
  - Create feature flag for new behavior
  - Implement dual-write pattern
  - Deploy in phases (canary → staged → full)
  - Maintain v1 and v2 simultaneously
  - Set up monitoring and alerts
  - Prepare rollback scripts
  - Communicate breaking changes to clients
```

### High Risk Changes

```yaml
mitigation:
  - Create comprehensive test suite
  - Run integration tests
  - Test with production-like data
  - Deploy to staging first
  - Monitor error rates post-deployment
  - Have rollback plan ready
```

### Medium Risk Changes

```yaml
mitigation:
  - Add unit tests for new behavior
  - Update integration tests
  - Code review before merge
  - Staging deployment verification
```

### Low Risk Changes

```yaml
mitigation:
  - Standard unit tests
  - Code review
  - Normal deployment process
```

## Versioning Strategies

### URL Versioning

```yaml
approach: URL-based versioning
pattern:
  v1: /v1/orders
  v2: /v2/orders

implementation:
  - Create separate controller for v2
  - Share service layer where possible
  - Maintain both versions
  - Set deprecation date for v1

pros:
  - Clear and explicit
  - Easy to understand
  - Simple routing

cons:
  - URL pollution
  - Duplicate code potential
```

### Header Versioning

```yaml
approach: Header-based versioning
pattern:
  v1: Header "API-Version: 1"
  v2: Header "API-Version: 2"

implementation:
  - Single controller with version detection
  - Conditional logic based on header
  - Default to latest if not specified

pros:
  - Clean URLs
  - Flexible versioning

cons:
  - Hidden from URL
  - More complex routing
```

### No Versioning (Backwards Compatible)

```yaml
approach: Maintain backwards compatibility
pattern:
  - Only additive changes
  - Optional fields only
  - No removals

implementation:
  - Add new optional fields
  - Never remove fields (deprecate instead)
  - Use sensible defaults

pros:
  - No version management
  - Simple for clients

cons:
  - Technical debt accumulation
  - Complex DTOs over time
```

## Usage Examples

### Example 1: Field Addition

```yaml
input:
  change: Add "priority" field to CreateOrderRequest
  type: optional string enum

analysis:
  breaking: false
  requires_version: false

plan:
  tasks:
    - Add Priority column to Order table (nullable)
    - Add Priority property to Order entity
    - Add Priority property to CreateOrderRequest DTO
    - Update AutoMapper profile
    - Add Priority enum
    - Update CreateOrder service method
    - Add unit tests for priority handling
    - Update Swagger documentation

  risk: LOW
  estimated_effort: 2-3 hours
```

### Example 2: Endpoint Removal

```yaml
input:
  change: Remove GET /orders/legacy endpoint
  type: endpoint deletion

analysis:
  breaking: true
  requires_version: true

plan:
  approach: Gradual deprecation
  tasks:
    - Mark endpoint as [Obsolete] with message
    - Add deprecation warning to response headers
    - Update documentation with replacement endpoint
    - Set sunset date (90 days)
    - Monitor usage metrics
    - Send client notifications
    - Remove endpoint after sunset

  risk: HIGH
  estimated_effort: 1 week (including deprecation period)
```

### Example 3: Schema Restructuring

```yaml
input:
  change: Split Order into Order + OrderItem entities
  type: schema restructuring

analysis:
  breaking: potentially
  requires_version: depends on API changes

plan:
  phase_1_database:
    - Create OrderItem table
    - Migrate existing Order.Items to OrderItem table
    - Add foreign key Order.Id → OrderItem.OrderId
    - Validate data migration

  phase_2_entities:
    - Create OrderItem entity
    - Update Order entity (remove items collection initially)
    - Add navigation properties

  phase_3_api:
    - IF existing API returns items: Keep v1, create v2
    - Create new endpoints for OrderItem CRUD
    - Update DTOs and mappings

  phase_4_testing:
    - Unit tests for new entities
    - Integration tests for new endpoints
    - Migration verification tests

  risk: HIGH
  estimated_effort: 1-2 weeks
```

## Integration with Commands

This skill works with these commands:

```bash
# Compare specs and generate plan
/compare-api-specs baseline.yaml updated.yaml

# Plan implementation from diff report
/plan-spec-changes diff-report.md

# Execute implementation plan
/implement-spec-changes implementation-plan.md

# Generate specific changes
/generate-db-schema Order
/generate-dtos OrderItem
/generate-controller OrdersController
```

## Best Practices

1. **Analyze Before Planning**
   - Always compare specs first
   - Understand all changes before creating plan
   - Identify dependencies early

2. **Prioritize Breaking Changes**
   - Handle breaking changes first
   - Decide on versioning early
   - Communicate to stakeholders

3. **Order Tasks Correctly**
   - Database → Entities → DTOs → Services → Controllers
   - Tests after each layer
   - Documentation last

4. **Consider Rollback**
   - Every task needs rollback plan
   - Test rollback procedures
   - Document rollback steps

5. **Validate Assumptions**
   - Check codebase structure
   - Verify architecture patterns
   - Confirm database capabilities

6. **Communicate Changes**
   - Document breaking changes
   - Update API documentation
   - Notify API consumers

## Error Handling

```yaml
error_scenarios:
  invalid_spec:
    action: Validate spec files first
    message: "Invalid OpenAPI specification"

  circular_dependencies:
    action: Detect and report
    message: "Circular dependency detected in task graph"

  missing_context:
    action: Request additional information
    message: "Need project context (architecture, database, etc.)"

  incompatible_change:
    action: Flag as critical risk
    message: "Change may be incompatible with current architecture"
```

## Output Artifacts

Generated files:

1. **implementation-plan.md**: Detailed task list with dependencies
2. **migration-guide.md**: Developer instructions
3. **client-migration-guide.md**: API consumer instructions
4. **rollback-plan.md**: Emergency rollback procedures
5. **tasks.json**: Machine-readable task manifest
6. **risk-assessment.md**: Risk analysis and mitigation

## Notes

- This skill focuses on PLANNING, not implementation
- Always validate plan with user before execution
- Complex changes may require multiple phases
- Consider team capacity when estimating effort
- Breaking changes should always trigger review process
