# API Spec Evolution Guide

Quick reference for managing API specification changes and implementing them systematically.

## Overview

This guide covers the complete workflow from API spec update to implementation, including change detection, risk assessment, and systematic implementation.

## When to Use This Workflow

```
TRIGGER: API specification updated
SCENARIOS:
- New version of OpenAPI/Swagger spec
- New endpoints added
- Existing endpoints modified
- Schemas/DTOs changed
- Breaking changes introduced
- API versioning required
```

## Complete Workflow

```
SPEC UPDATE WORKFLOW:

1. COMPARE SPECS
   ↓ /compare-api-specs baseline.yaml updated.yaml
   ↓
2. REVIEW DIFF REPORT
   ↓ Analyze changes, assess risk
   ↓
3. PLAN IMPLEMENTATION
   ↓ Review auto-generated plan
   ↓
4. IMPLEMENT CHANGES
   ↓ /implement-spec-changes plan.md
   ↓
5. VALIDATE
   ↓ Tests, compilation, manual checks
   ↓
6. DEPLOY
   ↓ PR, staging, production
```

## Phase 1: Compare Specifications

### Command

```bash
/compare-api-specs specs/v1.yaml specs/v2.yaml --plan
```

### What It Does

```
COMPARISON:
✓ Loads both spec files
✓ Validates OpenAPI format
✓ Compares endpoints (added/removed/modified)
✓ Compares schemas (DTOs, models)
✓ Compares parameters (query, path, header)
✓ Classifies changes (breaking vs non-breaking)
✓ Assesses risk level
✓ Generates implementation plan

OUTPUTS:
- api-diff-report.md
- breaking-changes.md
- implementation-plan.md
- migration-guide.md (for API consumers)
```

### Change Classification

```yaml
BREAKING CHANGES (Require versioning):
  - Endpoint removal
  - Required field removal
  - Type change (incompatible)
  - Required parameter addition
  - Enum value removal

NON-BREAKING CHANGES (Safe to add):
  - New endpoint
  - Optional field addition
  - Optional parameter addition
  - New enum value
  - Documentation update
```

### Risk Levels

```
CRITICAL (API versioning required):
- Multiple breaking changes
- Endpoint removals
- Contract incompatibilities
→ Action: Create v2 API

HIGH (Careful migration):
- Single breaking change
- Database schema changes
→ Action: Gradual migration

MEDIUM (Testing required):
- Non-breaking with logic changes
- New features
→ Action: Standard deployment

LOW (Safe deployment):
- Optional additions
- Documentation only
→ Action: Deploy directly
```

## Phase 2: Review Diff Report

### Key Sections to Review

```markdown
1. SUMMARY
   - Total changes
   - Breaking vs non-breaking
   - Risk level

2. ENDPOINTS
   - Added endpoints
   - Removed endpoints (⚠️ breaking)
   - Modified endpoints

3. SCHEMAS
   - New DTOs
   - Modified DTOs
   - Removed DTOs (⚠️ breaking)

4. BREAKING CHANGES
   - Severity assessment
   - Impact analysis
   - Mitigation strategies

5. VERSIONING RECOMMENDATION
   - Whether versioning is needed
   - Recommended approach
   - Timeline

6. IMPLEMENTATION PLAN
   - Task list
   - Phase ordering
   - Dependencies
```

### Decision Tree

```
START: Review diff report
  ↓
[Breaking changes?]
  ├─ NO → Approve for direct implementation
  │       ↓
  │       Proceed to Phase 3
  │
  └─ YES → [How many breaking changes?]
           ├─ One → Consider gradual migration
           │        ↓
           │        Evaluate if versioning needed
           │        ↓
           │        Make decision
           │
           └─ Multiple → API versioning REQUIRED
                         ↓
                         Plan v1 + v2 approach
                         ↓
                         Set deprecation timeline
```

## Phase 3: Plan Review and Approval

### Implementation Plan Structure

```yaml
phases:
  1_database:
    tasks: [...]
    validation: SQL compiles, migrations work

  2_entities:
    tasks: [...]
    validation: Entities compile, DbContext valid

  3_dtos:
    tasks: [...]
    validation: DTOs compile, mappings valid

  4_services:
    tasks: [...]
    validation: Services compile, DI works

  5_controllers:
    tasks: [...]
    validation: Controllers compile, routes don't conflict

  6_tests:
    tasks: [...]
    validation: All tests pass

  7_documentation:
    tasks: [...]
    validation: Docs accurate
```

### Task Dependencies

```
EXECUTION ORDER (strict):

Database
  ↓
Entities
  ↓
DTOs + Mapping
  ↓
Repositories
  ↓
Services
  ↓
Controllers
  ↓
Tests
  ↓
Documentation

REASON: Each layer depends on the previous layer
```

### Approval Checklist

```
BEFORE APPROVING PLAN:
☐ Understand all breaking changes
☐ Versioning strategy is clear
☐ Timeline is reasonable
☐ Dependencies are correct
☐ Rollback plan exists
☐ Team is informed
☐ API consumers notified (if breaking)
```

