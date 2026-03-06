---
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
argument-hint:
description: Initialize project structure and setup user configurations
---

## Context

This command sets up the project structure before implementation begins. Run this FIRST before using `/implement-api`.

## Task: Initialize Project Structure

### Step 1: Move User-Level Configurations

1. **Detect user's home directory:**
   - Windows: `$env:USERPROFILE` or `%USERPROFILE%`
   - Unix/Mac: `$HOME` or `~`

2. **Create user-level `.claude` folder if it doesn't exist:**
   - Windows: `C:\Users\{Username}\.claude`
   - Unix/Mac: `~/.claude`

3. **Move contents from `.claude-user-level` to user's `.claude` folder:**
   - Copy all files and subdirectories
   - Preserve folder structure
   - If files already exist, ask user whether to overwrite

4. **Delete `.claude-user-level` folder** after successful move

5. **Confirm to user:**
   - List what was moved and where
   - Confirm deletion of `.claude-user-level`

### Step 2: Create Reference Documentation Folder

1. **Create `.forge-backend` folder** in project root

2. **Copy (NOT move) the root README.md** to `.forge-backend/README.md`
   - This becomes a reference copy of the original template README
   - Keep the original README.md at root for now (will be replaced in Step 3)
   - **IMPORTANT**: Do NOT move CLAUDE.md - it must stay at the project root
   - **IMPORTANT**: Do NOT move any code or implementation files - all code stays at project root
   - The `.forge-backend` folder is ONLY for documentation reference

### Step 3: Replace Project Root README

Replace the existing `README.md` in project root with new generic content (the original is now saved in `.forge-backend/README.md` for reference).

Create/overwrite `README.md` in project root with the following structure:

```markdown
# Project Name

> Replace this with your actual project name and description

## Overview

This project uses a design-first approach for building production-ready .NET APIs.

**Status:** 🚧 In Development

## Structure

- `/src` - API implementation source code
- `.forge-backend/` - Reference documentation (original template README)
- `.claude/` - Project-level Claude Code configurations
- `docs/` - Project documentation and guides

## Getting Started

### Prerequisites

- .NET 8.0 SDK or later
- PostgreSQL (or your preferred database)
- An OpenAPI/Swagger specification for your API

### Quick Start

1. **Ensure you have an API specification ready**
   - Place your `openapi.yaml` in the project root
   - Or use a custom path when running commands

2. **Implement the API**
   ```bash
   # This will guide you through database design and API implementation
   /implement-api
   ```

3. **Run the application**
   ```bash
   cd .forge-backend
   dotnet run
   ```

## Contributing

### General Best Practices

- **Keep commits small and focused** - One logical change per commit
- **Use descriptive commit messages** with prefixes:
  - `feature:` - New features
  - `fix:` - Bug fixes
  - `refactor:` - Code refactoring
  - `test:` - Adding or updating tests
  - `docs:` - Documentation changes
  - `chore:` - Maintenance tasks
- **Never commit secrets** - No passwords, API keys, or sensitive data
- **Test before committing** - Run `dotnet test` and ensure all tests pass
- **Format your code** - Run `dotnet format` before committing

### Code Standards

- Follow the conventions in [CLAUDE.md](CLAUDE.md)
- Use primary constructors for dependency injection
- Keep controllers thin - business logic belongs in services
- Keep repositories thin - only CRUD operations
- Always include unit tests for new/modified code
- Use AutoMapper for object-object mapping

### Pull Request Guidelines

1. Create a feature branch from `main`
2. Make your changes following the code standards above
3. Ensure all tests pass and code is formatted
4. Submit PR with a clear description of changes
5. Link any related issues

### Security

- **Never commit** `.env` files, `secrets.json`, or credential files
- Use Azure Key Vault or similar for production secrets
- Keep dependencies up to date (`dotnet list package --outdated`)

## Documentation

- [Development Guide](CLAUDE.md) - Detailed development guidelines and conventions
- [Template Reference](.forge-backend/README.md) - Original template documentation
- [API Specification](openapi.yaml) - OpenAPI/Swagger spec

## License

[Specify your license here]
```

### Step 4: Summary

After completing all steps, provide a summary:

```
✅ Project initialized successfully!

What was done:
1. User configurations moved to: [path]
2. Reference folder created: .forge-backend/
3. Original template README copied to: .forge-backend/README.md (for reference)
4. Project root README.md replaced with new generic content
5. CLAUDE.md: Left at root (as required)
6. All implementation code stays at project root

Project Structure:
  /                          (project root)
    ├── .forge-backend/
    │   └── README.md        (original template README for reference)
    ├── .claude/             (project-level configs)
    ├── CLAUDE.md            (project instructions)
    ├── README.md            (new generic project README)
    ├── openapi.yaml         (your API spec)
    └── /src                 (implementation code - created by /implement-api)

Next steps:
1. Replace "Project Name" in README.md with your actual project name
2. Ensure your openapi.yaml is in the project root
3. Run: /implement-api
```

### Important Notes

- This command should only be run ONCE at project setup
- If `.claude-user-level` doesn't exist, skip Step 1 and inform the user
- If `.forge-backend` already exists, warn the user before proceeding
- Always confirm before overwriting existing files
- **CRITICAL**: `.forge-backend` is ONLY for documentation reference (copy of original README)
- **CRITICAL**: All implementation code (`/src`, etc.) stays at project root, NOT in `.forge-backend`
- The folder structure after initialization should be:
  - Project root: CLAUDE.md, README.md (new), openapi.yaml, /src (after /implement-api)
  - `.forge-backend/`: Only README.md (reference copy of original template README)
