---
name: forge-init
description: Initialize a new project with Bunzl Forge extensions (stack selection, PRD, CLAUDE.md generation)
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
user-invocable: true
---

# Forge Init — Bootstrap a New Project

Initialize the current project with Bunzl Forge Claude Code extensions. This replaces the old PowerShell CLI with an AI-native workflow.

## Prerequisites

- Current directory should be a git repo (or will become one)
- The `bunzl-forge` repo must be cloned locally

## Step 1: Locate the Bunzl Forge Repository

Search for the `bunzl-forge` repo by checking these paths in order:
1. `$HOME/projects/bunzl-forge`
2. `$HOME/repos/bunzl-forge`
3. `C:/repos/bunzl-forge` (Windows)
4. `C:/projects/bunzl-forge` (Windows)
5. Check if `BUNZL_FORGE_PATH` environment variable is set

Verify the path by checking for the `shared/.claude/skills/` directory.

If not found, ask the user: "Where is your bunzl-forge repository cloned?"

## Step 2: Check for Existing Configuration

Check if `.claude-config.yaml` already exists in the current directory.
- If it exists, warn the user and ask if they want to reinitialize (this will overwrite).
- If they decline, stop.

## Step 3: Interactive Setup

Use AskUserQuestion to gather project configuration:

**Question 1 — Stack Selection** (multiSelect: true)
"Which tech stack(s) does this project use?"
Options:
- `backend-dotnet` — ".NET 8, EF Core, PostgreSQL"
- `frontend` — "React/Next.js, TypeScript, Tailwind, shadcn/ui"
- `backend-python` — "FastAPI/Django (coming soon)"
- `full-stack` — "Combined frontend + backend"

**Question 2 — PRD Setup**
"Do you have a Product Requirements Document (PRD)?"
Options:
- `Yes, I have a PRD` — "I'll provide the file path"
- `No, help me create one` — "Quick Q&A to generate a PRD"
- `Skip for now` — "I'll add a PRD later"

## Step 4: Prepare the docs/ Folder

Before prompting the user, create the `docs/` directory and write a placeholder OpenAPI spec:

```bash
mkdir -p docs
```

Write the following file to `docs/openapi.yaml` **only if it does not already exist**:

```yaml
# ============================================================
# OpenAPI Specification — PLACEHOLDER
# ============================================================
#
# Paste your OpenAPI spec here (YAML or JSON converted to YAML).
#
# Why this matters:
#   This file is the single most valuable input you can give
#   Claude for a backend project. With a spec here, Claude can
#   automatically generate:
#     - EF Core entities and DbContext
#     - Database migration scripts
#     - API controller stubs and route definitions
#     - Request/response DTOs and validators
#     - Unit and integration test scaffolding
#
#   Without it, Claude has to infer your API surface from prose,
#   which produces lower-quality, less consistent output.
#
# How to populate this file:
#   1. If you already have a spec: paste or copy it here.
#   2. If you don't have one yet: run /forge-init and choose
#      "scaffold one" — Claude will walk you through it.
#   3. Alternatively, export it from your API tool (Postman,
#      Insomnia, Swagger UI, etc.) and replace this file.
#
# Minimum viable spec (replace with your own):
# ------------------------------------------------------------

openapi: "3.1.0"
info:
  title: "TODO — Replace with your API title"
  version: "1.0.0"
  description: "TODO — Paste your OpenAPI specification here."
paths: {}
```

Then prompt the user with the following message (display it clearly, formatted):

---
**Before we continue — gather your project documentation.**

If you have any existing project documentation, please add it to the `docs/` folder now. This includes:

- Requirements docs, briefs, or design specs
- User stories or feature lists
- Architecture diagrams or technical notes

> **Highly recommended: OpenAPI Specification**
>
> If this project has a backend API (existing or planned), adding an OpenAPI spec now will **dramatically improve** the quality of everything Claude generates — entities, endpoints, EF Core models, migration scripts, API implementations, and tests. It is the single most valuable input you can provide for a backend project.
>
> If you have one, drop it in `docs/openapi.yaml` now. If not, don't worry — the next step will offer to scaffold one with you.

Press Enter when ready, or let me know if you have nothing to add.

---

Wait for confirmation before proceeding.

## Step 5: Handle PRD

**If user has a PRD:**
- Ask for the file path
- Copy it to `docs/PRD.md`

**If user wants to generate one:**
Ask these questions conversationally (not all at once):
1. What is the primary purpose of this application?
2. Who are the main users?
3. What are the core features? (list them)
4. Any external system integrations?
5. Any non-functional requirements? (performance, security, etc.)

Then generate a PRD using the template at `{FORGE_REPO}/templates/PRD.md`, filling in the answers. Write to `docs/PRD.md`.

## Step 5b: Derive Project Name

Read all files in `docs/` and use their content to suggest 3–5 project name options. Present them to the user and ask them to select one or provide their own. Use the chosen name (and derive a one-line description from the docs) as `{PROJECT_NAME}` and `{PROJECT_DESCRIPTION}` for the rest of the flow.

## Step 6: Handle OpenAPI Spec