## Phase 4: Implementation

### Command

```bash
/implement-spec-changes reports/implementation-plan.md
```

### Execution Modes

```bash
# Standard execution
/implement-spec-changes plan.md

# Dry run (preview only)
/implement-spec-changes plan.md --dry-run

# Interactive (confirm each task)
/implement-spec-changes plan.md --interactive

# Phase-by-phase
/implement-spec-changes plan.md --phase=database
/implement-spec-changes plan.md --phase=entities
# ... continue through phases

# With auto-commit
/implement-spec-changes plan.md --auto-commit
```

### Implementation Flow

```
FOR EACH PHASE:
  FOR EACH TASK:
    1. Check dependencies satisfied
    2. Execute task (generate/edit code)
    3. Validate output
    4. Run verification
    5. Mark complete

  Phase validation:
    - Compilation check
    - Format check
    - Partial test run

  Optional: Create git commit

ALL PHASES COMPLETE:
  - Run full test suite
  - Run dotnet format
  - Generate summary report
```

## Phase 5: Validation

### Validation Checklist

```bash
# 1. Compilation
dotnet build
→ Must exit with code 0

# 2. Code formatting
dotnet format --verify-no-changes
→ Must exit with code 0

# 3. Unit tests
dotnet test
→ All tests must pass

# 4. API functionality (manual)
- Start application
- Test new endpoints
- Verify responses
- Check breaking change mitigation

# 5. Database (if schema changed)
- Verify migrations apply
- Check data integrity
- Test rollback migration
```

### Common Issues

```
COMPILATION ERRORS:
- Missing using statements
- Type mismatches
- Circular dependencies
→ Fix before proceeding

TEST FAILURES:
- Update test data
- Fix mock setups
- Add new test cases
→ Never skip failing tests

MAPPING ERRORS:
- AutoMapper config invalid
- Property name mismatches
→ Regenerate mapper profiles

BREAKING CHANGES NOT HANDLED:
- v1 still broken
- Migration path missing
→ Review versioning strategy
```

## Phase 6: Deployment

### Pre-Deployment

```bash
# Create feature branch
git checkout -b feat/api-v2-updates

# Review all changes
git diff main

# Create PR
/create-pr
```

### Deployment Strategy

```yaml
NON-BREAKING CHANGES:
  strategy: Direct deployment
  steps:
    - Deploy to staging
    - Smoke test
    - Deploy to production
    - Monitor

BREAKING CHANGES (with versioning):
  strategy: Dual version deployment
  steps:
    - Deploy v2 alongside v1
    - Route new clients to v2
    - Keep v1 active (deprecation period)
    - Monitor v1 usage
    - Sunset v1 after timeline
    - Remove v1 code

GRADUAL MIGRATION (single breaking change):
  strategy: Feature flag or adapter pattern
  steps:
    - Deploy new code with feature flag OFF
    - Test in production (flag OFF)
    - Enable for internal users first
    - Gradual rollout percentage
    - Monitor errors/performance
    - Full rollout
    - Remove feature flag
```

### Post-Deployment

```
MONITORING:
☐ Check error rates
☐ Monitor API latency
☐ Verify new endpoints work
☐ Check v1 usage (if versioned)
☐ Review logs for issues

COMMUNICATION:
☐ Notify stakeholders of deployment
☐ Update API documentation
☐ Publish migration guide (if breaking)
☐ Set deprecation timeline (if applicable)
```

## Versioning Strategies

### URL Versioning (Recommended)

```yaml
pattern: /v{version}/{resource}
examples:
  v1: /v1/orders
  v2: /v2/orders

implementation:
  - Separate controllers for each version
  - Shared service layer (if possible)
  - Maintain both until v1 sunset

pros:
  - Clear and explicit
  - Easy to route
  - Simple for clients

cons:
  - URL changes
  - Code duplication risk
```

### Header Versioning

```yaml
pattern: Header "API-Version: {version}"
examples:
  v1: API-Version: 1
  v2: API-Version: 2

implementation:
  - Single controller with version detection
  - Conditional logic based on header
  - Default to latest version

pros:
  - Clean URLs
  - Flexible

cons:
  - Hidden from URL
  - More complex routing
```

### No Versioning (Backwards Compatible Only)

```yaml
pattern: Single version, only additive changes
approach:
  - Only add optional fields
  - Never remove fields
  - Never break contracts

implementation:
  - Evolve DTOs with optional properties
  - Deprecate but don't remove
  - Use sensible defaults

pros:
  - Simple for clients
  - No version management

cons:
  - Technical debt accumulates
  - Limited flexibility
  - Can't make breaking changes
```

## Breaking Change Patterns

### Pattern 1: Field Removal

```
PROBLEM: Need to remove field from DTO
SOLUTION: API versioning

v1 (deprecated):
  GET /v1/orders → includes oldField

v2 (new):
  GET /v2/orders → excludes oldField

Timeline:
  - Deploy v2
  - Mark v1 deprecated
  - 90-day notice
  - Remove v1
```

