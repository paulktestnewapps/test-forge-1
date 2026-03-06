# API Spec Evolution Workflow

Complete end-to-end workflow for managing API specification updates and implementing changes systematically.

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    API SPEC UPDATE                          │
│             (OpenAPI/Swagger updated)                       │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 1: Compare Specifications                              │
│ Command: /compare-api-specs baseline.yaml updated.yaml      │
│ Output: Diff report + Implementation plan                   │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 2: Review Changes                                      │
│ Action: Analyze diff report, assess risk                    │
│ Decision: Approve/modify plan, decide on versioning         │
└─────────────────────────────────────────────────────────────┘
                           ↓
                    [Breaking changes?]
                     /              \
                  YES                NO
                   ↓                  ↓
        ┌───────────────────┐  ┌──────────────────┐
        │ Plan API          │  │ Direct           │
        │ Versioning        │  │ Implementation   │
        │ (v1 + v2)         │  │                  │
        └───────────────────┘  └──────────────────┘
                   ↓                  ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 3: Implement Changes                                   │
│ Command: /implement-spec-changes plan.md                    │
│ Phases: DB → Entities → DTOs → Services → Controllers       │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 4: Validate                                            │
│ Actions: Build, test, format check                          │
│ Commands: dotnet build && dotnet test                       │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 5: Manual Testing                                      │
│ Actions: Test new endpoints, verify behavior                │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 6: Deploy                                              │
│ Actions: Create PR, deploy to staging, then production      │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 7: Monitor & Communicate                               │
│ Actions: Monitor metrics, notify consumers, track usage     │
└─────────────────────────────────────────────────────────────┘
```

## Detailed Workflow Steps

### Step 1: Compare Specifications

**Trigger**: API specification file updated

**Command**:
```bash
/compare-api-specs specs/baseline.yaml specs/updated.yaml --plan
```

**What Happens**:
1. **api-diff-analyzer agent** is invoked
2. Loads both specification files
3. Validates OpenAPI format
4. Performs deep comparison:
   - Endpoints (added/removed/modified)
   - Schemas/DTOs (field changes, type changes)
   - Parameters (query, path, header)
   - Request/response bodies
   - Authentication requirements
5. Classifies changes as breaking vs non-breaking
6. Assesses risk level (LOW/MEDIUM/HIGH/CRITICAL)
7. **spec-change-planner skill** is invoked
8. Generates implementation plan with task ordering

**Outputs**:
- `reports/api-diff-report.md` - Human-readable diff
- `reports/breaking-changes.md` - Critical changes summary
- `reports/implementation-plan.md` - Ordered task list
- `reports/migration-guide.md` - Client migration instructions
- `reports/changes.json` - Machine-readable manifest

**Duration**: 2-5 minutes

**Decision Point**: Review outputs before proceeding

---

### Step 2: Review and Decision

**Action**: Manual review of generated reports

**Key Questions**:
1. Are there breaking changes?
2. Is API versioning required?
3. Is the implementation plan correct?
4. Are all dependencies identified?
5. Is the timeline acceptable?
6. Do stakeholders need to be notified?

**Decision Matrix**:

```
IF breaking_changes = 0:
  → Proceed to Step 3 (Direct Implementation)

ELSE IF breaking_changes = 1 AND can_mitigate:
  → Consider gradual migration
  → May or may not need versioning
  → Proceed to Step 3 with caution

ELSE IF breaking_changes > 1 OR critical_risk:
  → API versioning REQUIRED
  → Plan v1 + v2 approach
  → Set deprecation timeline
  → Notify API consumers
  → Proceed to Step 3 with versioning
```

**Versioning Decision**:

```yaml
URL Versioning (Recommended for breaking changes):
  pattern: /v1/resource vs /v2/resource
  pros: Clear, explicit, easy to route
  cons: URL changes, potential code duplication

Header Versioning:
  pattern: Header "API-Version: 2"
  pros: Clean URLs, flexible
  cons: Hidden from URL, complex routing

No Versioning (Only if backwards compatible):
  pattern: Single evolving API
  pros: Simple for clients
  cons: Can't make breaking changes
