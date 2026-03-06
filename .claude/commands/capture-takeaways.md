---
allowed-tools: Read, Write, Edit, Glob, Grep, AskUserQuestion, Task
argument-hint: [optional: specific focus area]
description: Capture learnings from the current conversation and update agentic setup files (commands, agents, skills, CLAUDE.md)
---

# Capture Takeaways Command

This command analyzes the current conversation to extract actionable learnings and automatically updates your agentic setup to incorporate them.

## Instructions

### Step 1: Gather User's Initial Ideas

Ask the user using AskUserQuestion with these questions:

**Question 1: Takeaway Categories**
"What types of takeaways did you identify from this conversation?"
Options:
- New command ideas (slash commands for repetitive workflows)
- New agent ideas (autonomous agents for complex tasks)
- New skill ideas (model-invoked capabilities)
- CLAUDE.md updates (project rules, patterns, conventions)
- Guide updates (detailed documentation for specific topics)
- Hook ideas (automated triggers on tool use)
- Not sure, probably a mix of some of the options

Allow multi-select.

**Question 2: Specific Takeaways**
"Please describe the specific takeaways you want to capture (one per line or comma-separated):"
This should be a free-text "Other" response where they list their ideas.

### Step 2: Analyze the Conversation

Review the full conversation history to:

1. **Identify Patterns**: Look for repetitive tasks the user performed or asked about
2. **Extract Workflows**: Find multi-step processes that could be automated
3. **Note Pain Points**: Identify areas where the user struggled or needed clarification
4. **Capture Decisions**: Document architectural or design decisions made
5. **Find Best Practices**: Extract any coding patterns or conventions discussed

Cross-reference with the user's provided ideas to validate and expand on them.

### Step 3: Summarize Findings

Present a structured summary to the user:

```markdown
## Conversation Analysis Summary

### Confirmed Takeaways (from your input)
- [List each user-provided takeaway with brief context from conversation]

### Additional Discoveries
- [Any patterns/workflows found that user didn't mention]

### Recommended Actions
| Takeaway | Recommended Type | Priority | Rationale |
|----------|------------------|----------|-----------|
| [takeaway] | command/agent/skill/guide | high/medium/low | [why] |
```

Ask user to confirm which items to proceed with.

### Step 4: Generate Agentic Setup Updates

For each confirmed takeaway, generate the appropriate file:

#### For New Commands (`/.claude/commands/[name].md`):
```yaml
---
allowed-tools: [appropriate tools]
argument-hint: [usage hint]
description: [clear description]
---

# [Command Name]

## Context
[When to use this command]

## Task
[Step-by-step instructions]

## Output
[Expected deliverables]
```

#### For New Agents (`/.claude/agents/[name].md`):
```yaml
---
name: [agent-name]
description: [trigger conditions and purpose]
model: [sonnet/opus/haiku]
allowed-tools: [tool list]
---

# [Agent Name]

## Purpose
[What this agent does autonomously]

## Key Principles
[Core guidelines the agent follows]

## Process
[Step-by-step workflow]

## Success Criteria
[How to know the task is complete]
```

#### For New Skills (`/.claude/skills/[name]/SKILL.md`):
```yaml
---
name: [skill-name]
description: [activation triggers and purpose]
allowed-tools: [tool list]
---

# [Skill Name]

## Instructions
[Detailed procedural steps]

## Examples
[Concrete usage examples]
```

#### For CLAUDE.md Updates:
- Read existing CLAUDE.md
- Identify appropriate section for new content
- Add new rules, patterns, or conventions
- Maintain existing structure and formatting

#### For Guide Updates (`/.claude/guides/[name].md`):
- Create new guide or update existing one
- Include practical examples
- Cross-reference with other guides

### Step 5: Validate Generated Content

For each generated file:

1. **Read any existing file** at the target path
2. **Check for conflicts** with existing content
3. **Verify consistency** with project patterns
4. **Ensure proper formatting** (YAML frontmatter, markdown structure)

If conflicts exist, ask user how to resolve:
- Merge with existing content
- Replace existing content
- Create as new file with different name
- Skip this item

### Step 6: Write Files and Report

1. Write all approved files using Write or Edit tools
2. Generate a summary report:

```markdown
## Takeaway Capture Complete

### Files Created
- `/.claude/commands/new-command.md` - [description]
- `/.claude/skills/new-skill/SKILL.md` - [description]

### Files Modified
- `/.claude/CLAUDE.md` - Added [section/rule]
- `/.claude/guides/existing-guide.md` - Updated [section]

### Next Steps
- Test new commands: `/[command-name]`
- New skills will auto-activate when [trigger conditions]
- Review CLAUDE.md changes for accuracy

### Items Deferred
- [Any items user chose to skip, with reason]
```

## File Location Reference

| Type | Location | Naming Convention |
|------|----------|-------------------|
| Commands | `/.claude/commands/` | `kebab-case.md` |
| Agents | `/.claude/agents/` | `kebab-case.md` |
| Skills | `/.claude/skills/[name]/` | Directory with `SKILL.md` |
| Guides | `/.claude/guides/` | `kebab-case.md` |
| Project Rules | `/.claude/CLAUDE.md` or `/CLAUDE.md` | N/A |

## Example Workflow

**User invokes**: `/capture-takeaways`

**Claude asks**: "What types of takeaways..." (multi-select)
**User selects**: New command ideas, CLAUDE.md updates

**Claude asks**: "Please describe specific takeaways..."
**User responds**: "1. Command to sync API schemas, 2. Rule about always validating OpenAPI examples"

**Claude analyzes conversation**, finds:
- User repeatedly ran schema validation manually
- Discussion about OpenAPI/TypeScript drift
- Decision to make OpenAPI source of truth

**Claude presents summary** with recommendations

**User confirms** items to create

**Claude generates**:
- `/.claude/commands/sync-schemas.md`
- Updates `CLAUDE.md` with new rule in API section

**Claude reports** completion with next steps

## Tips for Effective Takeaways

1. **Be specific**: "Command to generate DTOs from OpenAPI" > "DTO command"
2. **Include context**: Mention why this takeaway matters
3. **Think reusability**: Will this help in future similar situations?
4. **Consider scope**: Project-specific vs. globally useful
5. **Prioritize**: Focus on high-impact, frequently-needed items first
