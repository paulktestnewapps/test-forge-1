# Claude Code Workflows

End-to-end workflows for common development scenarios.

## Purpose

Workflows provide complete, step-by-step processes that orchestrate multiple agents, skills, and commands to accomplish complex tasks. Each workflow includes detailed guidance, decision points, and validation steps.

## Available Workflows

### [api-spec-evolution.md](api-spec-evolution.md)

**Purpose**: Manage API specification updates from comparison through implementation and deployment.

**Trigger**: OpenAPI/Swagger specification file is updated

**Steps**:
1. Compare specifications (identify changes)
2. Review and decision (assess risk, plan versioning)
3. Implement changes (generate code)
4. Validate (build, test, format)
5. Manual testing
6. Deploy (staging, production)
7. Monitor and communicate

**Key Features**:
- Automated change detection
- Breaking change classification
- Risk assessment
- Versioning strategy recommendations
- Phase-by-phase implementation
- Rollback procedures

**Duration**: 2 hours - 3.5 days (depending on complexity)

**Use When**:
- API spec has been updated
- New endpoints need to be added
- Existing endpoints are modified
- Breaking changes are introduced
- API versioning is required

---

## Workflow vs Commands vs Skills vs Agents

### Workflows
- **Complete end-to-end processes**
- Orchestrate multiple tools and steps
- Include decision points and validation
- Provide detailed guidance at each step
- Example: API Spec Evolution (7 steps, multiple days)

### Commands
- **Single discrete actions**
- Execute specific tasks
- Can be part of a workflow
- Example: `/compare-api-specs` or `/implement-spec-changes`

### Skills
- **Specialized capabilities**
- Provide domain knowledge
- Used by agents and commands
- Example: spec-change-planner (creates implementation plans)

### Agents
- **Autonomous task executors**
- Use skills and tools
- Can work independently or as part of workflow
- Example: api-diff-analyzer (compares specs)

## How Workflows Are Used

### Manual Execution
User follows workflow documentation step-by-step, executing commands at appropriate times.

```bash
# Step 1: Compare specs
/compare-api-specs baseline.yaml updated.yaml --plan

# User reviews output, makes decisions

# Step 3: Implement changes
/implement-spec-changes implementation-plan.md

# User validates, tests, deploys
```

### Automated Execution (Future)
Workflows can be triggered automatically based on events.

```yaml
trigger: API spec file changed
workflow: api-spec-evolution
automation_level: semi-automated  # User approves at decision points
```

## Workflow Structure

All workflows follow a consistent structure:

```markdown
# Workflow Name

## Workflow Overview
- Visual flowchart
- High-level steps
- Decision points

## Detailed Workflow Steps
For each step:
  - Trigger/condition
  - Command to execute
  - What happens behind the scenes
  - Outputs generated
  - Duration estimate
  - Decision point (if applicable)

## Example Scenarios
- Simple case
- Complex case
- Edge cases

## Quick Reference
- Commands summary
- Decision flows
- Troubleshooting

## Related Documentation
- Links to related guides, commands, agents, skills
```

## Creating New Workflows

### When to Create a Workflow

Create a workflow when:
1. Task involves multiple steps over time
2. Requires coordination of multiple agents/commands
3. Has decision points requiring user input
4. Needs detailed guidance at each step
5. Will be repeated frequently
6. Needs to be standardized across team

### Workflow Template

```markdown
# [Workflow Name]

## Workflow Overview
[Flowchart showing complete process]

## Trigger
[What initiates this workflow]

## Prerequisites
[What must be in place before starting]

## Steps

### Step 1: [Name]
**Trigger**: [What starts this step]
**Command**: `[command]`
**What Happens**: [Detailed explanation]
**Outputs**: [What is generated]
**Duration**: [Estimated time]
**Decision Point**: [If applicable]

[Repeat for each step]

## Example Scenarios
[Real-world examples with timelines]

## Quick Reference
[Commands, decisions, troubleshooting]

## Related Documentation
[Links]
```

## Common Workflow Patterns

### Linear Workflow
```
Step 1 → Step 2 → Step 3 → Step 4 → Complete
```
Example: Simple feature implementation

### Decision-Based Workflow
```
Step 1 → Decision Point → Branch A or Branch B → Converge → Complete
```
Example: API spec evolution (breaking vs non-breaking)

### Iterative Workflow
```
Step 1 → Step 2 → Validation → (if fail, return to Step 1) → Complete
```
Example: Code generation with validation loops

### Phase-Based Workflow
```
Phase 1 (multiple steps) → Validate → Phase 2 → Validate → ... → Complete
```
Example: Multi-phase implementation

## Best Practices

### Documentation
1. **Always include visual flowchart** at the top
2. **Estimate durations** for each step
3. **Mark decision points clearly** with [Decision Point] tags
4. **Provide examples** for simple and complex cases
5. **Link to related documentation**

### Design
1. **Keep workflows focused** on a single goal
2. **Make decision criteria clear** and actionable
3. **Include validation steps** after each phase
4. **Provide rollback/recovery** procedures
5. **Estimate realistic timelines**

### Maintenance
1. **Update when commands/agents change**
2. **Add new scenarios** as they arise
3. **Keep examples current** with latest patterns
4. **Test workflows** periodically
5. **Gather feedback** from users

## Future Workflows (Planned)

### Feature Implementation Workflow
```
Trigger: New feature request
Steps: Requirements → Design → Implementation → Testing → Deployment
```

### Bug Fix Workflow
```
Trigger: Bug reported
Steps: Reproduce → Root cause → Fix → Test → Deploy
```

### Legacy Modernization Workflow
```
Trigger: Legacy code identified
Steps: Analyze → Plan → Migrate → Validate → Deploy
```

### Database Migration Workflow
```
Trigger: Schema changes needed
Steps: Design → Generate → Test → Apply → Validate
```

## Related Documentation

- [Guides](../guides/) - Quick reference for specific topics
- [Commands](../commands/) - Individual command documentation
- [Skills](../skills/) - Specialized capabilities
- [Agents](../agents/) - Autonomous task executors
- [CLAUDE.md](../../CLAUDE.md) - Project overview and conventions
