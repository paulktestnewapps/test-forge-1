# Compare API Specs Command

## Command

```bash
/compare-api-specs <baseline-spec> <updated-spec> [options]
```

## Purpose

Compare two OpenAPI/Swagger specification files to identify changes, additions, deletions, and modifications. Generates a detailed diff report and implementation plan.

## Usage

### Basic Usage

```bash
# Compare two spec files
/compare-api-specs specs/v1.yaml specs/v2.yaml

# Compare with custom output location
/compare-api-specs specs/v1.yaml specs/v2.yaml --output=reports/api-diff.md

# Compare and auto-generate implementation plan
/compare-api-specs specs/v1.yaml specs/v2.yaml --plan
```

### Advanced Options

```bash
# Focus on breaking changes only
/compare-api-specs specs/v1.yaml specs/v2.yaml --breaking-only

# Include codebase analysis for impact assessment
/compare-api-specs specs/v1.yaml specs/v2.yaml --analyze-impact

# Generate migration guide for clients
/compare-api-specs specs/v1.yaml specs/v2.yaml --client-guide

# Specify project architecture for better recommendations
/compare-api-specs specs/v1.yaml specs/v2.yaml --architecture=service-repository

# Compare with specific focus areas
/compare-api-specs specs/v1.yaml specs/v2.yaml --focus=endpoints,schemas
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `baseline-spec` | path | Yes | Path to original/baseline API spec |
| `updated-spec` | path | Yes | Path to updated/new API spec |
| `--output` | path | No | Custom output path for diff report |
| `--plan` | flag | No | Auto-generate implementation plan |
| `--breaking-only` | flag | No | Show only breaking changes |
| `--analyze-impact` | flag | No | Analyze impact on existing codebase |
| `--client-guide` | flag | No | Generate migration guide for API consumers |
| `--architecture` | string | No | Specify project architecture pattern |
| `--focus` | string | No | Focus on specific areas (endpoints, schemas, etc.) |

## What It Does

### 1. Loads and Validates Specs

- Reads both specification files
- Validates OpenAPI/Swagger format
- Ensures both specs are comparable
- Extracts version information

### 2. Compares Endpoints

Identifies:
- New endpoints (additions)
- Removed endpoints (deletions)
- Modified endpoint paths
- HTTP method changes
- Parameter changes (query, path, header)
- Request body changes
- Response changes (status codes, schemas)

### 3. Compares Schemas

Identifies:
- New schemas/DTOs
- Removed schemas
- Modified schemas:
  - Added fields
  - Removed fields
  - Changed field types
  - Required vs optional changes
  - Enum value changes
  - Validation rule changes

### 4. Classifies Changes

**Breaking Changes:**
- Endpoint removals
- Required field removals
- Type changes (incompatible)
- Required parameter additions
- Authentication requirement changes
- Enum value removals

**Non-Breaking Changes:**
- New endpoints
- Optional field additions
- Documentation updates
- Optional parameter additions
- New enum values
- Default value changes

### 5. Assesses Risk

**Risk Levels:**
- **CRITICAL**: Requires API versioning, major client impact
- **HIGH**: Breaking changes, careful migration needed
- **MEDIUM**: Non-breaking but requires code changes
- **LOW**: Safe changes, minimal impact

### 6. Generates Reports

Creates:
- **Diff Report**: Detailed comparison of all changes
- **Breaking Changes Summary**: Critical changes requiring attention
- **Implementation Plan**: Ordered tasks for implementing changes
- **Migration Guide**: Instructions for API consumers (if requested)
- **Risk Assessment**: Risk analysis and mitigation strategies

## Output Format

### Diff Report Structure

```markdown
# API Specification Comparison Report

**Baseline**: specs/v1.yaml
**Updated**: specs/v2.yaml
**Date**: 2026-01-21

## Summary

- Total Changes: 23
- Breaking Changes: 3
- Non-Breaking Changes: 20
- Risk Level: HIGH

## Risk Assessment

⚠️ CRITICAL: API versioning required
- 3 breaking changes detected
- Endpoints removed: 1
- Required fields changed: 2

