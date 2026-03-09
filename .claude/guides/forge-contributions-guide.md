# Forge Contributions Guide

How to contribute `.claude/` extensions you've built locally back to the central bunzl-forge repository.

## Overview

When you create a new skill, agent, or guide in your project's `.claude/` directory — or improve an existing forge-managed file — that work may be valuable to the whole team. The forge contribution workflow captures your intent and stages the changes for review via a GitHub Actions-generated PR.

> **Prerequisite: GitHub Actions workflow required.** The automated PR flow only works if your downstream project has the `forge-contribute.yml` workflow installed and configured. See [Setting Up the GitHub Actions Workflow](#setting-up-the-github-actions-workflow) below — this is a one-time setup per project.

## The Workflow

```
Your project                     bunzl-forge
─────────────────────────────    ──────────────────────────
1. Build something useful
   (.claude/agents/my-agent.md)

2. Run /forge-contribute
   → writes .forge-contributions.md

3. Commit & push both files
   ──────────────────────────────►  GH Actions detects
                                    .forge-contributions.md
                                    → opens PR with your files
                                    + changelog as context
```

## What Gets Detected

`/forge-contribute` scans for three categories of changes:

| Category | How it's detected |
|----------|-------------------|
| New `.claude/` files | Files under `.claude/` not listed in `.forge-manifest.yaml` |
| Modified forge files | Files in the manifest whose checksum no longer matches |
| CLAUDE.md project-specific content | Non-empty `<!-- bunzl-forge:section:project-specific -->` block |

## The `.forge-contributions.md` File

This file is written to your project root by `/forge-contribute`. It is **current state only** — re-running the command overwrites it entirely.

### Purpose

- Provides human-readable context for the GH Actions workflow and PR reviewer
- Documents which files are proposed for contribution and to which target path in bunzl-forge
- Distinguishes between "please upstream this" and "documenting a local-only change"

### Location

```
my-project/
├── .forge-contributions.md    ← written here
├── .forge-manifest.yaml
├── .claude-config.yaml
└── .claude/
```

### Structure

```markdown
# Forge Contributions

generated_at: 2026-03-06T14:30:00Z
project: my-project
stacks: backend-dotnet

---

## Proposed Contributions

### my-new-agent.md — new

- **Local path:** `.claude/agents/my-new-agent.md`
- **Proposed target:** `stacks/backend-dotnet/.claude/agents/my-new-agent.md`
- **Type:** `new`
- **Description:** Agent that validates EF Core migrations against the OpenAPI spec before they run.
- **Caveats:** Depends on dotnet-ef CLI being available in the project.

---

## CLAUDE.md Excerpts

<!-- Optional — only present if user opted to include project-specific content -->

---

## Notes for Reviewer

None
```

### Editing the File

You can edit the description fields directly after generation. The GH Actions workflow reads the file structurally (by heading and bullet labels), so keep the format intact. The `generated_at`, `project`, `stacks`, and metadata lines are informational only.

## Step-by-Step

1. Build or modify a skill, agent, or guide in `.claude/`
2. When ready to contribute, run `/forge-contribute` in Claude Code
3. Answer the prompts — describe what each file does and where it should live in bunzl-forge
4. Review `.forge-contributions.md` and edit descriptions if needed
5. Commit both the `.claude/` changes and `.forge-contributions.md`
6. Push — GH Actions will detect the file and open a PR to bunzl-forge

## Setting Up the GitHub Actions Workflow

> **Required once per downstream project.** Without this workflow, pushing `.forge-contributions.md` will do nothing — no PR to bunzl-forge will be opened.

### 1. Copy the workflow template

From your downstream project root, run:

```bash
mkdir -p .github/workflows
cp ../bunzl-forge/templates/forge-contribute.yml .github/workflows/forge-contribute.yml
```

### 2. Add a repository secret

In your downstream project's GitHub repository settings (**Settings → Secrets and variables → Actions**), add:

| Secret name | Value |
|-------------|-------|
| `BUNZL_FORGE_CONTRIBUTE_PAT` | A GitHub Personal Access Token with `repo` scope, owned by someone with write access to the bunzl-forge repository |

### 3. Set the target repository

Open `.github/workflows/forge-contribute.yml` and set the `BUNZL_FORGE_REPO` environment variable to your bunzl-forge repository:

```yaml
env:
  BUNZL_FORGE_REPO: your-org/bunzl-forge   # default: bunzl/bunzl-forge
```

### 4. Commit and push the workflow file

```bash
git add .github/workflows/forge-contribute.yml
git commit -m "chore: add forge contribution workflow"
git push
```

Once in place, every push that adds or changes `.forge-contributions.md` will automatically open a PR to bunzl-forge with your files and changelog as context.

## Tips

- **Describe intent, not implementation.** The reviewer can read the file; they need to know *why* it's useful and *who* it's for.
- **Be specific about the target path.** If you know which stack the file belongs to, say so. "TBD" is fine if you're unsure.
- **One contribution at a time is fine.** You don't need to batch up multiple files — run `/forge-contribute` whenever you have something worth sharing.
- **Local-only changes are valid.** If you modified a forge file for project-specific reasons, you can document it as local-only. The file records it without proposing an upstream change.

## What Happens After You Push

The GH Actions workflow (to be implemented) will:
1. Detect that `.forge-contributions.md` was added or changed
2. Read the file to identify proposed source paths
3. Copy the files to their proposed target paths in a new branch
4. Open a PR to bunzl-forge with `.forge-contributions.md` as the PR description context

The PR is a proposal — forge maintainers review, adjust placement if needed, and merge or close.