Only ask this if a backend stack (`backend-dotnet`, `backend-python`, or `full-stack`) was selected.

Ask the user:
"Do you have an existing OpenAPI specification? (Highly recommended for backend projects)"
Options:
- `Yes, I have one` — ask for the file path, copy to `docs/openapi.yaml`
- `No — help me scaffold one` — walk through a conversational API design process to generate a starter `docs/openapi.yaml`
- `Skip for now` — continue without one (mention it can be added later)

## Step 7: Create Project Structure

Using Bash, create the following directories (`docs/` was already created in Step 4):
```bash
mkdir -p .claude/agents
mkdir -p .claude/skills
mkdir -p .claude/guides
```

## Step 8: Generate `.claude-config.yaml`

Write this file to the project root:

```yaml
# Bunzl Forge Configuration
# Generated: {TODAY'S DATE}

stacks:
  - {SELECTED_STACKS}

project:
  name: "{PROJECT_NAME}"
  description: "{PROJECT_DESCRIPTION}"
```

## Step 9: Sync Extensions

Copy extensions from the forge repo to the project's `.claude/` directory:

1. **Shared extensions first** — Copy from `{FORGE_REPO}/shared/.claude/` (agents, skills, guides)
2. **Stack-specific extensions** — For each selected stack, copy from `{FORGE_REPO}/stacks/{STACK}/.claude/` (agents, skills, guides, workflows)
3. **Forge management skills** — Ensure the forge skills themselves are copied: the `forge-init`, `forge-update`, and `forge-sync` skill directories

Skills are directories, not flat files — copy entire skill directories:
```bash
cp -r {FORGE_REPO}/shared/.claude/skills/ .claude/skills/
```

Use `cp -r` via Bash. If files already exist, overwrite them (this is initial setup).

## Step 10: Generate CLAUDE.md

1. Read the primary stack's CLAUDE.md template from `{FORGE_REPO}/stacks/{PRIMARY_STACK}/CLAUDE.md`
2. Replace placeholders:
   - `{{PROJECT_NAME}}` → project name
   - `{{PROJECT_DESCRIPTION}}` → project description
3. If no PRD was created, remove the `<!-- bunzl-forge:section:prd-reference -->` block
4. If no OpenAPI spec was provided, remove the `<!-- bunzl-forge:section:openapi-reference -->` block
5. Write the result to `CLAUDE.md` at the project root

## Step 11: Write Forge Manifest

Write `.forge-manifest.yaml` to track which files are managed by forge. Each entry includes the local path, the source path within the bunzl-forge repo, and a SHA-256 checksum for change detection.

Compute checksums using `sha256sum` (Unix/Git Bash) or `Get-FileHash` (PowerShell) for each copied file.

```yaml
# Auto-generated by /forge-init — do not edit manually
synced_at: "{TIMESTAMP}"

managed_files:
  - path: .claude/agents/api-implementer.md
    source: stacks/backend-dotnet/.claude/agents/api-implementer.md
    checksum: "sha256:{HASH}"

  - path: .claude/skills/forge-init/SKILL.md
    source: shared/.claude/skills/forge-init/SKILL.md
    checksum: "sha256:{HASH}"

  - path: .claude/guides/security-guide.md
    source: shared/.claude/guides/security-guide.md
    checksum: "sha256:{HASH}"

  # ... entry for every file copied from forge

  - path: CLAUDE.md
    source: stacks/backend-dotnet/CLAUDE.md
    checksum: "sha256:{HASH}"
    sections:
      - name: project-info
        origin: template-placeholder
      - name: prd-reference
        origin: stacks/backend-dotnet/CLAUDE.md
      - name: openapi-reference
        origin: stacks/backend-dotnet/CLAUDE.md
      - name: security
        origin: shared/CLAUDE.md.common
      - name: quality-commands
        origin: shared/CLAUDE.md.common
      - name: project-specific
        origin: local
```

Use `find .claude/ -name "*.md" -type f` to generate the file list. For each file, record its `source` path based on which forge directory it was copied from (shared vs. stack-specific).

## Step 12: Summary

Report to the user what was created:

```
Bunzl Forge initialized successfully!

Created:
  .claude-config.yaml     — Extension configuration
  .forge-manifest.yaml    — Tracks forge-managed files
  CLAUDE.md               — Project instructions for Claude
  .claude/                — Extensions (agents, skills, guides)
  docs/PRD.md             — Product Requirements Document (if created)
  docs/openapi.yaml       — OpenAPI specification (if provided)

Forge skills available:
  /forge-update           — Pull latest extensions from bunzl-forge and intelligently merge them,
                            preserving any local edits. Use this in your normal update workflow.
  /forge-sync             — Overwrite all forge-managed extensions with the canonical versions
                            from bunzl-forge, discarding local edits. Use this to recover from
                            a corrupted or broken state.

To get updates:
  1. Pull latest changes in your local bunzl-forge clone
  2. Run /forge-update or /forge-sync to apply changes

Next steps:
  1. Review CLAUDE.md and customize the project-specific section at the bottom
  2. Review docs/PRD.md and refine requirements (if created)
  3. Start coding with Claude Code!
```