```

**Approval Checklist**:
- [ ] All changes understood
- [ ] Breaking changes identified
- [ ] Versioning strategy decided
- [ ] Timeline is reasonable
- [ ] Team informed
- [ ] API consumers notified (if breaking)

**Duration**: 15-30 minutes

---

### Step 3: Implement Changes

**Command**:
```bash
# Standard execution
/implement-spec-changes reports/implementation-plan.md

# Or interactive for first-time/complex changes
/implement-spec-changes reports/implementation-plan.md --interactive

# Or phase-by-phase for critical changes
/implement-spec-changes reports/implementation-plan.md --phase=database
```

**What Happens**:

**Phase 1: Database Schema Changes**
```
Tasks:
  - Create new tables
  - Add/modify columns
  - Create migrations
  - Update indexes

Validation:
  - Migrations compile
  - Can apply/rollback

Tools Used:
  - /generate-sql
  - dotnet ef migrations add
```

**Phase 2: Entity Model Updates**
```
Tasks:
  - Create new entities
  - Update existing entities
  - Add navigation properties
  - Update DbContext

Validation:
  - Entities compile
  - DbContext is valid

Tools Used:
  - /generate-db-schema
  - File edits
```

**Phase 3: DTOs and Mapping**
```
Tasks:
  - Generate request/response DTOs
  - Update existing DTOs
  - Create AutoMapper profiles
  - Validate mappings

Validation:
  - DTOs compile
  - Mappings are valid
  - No circular references

Tools Used:
  - /generate-dtos
  - /generate-mapper
```

**Phase 4: Service Layer**
```
Tasks:
  - Generate repository methods
  - Implement service methods
  - Add business logic
  - Update validation

Validation:
  - Services compile
  - DI resolves correctly

Tools Used:
  - /generate-repository
  - File edits
```

**Phase 5: Controllers**
```
Tasks:
  - Generate new endpoints
  - Update existing endpoints
  - Configure routing
  - Add attributes

Validation:
  - Controllers compile
  - Routes don't conflict
  - DI configured

Tools Used:
  - /generate-controller
  - /implement-endpoint
```

**Phase 6: Testing**
```
Tasks:
  - Generate unit tests
  - Update existing tests
  - Add test data factories
  - Run test suite

Validation:
  - Tests compile
  - All tests pass
  - Coverage meets threshold

Tools Used:
  - /generate-tests
  - dotnet test
```

**Phase 7: Documentation**
```
Tasks:
  - Update Swagger annotations
  - Generate API docs
  - Create migration guides
  - Update README

Validation:
  - Documentation accurate
  - Examples work

Tools Used:
  - File edits
```

**Progress Tracking**:
```
✓ Phase 1/7: Database Schema (3/3 tasks)
✓ Phase 2/7: Entity Models (5/5 tasks)
⏳ Phase 3/7: DTOs and Mapping (2/4 tasks)
  Pending: Controllers, Testing, Documentation

Overall: 10/23 tasks complete (43%)
```

**Duration**: 30 minutes - 4 hours (depends on complexity)

---

### Step 4: Automated Validation

**Commands**:
```bash
# 1. Build check
dotnet build

# 2. Format check
dotnet format --verify-no-changes

# 3. Test execution
dotnet test
```

**Success Criteria**:
```
✓ Build exits with code 0
✓ Format check exits with code 0
✓ All tests pass (no failures/skips)
```

**If Validation Fails**:
```
Compilation Error:
  → Review error messages
  → Fix code issues
  → Re-run dotnet build

Format Issues:
  → Run dotnet format (without --verify)
  → Commit formatting changes

Test Failures:
  → Review failing tests
  → Update test data/mocks
  → Fix implementation if needed
  → NEVER skip failing tests
  → Re-run dotnet test
