---
name: commit
description: Generate a git commit message following project commit best practices (prefix tags, concise subject, single logical change). Use when the user wants to commit staged changes or needs a commit message suggestion.
argument-hint: [optional: description hint or scope]
allowed-tools: Bash, Read, Glob, Grep, AskUserQuestion
---

# Commit Skill

Generates a well-formed commit message following project conventions, then commits.

## Best Practices Reference

See [.claude/guides/commit-best-practices.md](.claude/guides/commit-best-practices.md) for full rules.

**Summary:**
- Use a prefix tag: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`, `style:`, `perf:`, `ci:`, `revert:`
- Subject line under 72 characters, imperative mood, no period
- One logical change per commit — do not group unrelated changes
- Be specific about WHAT and WHY, not HOW

## Instructions

### Step 1: Check for Staged Changes

```bash
git diff --staged --stat
git status
git log --oneline -5
```

If nothing is staged, inform the user and stop. Do not auto-stage files.

### Step 2: Analyze the Changes

- Identify the **type** of change (new feature, bug fix, refactor, etc.)
- Determine the **scope** — what area/layer is affected (e.g., controller, service, entity)
- Check if staged changes represent **one logical unit of work**

### Step 3: Warn on Mixed Commits

If the diff spans unrelated files or multiple logical concerns:
- Flag this: "The staged changes appear to include unrelated concerns: [list them]. Consider splitting into separate commits."
- Use AskUserQuestion to ask: "How would you like to proceed?" with options: "Commit as-is", "I'll unstage some files first"

### Step 4: Generate the Commit Message

If the user provided an argument hint (`$ARGUMENTS`), use it to guide the message.

Produce a commit message following this format:

```
<prefix>: <concise subject in imperative mood>

<optional body: 1-3 lines explaining WHY if not obvious>
```

**Prefix selection guide:**

| Prefix | When to use |
|--------|-------------|
| `feat:` | New feature or endpoint |
| `fix:` | Bug fix |
| `refactor:` | Restructure without behavior change |
| `test:` | Adding or updating tests |
| `docs:` | Documentation only |
| `chore:` | Config, tooling, build scripts |
| `style:` | Formatting only |
| `perf:` | Performance improvement |
| `ci:` | CI/CD changes |
| `revert:` | Reverting a previous commit |

### Step 5: Commit

```bash
git commit -m "$(cat <<'EOF'
<prefix>: <subject>
EOF
)"
```

Report success and show the commit hash. If a pre-commit hook fails, report the error and do NOT amend — fix the issue and create a new commit.
