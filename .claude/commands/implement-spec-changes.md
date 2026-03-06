# Implement Spec Changes Command

## Command

```bash
/implement-spec-changes [implementation-plan] [options]
```

## Purpose

Execute the implementation plan generated from API spec comparison. Orchestrates all code generation, database migrations, and testing required to implement spec changes.

## Usage

### Basic Usage

```bash
# Implement from plan file
/implement-spec-changes reports/implementation-plan.md

# Implement from diff report (auto-generates plan)
/implement-spec-changes reports/api-diff-report.md

# Implement specific phase only
/implement-spec-changes reports/implementation-plan.md --phase=database

# Dry run (show what would be done)
/implement-spec-changes reports/implementation-plan.md --dry-run
```

### Advanced Options

```bash
# Interactive mode (confirm each task)
/implement-spec-changes reports/implementation-plan.md --interactive

# Skip testing phase
/implement-spec-changes reports/implementation-plan.md --skip-tests

# Auto-commit after each phase
/implement-spec-changes reports/implementation-plan.md --auto-commit

# Resume from specific task
/implement-spec-changes reports/implementation-plan.md --resume-from=TASK-015
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `implementation-plan` | path | Yes | Path to implementation plan or diff report |
| `--phase` | string | No | Execute specific phase only |
| `--dry-run` | flag | No | Show planned actions without executing |
| `--interactive` | flag | No | Confirm each task before execution |
| `--skip-tests` | flag | No | Skip test generation and execution |
| `--auto-commit` | flag | No | Create git commits after each phase |
| `--resume-from` | string | No | Resume from specific task ID |
| `--phases` | string | No | Execute specific phases (comma-separated) |

## What It Does

### Execution Flow

1. **Parse Implementation Plan**
   - Load plan file
   - Validate task dependencies
   - Build execution graph
   - Identify phases

2. **Execute Phases in Order**
   - Phase 1: Database schema changes
   - Phase 2: Entity model updates
   - Phase 3: DTO and mapping updates
   - Phase 4: Service layer changes
   - Phase 5: Controller updates
   - Phase 6: Test generation
   - Phase 7: Documentation updates

3. **Task Execution**
   - Check dependencies satisfied
   - Execute task (code generation, file edits)
   - Validate output
   - Run verification commands
   - Mark task complete

4. **Validation**
   - Compile checks
   - Test execution
   - Format validation
   - Breaking change verification

5. **Reporting**
   - Progress tracking
   - Error reporting
   - Summary generation

## Task Types and Actions

### Database Tasks

```yaml
task_type: database
actions:
  - Generate SQL migration script
  - Apply migration (if --apply flag)
  - Verify migration success
  - Update entity models

commands_used:
  - /generate-sql --add-column
  - /generate-sql --create-table
  - dotnet ef migrations add
```

### Entity Tasks

```yaml
task_type: entity
actions:
  - Create new entity classes
  - Update existing entities
  - Add navigation properties
  - Update DbContext

commands_used:
  - /generate-db-schema
  - File edits for updates
```

### DTO Tasks

```yaml
task_type: dto
actions:
  - Generate request/response DTOs
  - Update existing DTOs
  - Create AutoMapper profiles
  - Validate mappings

commands_used:
  - /generate-dtos
  - /generate-mapper
```

### Service Tasks

```yaml
task_type: service
actions:
  - Generate repository methods
  - Generate service methods
  - Update business logic
  - Add validation

commands_used:
  - /generate-repository
  - File edits for service layer
```

### Controller Tasks

```yaml
task_type: controller
actions:
  - Generate new endpoints
  - Update existing endpoints
  - Add routing
  - Configure DI

commands_used:
  - /generate-controller
  - /implement-endpoint
```

### Test Tasks

```yaml
task_type: test
actions:
  - Generate unit tests
  - Update existing tests
  - Validate test coverage
  - Run tests

commands_used:
  - /generate-tests
  - dotnet test
```

### Documentation Tasks

```yaml
task_type: documentation
actions:
  - Update Swagger annotations
  - Generate API docs
  - Create migration guides
  - Update README

commands_used:
  - File edits
  - Documentation generation
```

## Execution Phases

### Phase 1: Database Schema

```
Tasks:
  - Create new tables
  - Add columns
  - Modify column types
  - Add indexes
  - Create foreign keys

Validation:
  - Migration compiles
  - Migration can be applied
  - Rollback script works

Output:
  - Migration files
  - SQL scripts
```

### Phase 2: Entity Models

```
Tasks:
  - Create new entities
  - Update existing entities
  - Add navigation properties
  - Update DbContext

Validation:
  - Entities compile
  - DbContext is valid
  - Mappings are correct

Output:
  - Entity classes
  - DbContext updates