```

**Duration**: 2-5 minutes

---

### Step 5: Manual Testing

**Actions**:
1. **Start application**
   ```bash
   dotnet run --project YourApi/YourApi.csproj
   ```

2. **Test new endpoints**
   - Use Swagger UI (http://localhost:5000/swagger)
   - Test with Postman/curl
   - Verify request/response format

3. **Test modified endpoints**
   - Verify backwards compatibility (if applicable)
   - Test with old and new payloads
   - Check error handling

4. **Test versioning** (if applicable)
   - Test v1 endpoints still work
   - Test v2 endpoints with new contract
   - Verify routing works correctly

5. **Database verification** (if schema changed)
   - Check migrations applied
   - Verify data integrity
   - Test rollback migration

**Manual Test Checklist**:
- [ ] All new endpoints return expected data
- [ ] Existing endpoints not broken
- [ ] Validation rules work correctly
- [ ] Error responses are correct
- [ ] v1 endpoints work (if versioned)
- [ ] v2 endpoints work with new contract
- [ ] Database state is correct

**Duration**: 15-30 minutes

---

### Step 6: Deploy

**Phase A: Create Pull Request**

```bash
# Review changes
git diff main

# Create PR with detailed description
/create-pr
```

**PR Description Template**:
```markdown
## API Spec Update: [Brief Description]

### Summary
- [Number] endpoints added
- [Number] endpoints modified
- [Number] schemas updated
- Breaking changes: [YES/NO]

### Changes
- Added: [list new endpoints]
- Modified: [list changed endpoints]
- Removed: [list removed endpoints (if any)]

### Breaking Changes
[If applicable, list breaking changes and mitigation]

### Versioning
- New version: v2
- Deprecation timeline: v1 sunset on [date]

### Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing complete
- [ ] Backwards compatibility verified (if applicable)

### Migration Guide
[Link to migration guide for API consumers]

### Related
- Spec comparison report: [link]
- Implementation plan: [link]
```

**Phase B: Deploy to Staging**

```bash
# Triggered by PR merge or manual deployment
# Deployment pipeline runs:
1. Build
2. Run tests
3. Deploy to staging environment
4. Smoke tests
```

**Staging Validation**:
- [ ] Application starts successfully
- [ ] Health checks pass
- [ ] New endpoints accessible
- [ ] Existing endpoints still work
- [ ] Database migrations applied
- [ ] No errors in logs

**Phase C: Deploy to Production**

```yaml
Non-Breaking Changes:
  - Direct deployment after staging validation
  - Monitor for issues
  - Rollback plan ready

Breaking Changes (with versioning):
  - Deploy v2 alongside v1
  - Route configuration updated
  - Both versions active
  - Monitor v1 usage
  - Gradual sunset of v1
```

**Duration**: 1-4 hours (including staging validation)

---

### Step 7: Monitor & Communicate

**Monitoring** (First 24 hours):
```
METRICS TO WATCH:
- API error rate
- Response times
- v1 vs v2 usage (if versioned)
- Database performance
- Failed requests

TOOLS:
- Application Insights
- Log Analytics
- Custom dashboards

ALERTS:
- Error rate > baseline
- Response time > SLA
- Failed migrations
```

**Communication**:

**Internal Team**:
```markdown
✅ API v2 Deployed

Changes:
- [Summary of changes]

Versions:
- v1: Active, deprecated [date]
- v2: Active, current

Monitoring:
- Dashboard: [link]
- Logs: [link]

Next Steps:
- Monitor metrics for 24h
- Track v1 usage
- Sunset v1 on [date]
```

**API Consumers** (if breaking changes):
```markdown
Subject: API Update - Breaking Changes in v2

We've deployed v2 of the [API Name] API with important updates.

Breaking Changes:
- [List of breaking changes]

What You Need To Do:
1. Review migration guide: [link]
2. Update your integration to use v2 endpoints
3. Test thoroughly in sandbox environment
4. Deploy before [v1 sunset date]

Timeline:
- Today: v2 available
- [Date + 90 days]: v1 deprecated (sunset)

Support:
- Migration guide: [link]
- API docs: [link]
- Support: [contact]
```

**Duration**: Ongoing for 90+ days (if deprecation timeline)

---

## Example Scenarios

### Scenario 1: Simple Non-Breaking Addition

```
CHANGE: Add optional "priority" field to Order

Day 1:
10:00 - /compare-api-specs old.yaml new.yaml
        → Risk: LOW, no breaking changes
10:05 - Review diff report
        → Approve for implementation
10:10 - /implement-spec-changes plan.md
        → All phases execute automatically
10:40 - dotnet build && dotnet test
        → All pass