## Endpoints

### Added Endpoints (5)

1. **GET /orders/{id}/items**
   - Purpose: Retrieve order items
   - Risk: LOW
   - Implementation: New controller action

2. **POST /orders/{id}/cancel**
   - Purpose: Cancel order
   - Risk: MEDIUM
   - Implementation: New service method + controller action

[...]

### Removed Endpoints (1)

1. **GET /orders/legacy** ⚠️ BREAKING
   - Impact: Existing clients will break
   - Mitigation: Provide deprecation period
   - Replacement: GET /orders with filters

### Modified Endpoints (3)

1. **POST /orders**
   - Changes:
     - Request body: Added optional field "priority"
     - Response: Added "estimatedDelivery" field
   - Breaking: NO
   - Risk: LOW

[...]

## Schemas

### Added Schemas (4)

1. **OrderItemResponse**
   - Fields: id, productId, quantity, price
   - Used by: GET /orders/{id}/items

[...]

### Modified Schemas (6)

1. **CreateOrderRequest** ⚠️ BREAKING
   - Added required field: "customerId" (was optional)
   - Impact: Existing clients must provide customerId
   - Mitigation: Add default value or make optional

[...]

## Breaking Changes Detail

### 1. Required Field Addition

**Schema**: CreateOrderRequest
**Change**: customerId (optional → required)
**Impact**: All POST /orders requests must include customerId
**Severity**: CRITICAL

**Mitigation Strategy**:
- Option 1: Create v2 API, keep v1 with optional customerId
- Option 2: Add default customer for backwards compatibility
- Option 3: Gradual migration with deprecation period

**Recommended**: Option 1 (API versioning)

[...]

## Implementation Plan

### Phase 1: Database Changes
- [ ] Add Priority column to Order table (nullable)
- [ ] Create OrderItem table
- [ ] Migrate existing data

### Phase 2: Entity Updates
- [ ] Update Order entity with Priority
- [ ] Create OrderItem entity
- [ ] Add navigation properties

### Phase 3: DTOs and Mapping
- [ ] Create OrderItemResponse DTO
- [ ] Update CreateOrderRequest with priority
- [ ] Add AutoMapper profiles

### Phase 4: Service Layer
- [ ] Implement GetOrderItems service method
- [ ] Implement CancelOrder service method
- [ ] Update CreateOrder with priority handling

### Phase 5: Controllers
- [ ] Add GetOrderItems endpoint
- [ ] Add CancelOrder endpoint
- [ ] Update CreateOrder endpoint

### Phase 6: Testing
- [ ] Unit tests for new service methods
- [ ] Integration tests for new endpoints
- [ ] Migration tests

### Phase 7: Documentation
- [ ] Update Swagger docs
- [ ] Create client migration guide
- [ ] Document breaking changes

## Versioning Recommendation

**Approach**: URL-based versioning (v1 → v2)

**Reasoning**:
- 3 breaking changes require new version
- Clear separation for clients
- Easier rollback and monitoring

**Implementation**:
- Keep v1 endpoints for 90 days
- Implement v2 with new contracts
- Add deprecation headers to v1
- Monitor v1 usage for sunset decision

## Migration Timeline

**Week 1-2**: Implement changes in v2
**Week 3**: Internal testing and validation
**Week 4**: Deploy to staging
**Week 5-6**: Client notification and migration support
**Week 7-16**: Deprecation period (v1 still active)
**Week 17**: Sunset v1 endpoints

## Next Steps

1. Review this diff report with team
2. Approve versioning strategy
3. Run: `/implement-spec-changes` to begin implementation
4. Execute implementation plan phase by phase
5. Communicate changes to API consumers
```

## Examples

### Example 1: Simple Field Addition

```bash
$ /compare-api-specs specs/current.yaml specs/updated.yaml

📊 API Spec Comparison Complete

Summary:
- 1 schema modified
- 0 breaking changes
- Risk Level: LOW

Changes:
  ✓ OrderResponse: Added optional field "trackingNumber"

Recommendation: Safe to implement without versioning