```

### Phase 3: DTOs and Mapping

```
Tasks:
  - Generate request DTOs
  - Generate response DTOs
  - Create AutoMapper profiles
  - Update existing DTOs

Validation:
  - DTOs compile
  - Mappings are valid
  - No circular references

Output:
  - DTO classes
  - AutoMapper profiles
```

### Phase 4: Service Layer

```
Tasks:
  - Generate repository methods
  - Implement service methods
  - Add business logic
  - Update validation

Validation:
  - Services compile
  - Dependencies resolve
  - Logic is correct

Output:
  - Repository interfaces/implementations
  - Service classes
```

### Phase 5: Controllers

```
Tasks:
  - Generate new endpoints
  - Update existing endpoints
  - Configure routing
  - Add attributes

Validation:
  - Controllers compile
  - Routes don't conflict
  - DI is configured

Output:
  - Controller classes
  - Updated Startup/Program.cs
```

### Phase 6: Testing

```
Tasks:
  - Generate unit tests
  - Update integration tests
  - Add test data factories
  - Run tests

Validation:
  - Tests compile
  - All tests pass
  - Coverage meets threshold

Output:
  - Test classes
  - Test data factories
```

### Phase 7: Documentation

```
Tasks:
  - Update Swagger docs
  - Create migration guide
  - Update API documentation
  - Update README

Validation:
  - Documentation is accurate
  - Examples work

Output:
  - Updated docs
  - Migration guides
```

## Output and Progress

### Console Output

```
🚀 Implementing API Spec Changes

Phase 1/7: Database Schema Changes
  ✓ TASK-001: Add Priority column to Order table
  ✓ TASK-002: Create OrderItem table
  ✓ TASK-003: Add foreign key constraints
  Phase 1 complete (3/3 tasks) ✓

Phase 2/7: Entity Model Updates
  ✓ TASK-004: Create OrderItem entity
  ✓ TASK-005: Update Order entity with Priority
  ⏳ TASK-006: Add navigation properties
  Phase 2 in progress (2/3 tasks)...

Overall Progress: 5/23 tasks complete (22%)
```

### Generated Reports

1. **execution-report.md**: Detailed execution log
2. **changes-summary.md**: Summary of all changes made
3. **validation-results.md**: Compilation and test results
4. **next-steps.md**: Manual steps required (if any)

## Interactive Mode

When `--interactive` flag is used:

```
Task: TASK-005: Update Order entity with Priority
  Type: entity
  Files: Models/Order.cs
  Action: Add Priority property

  Show diff? [y/n]: y

  --- Models/Order.cs
  +++ Models/Order.cs
  @@ -10,6 +10,7 @@
       public string Status { get; set; }
  +    public string Priority { get; set; }
       public DateTime CreatedOn { get; set; }

  Execute this task? [y/n/s/q]:
    y - Yes, execute
    n - No, skip
    s - Skip remaining in this phase
    q - Quit implementation

  > y

  ✓ Task executed successfully
```

## Error Handling

### Compilation Errors

```
❌ Phase 2 Failed: Compilation Error

Task: TASK-006: Add navigation properties
File: Models/Order.cs
Error: CS0246: The type 'OrderItem' could not be found

Recommendation:
  - Check that OrderItem entity was created in previous task
  - Verify using statements
  - Run: dotnet build for full error details

Options:
  [r] Retry task
  [s] Skip task and continue
  [q] Quit implementation
  [d] Show detailed error
```

### Dependency Errors

```
⚠️  Dependency Not Satisfied

Task: TASK-015: Generate OrderController
Depends on: TASK-012 (Create OrderService)
Status: TASK-012 failed

Cannot proceed. Please fix TASK-012 first.
```

### Test Failures

```
❌ Phase 6 Failed: Test Failures

Passed: 45
Failed: 3
Skipped: 0

Failed Tests:
  - OrderServiceTests.CreateOrder_WithPriority_SetsCorrectly
  - OrderServiceTests.GetOrderItems_ReturnsItems
  - OrderControllerTests.CancelOrder_Returns200

Options:
  [f] Fix tests before continuing (recommended)
  [s] Skip test failures and continue
  [v] View test details
  [q] Quit implementation
```

## Rollback

If implementation fails or needs to be reverted:

```bash
# Rollback all changes
/rollback-spec-changes reports/execution-report.md

# Rollback specific phase
/rollback-spec-changes reports/execution-report.md --phase=database

# Rollback to specific task
/rollback-spec-changes reports/execution-report.md --to-task=TASK-010
```

## Phase-by-Phase Execution

```bash
# Execute only database phase
/implement-spec-changes plan.md --phase=database

# Review changes, then continue with entities
/implement-spec-changes plan.md --phase=entities

