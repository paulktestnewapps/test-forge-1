# DevOps Quick Reference

Fast lookup for Azure DevOps pipeline configuration and troubleshooting.

## Required Files Checklist

```
PROJECT ROOT:
☐ .devops/azure-pipelines.yaml
☐ .devops/variables.yml
☐ nuget.config
☐ Dockerfile
☐ .dockerignore

PIPELINE VARIABLES:
☐ AZURE_DEVOPS_PAT (secret, packaging read/write)
```

## Variable Reference Syntax

```yaml
YAML context:
  $(VAR_NAME)
  Example: $(AZURE_DEVOPS_PAT)

Bash script:
  $(VAR_NAME)
  Example: $(AZURE_DEVOPS_PAT)

PowerShell:
  ${env:VAR_NAME}
  Example: ${env:AZURE_DEVOPS_PAT}

Batch:
  %VAR_NAME%
  Example: %AZURE_DEVOPS_PAT%

IMPORTANT:
- Replace '.' with '_'
- Replace spaces with '_'
- Capitalize all letters
```

## Secret Variable Mapping

```yaml
CRITICAL: Secrets must be explicitly mapped as env vars

PATTERN:
- script: dotnet restore
  env:
    AZURE_DEVOPS_PAT: $(AZURE_DEVOPS_PAT)

WITHOUT MAPPING:
Secret variables are NOT accessible in scripts
```

## NuGet Feed Setup

### Command Line

```bash
# Add source
dotnet nuget add source \
  "https://bunzlweb.pkgs.visualstudio.com/BunzlEcom/_packaging/BunzlEcomArtifacts/nuget/v3/index.json" \
  --name "BunzlEcomArtifacts"

# If authentication fails, use interactive
dotnet nuget add source \
  "https://bunzlweb.pkgs.visualstudio.com/BunzlEcom/_packaging/BunzlEcomArtifacts/nuget/v3/index.json" \
  --name "BunzlEcomArtifacts" \
  --interactive
```

### nuget.config File

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="BunzlEcomArtifacts"
         value="https://bunzlweb.pkgs.visualstudio.com/BunzlEcom/_packaging/BunzlEcomArtifacts/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

## AZURE_DEVOPS_PAT Setup

```
STEPS:
1. Azure DevOps → User Settings → Personal Access Tokens
2. Create new token
3. Scopes: ONLY "Packaging: Read & Write"
4. Copy token value
5. Pipeline → Variables → Add variable
6. Name: AZURE_DEVOPS_PAT
7. Mark as: Secret ✓
8. Save

SCOPE REQUIRED:
- Packaging: Read & Write

DO NOT ADD:
- Build permissions
- Code permissions
- Other scopes (security risk)
```

## Environment Approvals

```
REQUIRED APPROVALS:

Resource                  | Approver Required
--------------------------|------------------
DevOps Environment (sbx)  | Environment admin
DevOps Environment (uat)  | Environment admin
DevOps Environment (prod) | Environment admin
Artifacts (NuGet feed)    | Feed admin
Library (variable groups) | Library admin

WHEN:
First pipeline run or new environment deployment

ACTION:
Request approvals BEFORE running pipeline
```

## Standard Pipeline Structure

```yaml
stages:
  - stage: Build
    displayName: 'Build and Test'
    jobs:
      - job: Build
        steps:
          - task: UseDotNet@2
            inputs:
              version: '8.x'

          - script: dotnet format --verify-no-changes
            displayName: 'Verify code formatting'

          - script: dotnet restore
            displayName: 'Restore packages'
            env:
              AZURE_DEVOPS_PAT: $(AZURE_DEVOPS_PAT)

          - script: dotnet build --configuration Release --no-restore
            displayName: 'Build'

          - script: dotnet test --configuration Release --no-build
            displayName: 'Run tests'

  - stage: DeploySbx
    displayName: 'Deploy to Sandbox'
    dependsOn: Build
    condition: succeeded()
    jobs:
      - deployment: Deploy
        environment: 'sbx'

  - stage: DeployUat
    displayName: 'Deploy to UAT'
    dependsOn: DeploySbx
    condition: succeeded()
    jobs:
      - deployment: Deploy
        environment: 'uat'

  - stage: DeployProd
    displayName: 'Deploy to Production'
    dependsOn: DeployUat
    condition: succeeded()
    jobs:
      - deployment: Deploy
        environment: 'prod'
```

## Dockerfile Template

```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy csproj first for layer caching
COPY ["ProjectName.Api/ProjectName.Api.csproj", "ProjectName.Api/"]
RUN dotnet restore "ProjectName.Api/ProjectName.Api.csproj"

# Copy everything else and build
COPY . .
WORKDIR "/src/ProjectName.Api"
RUN dotnet publish -c Release -o /app/publish --no-restore

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 8080
ENTRYPOINT ["dotnet", "ProjectName.Api.dll"]
```

### Key Points

```
MULTI-STAGE:
- Build stage: SDK image (larger)
- Runtime stage: ASP.NET image (smaller)

LAYER CACHING:
- Copy .csproj first
- Run restore
- Then copy rest of files

BENEFITS:
- Faster rebuilds
- Smaller final image
- Only runtime dependencies in production
```

