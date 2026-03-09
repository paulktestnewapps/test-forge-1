---
name: forge-update
description: Update Bunzl Forge extensions to the latest version with smart merging
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
user-invocable: true
---

# Forge Update — Update Extensions from Local Forge

Pull the latest extensions from the local bunzl-forge repository and intelligently merge them into the current project.

## Step 1: Read Current Configuration

1. Read `.claude-config.yaml` from the project root — fail with a helpful message if it doesn't exist ("Run /forge-init first")
2. Extract: stacks, project name, project description
3. Read `.forge-manifest.yaml` to get the list of forge-managed files

## Step 2: Locate Forge Repository

Find the bunzl-forge repo using the same search logic as forge-init:
1. `$HOME/projects/bunzl-forge`
2. `$HOME/repos/bunzl-forge`
3. `C:/repos/bunzl-forge` or `C:/projects/bunzl-forge`
4. `BUNZL_FORGE_PATH` environment variable

Verify the path by checking for the `shared/.claude/skills/` directory.

## Step 3: Identify Changes

For each file in `.forge-manifest.yaml`:
1. Read the local version of the file
2. Read the corresponding file from the forge repo
3. Categorize each file:
   - **Unchanged locally** — Local matches the previous forge version → safe to replace
   - **Modified locally** — Local differs from what forge originally provided → needs merge
   - **Deleted locally** — File was in manifest but no longer exists → ask user
   - **New in forge** — File exists in forge but not in manifest → new addition

Also check for new files in the forge repo that aren't in the manifest (new agents, skills, etc. added since last sync).

## Step 4: Report and Confirm

Present a summary to the user:

```
Forge Update Summary

Files to update:        {N} (unchanged locally, safe to replace)
Files with local edits: {N} (will need review)
New files to add:       {N}
Deleted locally:        {N}

Proceed with update?
```

Use AskUserQuestion to confirm.

## Step 5: Apply Updates

**For unchanged files:** Replace with the new forge version directly.

**For locally modified files:** Read both the local version and the new forge version. Use your understanding of the content to intelligently merge:
- If the changes don't conflict (e.g., forge added a new section, user modified a different section), merge both changes
- If changes conflict, present both versions to the user and ask which to keep, or suggest a merged version

**For new files:** Copy from forge repo to the appropriate `.claude/` subdirectory.

**For deleted files:** Ask the user if they want to restore them from the new forge version or keep them deleted.

## Step 6: Update CLAUDE.md

Read the current `CLAUDE.md` and the new stack template from forge.

**Smart merge strategy:**
- Sections delimited by `<!-- bunzl-forge:section:NAME -->` markers are forge-managed — update these from the template
- The `<!-- bunzl-forge:section:project-specific -->` section is NEVER overwritten — this is the user's custom content
- Replace `{{PROJECT_NAME}}` and `{{PROJECT_DESCRIPTION}}` placeholders with values from config
- Preserve any custom sections the user added outside of forge markers

## Step 7: Update Configuration

1. Regenerate `.forge-manifest.yaml` with the updated file list and new timestamp, using the enhanced format:
   - Each entry must include `path`, `source` (the file's origin path within the bunzl-forge repo), and `checksum` (SHA-256 hash of the file at sync time)
   - For CLAUDE.md, include a `sections` sub-field mapping each `<!-- bunzl-forge:section:NAME -->` marker to its origin file
   - Compute checksums using `sha256sum` or equivalent for each synced file

## Step 8: Summary

Report what was done:

```
Forge update complete!

Updated:  {list of updated files}
Added:    {list of new files}
Merged:   {list of files that needed merging}
Skipped:  {list of files user chose to skip}

CLAUDE.md: {updated/unchanged}

To get future updates:
  1. Pull latest changes in your local bunzl-forge clone
  2. Run /forge-update or /forge-sync to apply changes
```
