---
allowed-tools: Read, Edit, Write, Glob, Grep
argument-hint: [platform] (azure-pipelines|github-actions|gitlab-ci)
description: Generate CI/CD pipeline configuration
---

## Context

Analyze:
- Project type and structure
- Build requirements
- Test frameworks
- Deployment targets
- Existing .devops folder if present

## Task

Generate CI/CD pipeline for: $ARGUMENTS

1. **Build Stage**
   - Restore dependencies (include NuGet feed authentication for Azure Artifacts)
   - Compile/build
   - Run linters and `dotnet format`

2. **Test Stage**
   - Unit tests
   - Integration tests
   - Code coverage

3. **Package Stage**
   - Create artifacts
   - Container build (if applicable)

4. **Deploy Stages**
   - Sandbox (sbx)
   - UAT
   - Production (with approval gates)

Include:
- Environment variables
- Secret references (use variables files for Azure Pipelines)
- Caching strategies
- Parallel job optimization

## Azure Pipelines Specifics

When generating Azure Pipelines (`azure-pipelines.yaml`):

### Variables Files
Use separate variables files in `.devops/variables.yml`:
```yaml
variables:
  - template: .devops/variables.yml
```

### AZURE_DEVOPS_PAT Variable
Add `AZURE_DEVOPS_PAT` to pipeline variables for NuGet feed authentication:
- Reference in YAML: `$(AZURE_DEVOPS_PAT)`
- In Bash scripts: `$(AZURE_DEVOPS_PAT)`
- In PowerShell scripts: `${env:AZURE_DEVOPS_PAT}`
- In Batch scripts: `%AZURE_DEVOPS_PAT%`

**Important**: PAT must be generated with `packaging:read/write` permissions.

### Secret Variables
Map secret variables explicitly in scripts:
```yaml
- script: |
    echo "Using secure variable"
  env:
    AZURE_DEVOPS_PAT: $(AZURE_DEVOPS_PAT)
```

### Environment Approvals
Pipeline requires manual approval for access to:
- DevOps Environments (sbx, uat, prod)
- DevOps Artifacts (NuGet feed)
- DevOps Library (variable groups)

### NuGet Configuration
Include `nuget.config` for private feeds:
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="BunzlEcomArtifacts" value="https://bunzlweb.pkgs.visualstudio.com/BunzlEcom/_packaging/BunzlEcomArtifacts/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

### Docker Configuration
Include `.dockerignore` to avoid bloating images and breaking layer caching.