Next: Run /implement-spec-changes to apply changes
```

### Example 2: Breaking Changes Detected

```bash
$ /compare-api-specs specs/v1.yaml specs/v2.yaml

⚠️  API Spec Comparison Complete - BREAKING CHANGES DETECTED

Summary:
- 2 endpoints removed
- 3 schemas modified (breaking)
- 5 breaking changes total
- Risk Level: CRITICAL

Breaking Changes:
  ❌ Removed endpoint: GET /orders/legacy
  ❌ CreateOrderRequest: customerId (optional → required)
  ❌ OrderResponse: Removed field "oldStatus"

Recommendation: API versioning REQUIRED (v1 → v2)

Next: Review detailed report, then run /implement-spec-changes
```

### Example 3: With Implementation Plan

```bash
$ /compare-api-specs specs/v1.yaml specs/v2.yaml --plan

📊 Analyzing API specifications...
✓ Loaded baseline spec (v1.yaml)
✓ Loaded updated spec (v2.yaml)
✓ Compared 15 endpoints
✓ Compared 22 schemas
✓ Classified 18 changes
✓ Generated implementation plan

📋 Implementation Plan Created:
   - 23 tasks across 7 phases
   - Estimated effort: 3-4 weeks
   - Risk: HIGH (versioning required)

Outputs:
   - reports/api-diff-report.md
   - reports/implementation-plan.md
   - reports/breaking-changes.md
   - reports/migration-guide.md

Next: Review plans, then run /implement-spec-changes
```

## Integration with Workflow

This command is the FIRST step in the API evolution workflow:

```
1. /compare-api-specs        ← You are here
   ↓
2. Review diff report
   ↓
3. Approve implementation plan
   ↓
4. /implement-spec-changes
   ↓
5. Test and validate
   ↓
6. Deploy
```

## Behind the Scenes

When you run this command, Claude Code:

1. **Invokes api-diff-analyzer agent**
   - Loads both spec files
   - Performs deep comparison
   - Generates structured diff

2. **Invokes spec-change-planner skill** (if --plan flag)
   - Analyzes impact on codebase
   - Creates implementation task list
   - Orders tasks by dependencies

3. **Generates reports**
   - Markdown reports for humans
   - JSON manifests for automation
   - Migration guides for clients

4. **Provides recommendations**
   - Versioning strategy
   - Risk mitigation
   - Timeline estimates

## Best Practices

1. **Always compare before implementing**
   - Never implement spec changes without comparison
   - Understand full scope of changes

2. **Review breaking changes carefully**
   - Assess client impact
   - Plan deprecation timeline
   - Communicate early

3. **Use versioning for breaking changes**
   - Don't break existing clients
   - Maintain v1 during migration
   - Set clear sunset dates

4. **Generate implementation plan**
   - Use --plan flag for complex changes
   - Follow task ordering
   - Track progress

5. **Communicate with stakeholders**
   - Share diff report with team
   - Notify API consumers of breaking changes
   - Provide migration guides

## Related Commands

```bash
# After comparison, implement changes
/implement-spec-changes reports/implementation-plan.md

# Analyze specific endpoint changes
/analyze-endpoint POST /orders

# Generate specific components from spec
/derive-entities specs/v2.yaml
/implement-api specs/v2.yaml

# Review architecture patterns for changes
/recommend-pattern
```

## Troubleshooting

### Invalid Spec File

```
Error: Invalid OpenAPI specification in specs/v2.yaml
Line 45: Missing required field 'paths'

Solution: Validate your spec file with a linter
```

### Specs Not Comparable

```
Error: Cannot compare specs - different API domains
Baseline: Orders API
Updated: Payments API

Solution: Ensure both specs are versions of the same API
```

### No Changes Detected

```
Info: No changes detected between specifications
Both specs are identical

Solution: Verify you're comparing the correct files
```

## Notes

- Supports both OpenAPI 3.x and Swagger 2.0
- Can handle large specs (1000+ endpoints)
- Preserves comments and descriptions when comparing
- Can compare remote specs (URLs) with --url flag
- Output reports are git-friendly for version control