## .dockerignore Template

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

### Purpose

```
PREVENTS:
- Bloated images
- Broken layer caching
- Sensitive file inclusion

MUST EXCLUDE:
- Build outputs (bin, obj)
- IDE files (.vs, .vscode)
- Git history
- Documentation
- Pipeline configs
```

## Common Commands

### Local Development

```bash
# Format code
dotnet format ./ProjectName.Api/ProjectName.Api.csproj

# Build
dotnet build ./ProjectName.Api/ProjectName.Api.csproj

# Run tests
dotnet test ./ProjectName.Test/ProjectName.Test.csproj

# Run application
dotnet run --project ProjectName.Api/ProjectName.Api.csproj

# Watch mode
dotnet watch run --project ProjectName.Api/ProjectName.Api.csproj
```

### User Secrets (Local)

```bash
# Initialize
dotnet user-secrets init

# Add secret
dotnet user-secrets set "YourApi:Database:ConnectionString" "<value>"

# List all secrets
dotnet user-secrets list

# Remove secret
dotnet user-secrets remove "YourApi:Database:ConnectionString"

# Clear all
dotnet user-secrets clear
```

## Troubleshooting Guide

### Pipeline Fails: NuGet Restore

```
ERROR: Unable to load the service index for source

CAUSES:
1. AZURE_DEVOPS_PAT not set
2. Secret not mapped in env
3. PAT expired
4. PAT missing packaging scope

FIX:
1. Verify variable exists in pipeline
2. Check env mapping in script
3. Regenerate PAT if expired
4. Ensure "Packaging: Read & Write" scope
```

### Pipeline Fails: Environment Approval

```
ERROR: Environment 'sbx' requires approval

CAUSE:
First-time deployment to environment

FIX:
1. Request approval from environment admin
2. Wait for approval grant
3. Re-run pipeline
```

### Docker Build Fails: Layer Caching

```
ERROR: COPY failed: no source files found

CAUSE:
.dockerignore excluding required files

FIX:
1. Check .dockerignore patterns
2. Ensure .csproj files not excluded
3. Verify COPY paths in Dockerfile
```

### Restore Fails: Private Package

```
ERROR: Unable to find package 'BunzlEcom.API.Middleware'

CAUSES:
1. nuget.config missing
2. Package source not added
3. Authentication failed

FIX:
1. Add nuget.config to project root
2. Run dotnet nuget add source
3. Use --interactive flag for auth
```

## Variables File Pattern

```yaml
# .devops/variables.yml
variables:
  dotnetVersion: '8.x'
  buildConfiguration: 'Release'
  dockerRegistry: 'youracr.azurecr.io'
  imageName: 'yourapi'

# .devops/azure-pipelines.yaml
variables:
  - template: variables.yml

stages:
  - stage: Build
    jobs:
      - job: Build
        steps:
          - task: UseDotNet@2
            inputs:
              version: $(dotnetVersion)
```

## Security Best Practices

```
SECRETS:
☐ Never commit secrets to repository
☐ Use Azure Key Vault for production secrets
☐ Mark pipeline variables as secret
☐ Use user-secrets for local development
☐ Rotate PATs regularly (90 days)

PERMISSIONS:
☐ Minimal PAT scopes (only packaging)
☐ Environment-specific service principals
☐ Approval gates for prod deployments
☐ Separate variable groups per environment

DOCKER:
☐ Multi-stage builds (smaller images)
☐ Non-root user in runtime stage
☐ Scan images for vulnerabilities
☐ Use specific base image tags (not :latest)
```

## Pre-Deployment Checklist

```
CODE:
☐ dotnet format passes (no changes)
☐ dotnet build succeeds
☐ dotnet test passes (all tests green)

CONFIGURATION:
☐ nuget.config present
☐ Dockerfile present
☐ .dockerignore present
☐ azure-pipelines.yaml present
☐ variables.yml present

PIPELINE:
☐ AZURE_DEVOPS_PAT variable set
☐ Environment approvals requested
☐ Variable groups configured
☐ Service connections created

INFRASTRUCTURE:
☐ Azure resources exist (App Service, etc.)
☐ App Configuration values verified
☐ Connection strings validated
☐ Bicep parameters match environment
```

## Bicep Variable Verification

```
BEFORE DEPLOYMENT:

1. Check Azure App Configuration:
   - Navigate to sbx/uat/prod App Configuration
   - Verify all required keys exist
   - Confirm values are correct

2. Verify Resource Names:
   - App Service name
   - SQL Database name
   - Storage account name
   - Must match variables.yml

3. Validate Connection Strings:
   - Database connection string
   - Storage connection string
   - Any external service endpoints

MISMATCH = DEPLOYMENT FAILURE
```

## Quick Command Reference

```bash
# Pipeline validation (local)
dotnet format --verify-no-changes
dotnet build
dotnet test

# Docker local testing
docker build -t yourapi:local .
docker run -p 8080:8080 yourapi:local

# NuGet troubleshooting
dotnet nuget list source
dotnet nuget remove source <name>
dotnet restore --verbosity detailed

# Secrets management
dotnet user-secrets init
dotnet user-secrets set "Key" "Value"
dotnet user-secrets list
```