### Pattern 2: Required Field Addition

```
PROBLEM: Add required field to request
SOLUTION: Make optional + default OR versioning

Option A (if default makes sense):
  - Add field as optional
  - Use default value if not provided
  - No versioning needed

Option B (if no sensible default):
  - Create v2 with required field
  - Keep v1 with optional field
  - Gradual migration
```

### Pattern 3: Endpoint Removal

```
PROBLEM: Remove endpoint
SOLUTION: Deprecation timeline

Steps:
  1. Mark endpoint [Obsolete]
  2. Add deprecation header to response
  3. Update docs with replacement
  4. Notify API consumers
  5. Monitor usage
  6. Remove after timeline (90+ days)
```

### Pattern 4: Type Change

```
PROBLEM: Change field type (e.g., string → int)
SOLUTION: New field OR versioning

Option A (new field):
  - Add newField with new type
  - Keep oldField (deprecated)
  - Both in same version
  - Eventually remove oldField

Option B (versioning):
  - v1: oldField (string)
  - v2: field (int)
  - Maintain both versions
```

## Quick Reference Commands

```bash
# Compare specs
/compare-api-specs baseline.yaml updated.yaml --plan

# Review what would be done
/implement-spec-changes plan.md --dry-run

# Implement interactively
/implement-spec-changes plan.md --interactive

# Implement specific phase
/implement-spec-changes plan.md --phase=database

# Validate after implementation
dotnet build && dotnet test && dotnet format --verify-no-changes

# Create PR
/create-pr
```

## Best Practices Summary

```
1. ALWAYS compare before implementing
   → Never implement spec changes blindly

2. REVIEW breaking changes carefully
   → Assess client impact first

3. USE versioning for breaking changes
   → Don't break existing clients

4. FOLLOW phase order strictly
   → Database → Entities → DTOs → Services → Controllers → Tests

5. VALIDATE after each phase
   → Catch errors early

6. TEST thoroughly
   → Unit tests + integration tests + manual testing

7. COMMUNICATE breaking changes
   → Notify API consumers with migration guides

8. SET clear deprecation timelines
   → Give clients time to migrate

9. MONITOR after deployment
   → Watch for errors and performance issues

10. DOCUMENT all changes
    → Update API docs and changelogs
```

## Troubleshooting

### Comparison Fails

```
ISSUE: Specs can't be compared
CAUSES:
  - Invalid OpenAPI format
  - Missing required fields
  - Wrong file path

SOLUTIONS:
  - Validate spec with linter
  - Check file exists
  - Verify OpenAPI version compatibility
```

### Implementation Fails

```
ISSUE: Task execution errors
CAUSES:
  - Dependency not satisfied
  - Compilation error
  - Missing files

SOLUTIONS:
  - Check task dependencies
  - Run dotnet build for details
  - Review error messages
  - Use --dry-run first
```

### Tests Fail

```
ISSUE: Tests fail after implementation
CAUSES:
  - Test data outdated
  - Mock setups incorrect
  - Breaking change not handled in tests

SOLUTIONS:
  - Update test data
  - Fix mock expectations
  - Add new test cases
  - NEVER skip failing tests
```

### Breaking Changes Not Mitigated

```
ISSUE: Existing clients break
CAUSES:
  - Versioning not implemented
  - v1 removed too soon
  - Migration guide missing

SOLUTIONS:
  - Implement v1 + v2 simultaneously
  - Extend deprecation timeline
  - Create detailed migration guide
  - Communicate with consumers
```

## Examples

### Example 1: Non-Breaking Addition

```
CHANGE: Add optional "notes" field to Order

WORKFLOW:
1. /compare-api-specs v1.yaml v2.yaml
   → Risk: LOW, no breaking changes

2. Review: Optional field addition, safe

3. /implement-spec-changes plan.md
   → Adds column, entity, DTO, mapping, service, tests

4. dotnet test
   → All pass

5. Deploy
   → Direct to production, no versioning needed
```

### Example 2: Breaking Change with Versioning

```
CHANGE: Remove "legacyId" field from Order

WORKFLOW:
1. /compare-api-specs v1.yaml v2.yaml
   → Risk: CRITICAL, field removal breaking

2. Review: Versioning REQUIRED
   → Strategy: URL versioning (v1 + v2)

3. /implement-spec-changes plan.md
   → Creates v2 controllers/DTOs
   → Marks v1 deprecated
   → Both versions active

4. Test both v1 and v2

5. Deploy with both versions
   → Set 90-day deprecation for v1
   → Monitor v1 usage

6. After 90 days: Remove v1
```

## Related Documentation

- [compare-api-specs command](../commands/compare-api-specs.md)
- [implement-spec-changes command](../commands/implement-spec-changes.md)
- [api-diff-analyzer agent](../agents/api-diff-analyzer.md)
- [spec-change-planner skill](../skills/spec-change-planner/SKILL.md)