# Execute multiple specific phases
/implement-spec-changes plan.md --phases=database,entities,dtos
```

## Git Integration

With `--auto-commit` flag:

```
✓ Phase 1 Complete: Database Schema Changes

Creating git commit...
  Added:
    - Migrations/20260121_AddPriorityToOrder.cs
    - Migrations/20260121_CreateOrderItemTable.cs

  Commit message:
    "feat: Add Priority and OrderItem schema changes

    - Add Priority column to Order table
    - Create OrderItem table with foreign key
    - Generated from API spec comparison

    Phase: 1/7 (Database Schema)
    Tasks: TASK-001, TASK-002, TASK-003"

  Commit created: abc123f

Continue to Phase 2? [y/n]:
```

## Examples

### Example 1: Simple Implementation

```bash
$ /implement-spec-changes reports/implementation-plan.md

🚀 Implementing API Spec Changes

Phase 1/3: Entity Updates
  ✓ Add TrackingNumber to OrderResponse
  ✓ Update AutoMapper profile

Phase 2/3: Service Layer
  ✓ Update GetOrder service method

Phase 3/3: Testing
  ✓ Update OrderServiceTests
  ✓ Run all tests
  All tests passed ✓

✅ Implementation Complete!
  Tasks executed: 5/5
  Duration: 2 minutes
  Tests: All passing

Next Steps:
  - Review changes: git diff
  - Test manually
  - Create PR: /create-pr
```

### Example 2: Complex Implementation with Breaking Changes

```bash
$ /implement-spec-changes reports/implementation-plan.md --interactive

🚀 Implementing API Spec Changes (Interactive Mode)

⚠️  Breaking Changes Detected
This implementation includes API versioning (v1 → v2)

Phase 1/7: Database Schema Changes
  Execute all 3 database tasks? [y/n]: y
  ✓ All database tasks complete

Phase 2/7: Entity Model Updates
  ✓ Create OrderItem entity

  Task: Create v2 DTOs
  This will create new v2 request/response DTOs alongside v1
  Continue? [y/n]: y
  ✓ Complete

[... continues through all phases ...]

✅ Implementation Complete!
  API v2 created successfully
  v1 endpoints remain active

Next Steps:
  - Test v2 endpoints manually
  - Update deployment config for versioning
  - Notify API consumers
  - Set v1 deprecation date
```

### Example 3: Dry Run

```bash
$ /implement-spec-changes reports/implementation-plan.md --dry-run

🔍 DRY RUN - No changes will be made

Phase 1: Database Schema Changes
  Would execute:
    - Generate migration: Add Priority to Order
    - Generate migration: Create OrderItem table
    - Update DbContext

Phase 2: Entity Model Updates
  Would execute:
    - Create file: Models/OrderItem.cs
    - Edit file: Models/Order.cs (add Priority property)
    - Edit file: Data/AppDbContext.cs (add DbSet)

[... shows all planned actions ...]

Summary:
  Total tasks: 23
  Files to create: 8
  Files to modify: 15
  Commands to run: 12
  Estimated duration: 30-45 minutes

To execute: Remove --dry-run flag
```

## Best Practices

1. **Review Plan First**
   - Always review implementation plan before executing
   - Understand scope and impact
   - Check for unexpected tasks

2. **Use Dry Run**
   - Run with --dry-run first for complex changes
   - Verify all tasks make sense
   - Check file paths are correct

3. **Phase-by-Phase for Critical Changes**
   - Execute one phase at a time
   - Validate each phase before continuing
   - Easier to debug issues

4. **Commit Frequently**
   - Use --auto-commit for safety
   - Each phase gets its own commit
   - Easy to rollback if needed

5. **Test After Each Phase**
   - Don't skip testing
   - Fix test failures immediately
   - Don't continue with failing tests

6. **Interactive Mode for First Time**
   - Use --interactive for new workflows
   - Review each change
   - Build confidence in automation

## Integration with Workflow

This command is the EXECUTION step in the API evolution workflow:

```
1. /compare-api-specs
   ↓
2. Review diff report
   ↓
3. Approve implementation plan
   ↓
4. /implement-spec-changes    ← You are here
   ↓
5. dotnet test
   ↓
6. Manual testing
   ↓
7. Create PR and deploy
```

## Related Commands

```bash
# Before implementation
/compare-api-specs specs/v1.yaml specs/v2.yaml --plan

# During implementation
/implement-spec-changes --dry-run
/implement-spec-changes --interactive
/implement-spec-changes --phase=database

# After implementation
dotnet build
dotnet test
dotnet format
/create-pr

# If issues occur
/rollback-spec-changes
```

## Notes

- Always run tests after implementation
- Breaking changes require manual review
- Some tasks may require manual intervention
- Generated code follows project conventions
- Validates against CLAUDE.md standards
- Creates git-friendly commits
- Preserves existing code patterns