10:45 - Manual testing
        → Works as expected
11:00 - Create PR
11:30 - PR approved and merged
12:00 - Deploy to staging
14:00 - Deploy to production
14:30 - Monitor metrics
        → All green

Total Time: 4.5 hours
```

### Scenario 2: Breaking Change with Versioning

```
CHANGE: Remove "legacyId" field (breaking)

Week 1:
Mon 10:00 - /compare-api-specs v1.yaml v2.yaml
            → Risk: CRITICAL, versioning required
Mon 10:30 - Team review of diff report
            → Decide on v1/v2 approach
            → Set 90-day deprecation timeline
Mon 11:00 - /implement-spec-changes plan.md --interactive
            → Review each phase
            → Approve v2 controller creation
Mon 16:00 - Implementation complete
            → Both v1 and v2 exist
Mon 16:30 - Validation
            → All tests pass
            → Both versions work
Tue 09:00 - Create PR with migration guide
Wed 10:00 - PR approved
Wed 14:00 - Deploy to staging
Thu 09:00 - Staging validation complete
Thu 14:00 - Deploy to production (v1 + v2)
Thu 15:00 - Email API consumers
            → Migration guide sent
            → 90-day timeline communicated

Week 2-13: Monitor v1 usage, support migrations

Week 14:
Mon - Final reminder to remaining v1 users
Wed - Sunset v1 (remove endpoints)
Thu - Deploy v1 removal
```

### Scenario 3: Complex Multi-Endpoint Update

```
CHANGE: Add 5 new endpoints, modify 3 existing

Day 1 (Planning):
09:00 - /compare-api-specs baseline.yaml updated.yaml --plan
        → 23 changes detected
        → 2 breaking, 21 non-breaking
09:30 - Team review
        → Decide on versioning for breaking changes
        → Approve implementation plan
10:00 - Phase-by-phase implementation begins

Day 1-2 (Implementation):
10:00 - /implement-spec-changes plan.md --phase=database
        → Database changes complete
14:00 - /implement-spec-changes plan.md --phase=entities
        → Entity updates complete
Day 2:
09:00 - /implement-spec-changes plan.md --phase=dtos
        → DTO generation complete
11:00 - /implement-spec-changes plan.md --phase=services
        → Service layer complete
14:00 - /implement-spec-changes plan.md --phase=controllers
        → Controllers complete
16:00 - /implement-spec-changes plan.md --phase=tests
        → All tests passing

Day 3 (Validation & Deployment):
09:00 - Final validation
        → Build, test, format all pass
10:00 - Manual testing
        → All endpoints verified
12:00 - Create PR
14:00 - PR review and approval
16:00 - Deploy to staging
Day 4:
09:00 - Staging validation
14:00 - Production deployment
15:00 - Monitoring and communication

Total Time: 3.5 days
```

## Quick Reference

### Commands

```bash
# Compare specs
/compare-api-specs baseline.yaml updated.yaml --plan

# Implement (standard)
/implement-spec-changes plan.md

# Implement (interactive)
/implement-spec-changes plan.md --interactive

# Implement (dry run)
/implement-spec-changes plan.md --dry-run

# Implement (phase-by-phase)
/implement-spec-changes plan.md --phase=database

# Validate
dotnet build && dotnet test && dotnet format --verify-no-changes

# Deploy
/create-pr
```

### Decision Flows

```
Breaking Changes? → YES → API Versioning → v1 + v2
                  → NO → Direct Implementation

Risk Level? → CRITICAL → Phase-by-phase + manual review
            → HIGH → Interactive mode
            → MEDIUM → Standard execution
            → LOW → Automated execution

Timeline? → Immediate → Direct deployment
          → Gradual → Feature flag or phased rollout
          → Deprecation → v1 + v2 with sunset timeline
```

## Related Documentation

- [API Spec Evolution Guide](../guides/api-spec-evolution-guide.md)
- [Compare API Specs Command](../commands/compare-api-specs.md)
- [Implement Spec Changes Command](../commands/implement-spec-changes.md)
- [API Diff Analyzer Agent](../agents/api-diff-analyzer.md)
- [Spec Change Planner Skill](../skills/spec-change-planner/SKILL.md)
