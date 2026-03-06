---
allowed-tools: Read, Edit, Write, Glob, Grep
description: Scaffold complete DevOps configuration for a .NET API project
---

## Required Reading

Before scaffolding, use the Read tool to consult:
- [Azure DevOps Guide](docs/guides/azure_devops_guide.md) - Variable syntax, PAT setup, approvals

## Context

Analyze existing project:
- Project type (.NET version, API structure)
- Existing .devops folder contents
- Existing Docker/container configuration
- Infrastructure requirements

## Task

Generate complete DevOps scaffolding for this .NET API project.

### 1. NuGet Configuration

Create `nuget.config` with private feed:
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="BunzlEcomArtifacts" value="https://bunzlweb.pkgs.visualstudio.com/BunzlEcom/_packaging/BunzlEcomArtifacts/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

### 2. Docker Configuration

Create `.dockerignore`:
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
```

Create `Dockerfile` with multi-stage build:
```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["*.csproj", "./"]
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 8080
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

### 3. Azure Pipeline

Create `.devops/azure-pipelines.yaml`:
- Build, test, package stages
- Deploy to sbx, uat, prod environments
- Use variables from `.devops/variables.yml`
- Include `AZURE_DEVOPS_PAT` for NuGet authentication

Create `.devops/variables.yml`:
- Shared variables across environments
- App-specific configuration

### 4. Bicep Infrastructure

If `infra/` doesn't exist, create basic structure:
```
infra/
  main.bicep
  parameters/
    sbx.parameters.json
    uat.parameters.json
    prod.parameters.json
```

## Output Summary

After scaffolding, list:
- Files created/modified
- Manual steps required (PAT generation, environment approvals)
- Configuration values to verify in `.devops/variables.yml`
