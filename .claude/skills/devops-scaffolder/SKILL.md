---
name: devops-scaffolder
description: Generate DevOps configuration files (Dockerfile, .dockerignore, nuget.config, azure-pipelines.yaml) following project standards. Use when scaffolding CI/CD infrastructure for .NET APIs.
allowed-tools: Read, Edit, Write, Glob, Grep
---

# DevOps Scaffolder

## Purpose

Generates DevOps infrastructure files following established conventions for .NET API projects with Azure DevOps.

## When to Use

- Setting up CI/CD for a new project
- Adding missing DevOps configuration files
- Standardizing existing DevOps setup to match conventions

## File Templates

### nuget.config

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="BunzlEcomArtifacts" value="https://bunzlweb.pkgs.visualstudio.com/BunzlEcom/_packaging/BunzlEcomArtifacts/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

### .dockerignore

```
**/.git
**/.vs
**/.vscode
**/bin
**/obj
**/.dockerignore
**/Dockerfile*
**/*.md
**/*.yml
**/node_modules
.devops/
infra/
*.sln
.gitignore
```

### Dockerfile (.NET 8 API)

```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy csproj and restore (for layer caching)
COPY ["ProjectName.Api/ProjectName.Api.csproj", "ProjectName.Api/"]
RUN dotnet restore "ProjectName.Api/ProjectName.Api.csproj"

# Copy everything else and build
COPY . .
WORKDIR "/src/ProjectName.Api"
RUN dotnet publish -c Release -o /app/publish --no-restore

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser

COPY --from=build /app/publish .
EXPOSE 8080
ENTRYPOINT ["dotnet", "ProjectName.Api.dll"]
```

### azure-pipelines.yaml

```yaml
trigger:
  branches:
    include:
      - main
      - develop

variables:
  - template: variables.yml

stages:
  - stage: Build
    displayName: 'Build and Test'
    jobs:
      - job: Build
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: UseDotNet@2
            displayName: 'Use .NET 8'
            inputs:
              version: '8.x'

          - script: dotnet restore
            displayName: 'Restore packages'
            env:
              AZURE_DEVOPS_PAT: $(AZURE_DEVOPS_PAT)

          - script: dotnet build --configuration Release --no-restore
            displayName: 'Build'

          - script: dotnet format --verify-no-changes
            displayName: 'Check formatting'

          - script: dotnet test --configuration Release --no-build --logger trx
            displayName: 'Run tests'

          - task: Docker@2
            displayName: 'Build Docker image'
            inputs:
              command: 'build'
              Dockerfile: '**/Dockerfile'
              tags: '$(Build.BuildId)'

  - stage: DeploySbx
    displayName: 'Deploy to Sandbox'
    dependsOn: Build
    condition: succeeded()
    jobs:
      - deployment: Deploy
        environment: 'sbx'
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo 'Deploy to Sandbox'

  - stage: DeployUat
    displayName: 'Deploy to UAT'
    dependsOn: DeploySbx
    condition: succeeded()
    jobs:
      - deployment: Deploy
        environment: 'uat'
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo 'Deploy to UAT'

  - stage: DeployProd
    displayName: 'Deploy to Production'
    dependsOn: DeployUat
    condition: succeeded()
    jobs:
      - deployment: Deploy
        environment: 'prod'
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo 'Deploy to Production'
```

### variables.yml

```yaml
variables:
  dotnetVersion: '8.x'
  buildConfiguration: 'Release'
  # Add project-specific variables below
```

## Azure DevOps Variable Reference

Use the correct syntax depending on context:

| Context | Syntax | Example |
|---------|--------|---------|
| YAML | `$(VAR_NAME)` | `$(AZURE_DEVOPS_PAT)` |
| Bash script | `$(VAR_NAME)` | `$(AZURE_DEVOPS_PAT)` |
| PowerShell | `${env:VAR_NAME}` | `${env:AZURE_DEVOPS_PAT}` |
| Batch | `%VAR_NAME%` | `%AZURE_DEVOPS_PAT%` |

## Secret Variables

Map secrets explicitly in script tasks:

```yaml
- script: dotnet restore
  env:
    AZURE_DEVOPS_PAT: $(AZURE_DEVOPS_PAT)
```

## Manual Setup Steps

Document these for the user after scaffolding:

1. **Generate AZURE_DEVOPS_PAT**
   - Create PAT with `packaging:read/write` permissions only
   - Add as pipeline variable (secret)

2. **Approve Pipeline Access**
   - DevOps Environments (sbx, uat, prod)
   - DevOps Artifacts (NuGet feed)
   - DevOps Library (variable groups)

## Checklist

- [ ] nuget.config includes private feed (BunzlEcomArtifacts)
- [ ] BunzlEcom.API.Middleware package installed in .csproj
- [ ] Microsoft.Identity.Web package installed (v3.0.0)
- [ ] Program.cs configured with exception handlers and auth (if needed)
- [ ] .dockerignore excludes build artifacts and config
- [ ] Dockerfile uses multi-stage build
- [ ] Dockerfile copies csproj first for layer caching
- [ ] Pipeline includes all stages (build, sbx, uat, prod)
- [ ] Pipeline restores with AZURE_DEVOPS_PAT
- [ ] Variables in separate variables.yml file
- [ ] README includes manual setup steps

## Required NuGet Packages

**CRITICAL: Every .NET API project MUST include:**
```xml
<PackageReference Include="BunzlEcom.API.Middleware" Version="latest" />
<PackageReference Include="Microsoft.Identity.Web" Version="3.0.0" />
```

Verify with:
```bash
dotnet list package | grep -E "BunzlEcom.API.Middleware|Microsoft.Identity.Web"
```

If missing, install:
```bash
dotnet add package BunzlEcom.API.Middleware
dotnet add package Microsoft.Identity.Web --version 3.0.0
```

## Code Examples

**Bunzl API Middleware**: [docs/examples/BUNZL-API-MIDDLEWARE-USAGE-EXAMPLES.md](../../../docs/examples/BUNZL-API-MIDDLEWARE-USAGE-EXAMPLES.md)
**NuGet Configuration**: [docs/examples/NUGET-CONFIG-EXAMPLE.md](../../../docs/examples/NUGET-CONFIG-EXAMPLE.md)
**Program.cs/Startup**: [docs/examples/DOTNET-STARTUP-FILE-EXAMPLE.md](../../../docs/examples/DOTNET-STARTUP-FILE-EXAMPLE.md)
**Simple Project Structure**: [docs/examples/PROJECT-STRUCTURE.md](../../../docs/examples/PROJECT-STRUCTURE.md)
**Clean Architecture Structure**: [docs/examples/PROJECT-STRUCTURE-FOR-LARGER-PROJECT.md](../../../docs/examples/PROJECT-STRUCTURE-FOR-LARGER-PROJECT.md)

**Related Guides**:
- [Bunzl API Middleware Quick Reference](../../guides/bunzl-api-middleware-quick-reference.md)
- [DevOps Quick Reference](../../guides/devops-quick-reference.md)
