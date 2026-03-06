---
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
description: Force re-sync all Bunzl Forge extensions from the central repository
---

# Forge Sync — Force Re-Sync Extensions

Force re-sync all forge-managed extensions from the local bunzl-forge repository. This overwrites local versions of forge-managed files while preserving project-specific files.

## When to Use

- Extensions are out of sync or corrupted
- You want to reset forge files to their canonical versions
- After resolving merge conflicts from a forge update PR

## Step 1: Read Configuration

1. Read `.claude-config.yaml` — fail with a helpful message if missing
2. Extract stacks and version
3. Read `.forge-manifest.yaml` to identify forge-managed files

## Step 2: Locate Forge Repository

Same search logic as forge-init and forge-update.

## Step 3: Confirm with User

Warn the user:

```
This will overwrite ALL forge-managed files with the versions from the local bunzl-forge repo.

Project-specific files (not in .forge-manifest.yaml) will NOT be affected.
The project-specific section in CLAUDE.md will be preserved.

{N} files will be overwritten.

Proceed?
```

Use AskUserQuestion to confirm.

## Step 4: Re-Sync All Extensions

1. For each file in the manifest, delete the local version
2. Copy fresh versions from the forge repo:
   - Shared extensions from `{FORGE_REPO}/shared/.claude/`
   - Stack extensions from `{FORGE_REPO}/stacks/{STACK}/.claude/` for each configured stack
3. Ensure forge management commands are included (forge-init, forge-update, forge-sync)

**Important:** Do NOT touch:
- Files NOT in the forge manifest (project-specific additions)
- The `<!-- bunzl-forge:section:project-specific -->` section in CLAUDE.md

## Step 5: Re-Generate CLAUDE.md Forge Sections

1. Read the current CLAUDE.md
2. Extract the content of `<!-- bunzl-forge:section:project-specific -->` to preserve it
3. Generate fresh CLAUDE.md from the stack template
4. Replace placeholders with values from `.claude-config.yaml`
5. Re-insert the preserved project-specific content
6. Write the result

## Step 6: Update Manifest

Regenerate `.forge-manifest.yaml` with the current file list, timestamp, and enhanced format:
- Each entry must include `path`, `source` (the file's origin path within the bunzl-forge repo), and `checksum` (SHA-256 hash of the file at sync time)
- For CLAUDE.md, include a `sections` sub-field mapping each `<!-- bunzl-forge:section:NAME -->` marker to its origin file
- Compute checksums using `sha256sum` or equivalent for each synced file

## Step 7: Report

```
Forge sync complete.

Re-synced: {N} files from local bunzl-forge
Preserved: {N} project-specific files

CLAUDE.md forge sections refreshed, project-specific section preserved.
```
