---
name: pipeline-template-generator
description: Generate CI/CD pipeline configurations for Azure Pipelines, GitHub Actions, and GitLab CI. Use when creating build, test, and deployment pipelines.
allowed-tools: Read, Edit, Write, Glob, Grep
---

# Pipeline Template Generator

## Purpose

Generates CI/CD pipeline configurations following best practices for .NET API projects.

## When to Use

- Creating new CI/CD pipelines
- Adding stages to existing pipelines
- Standardizing pipeline configuration across projects

## Supported Platforms

- Azure Pipelines (primary)
- GitHub Actions
- GitLab CI

## Azure Pipelines Template

### Main Pipeline (azure-pipelines.yaml)

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

          - script: dotnet format --verify-no-changes
            displayName: 'Check formatting'

          - script: dotnet restore
            displayName: 'Restore packages'
            env:
              AZURE_DEVOPS_PAT: $(AZURE_DEVOPS_PAT)

          - script: dotnet build --configuration $(buildConfiguration) --no-restore
            displayName: 'Build'

          - script: dotnet test --configuration $(buildConfiguration) --no-build --logger trx --collect:"XPlat Code Coverage"
            displayName: 'Run tests'

          - task: PublishTestResults@2
            displayName: 'Publish test results'
            inputs:
              testResultsFormat: 'VSTest'
              testResultsFiles: '**/*.trx'

          - task: Docker@2
            displayName: 'Build Docker image'
            inputs:
              command: 'build'
              Dockerfile: '**/Dockerfile'
              tags: '$(Build.BuildId)'

  - stage: DeploySbx
    displayName: 'Deploy to Sandbox'
    dependsOn: Build
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/develop'))
    jobs:
      - deployment: Deploy
        environment: 'sbx'
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo 'Deploy to Sandbox'
                  displayName: 'Deploy'

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
                  displayName: 'Deploy'

  - stage: DeployProd
    displayName: 'Deploy to Production'
    dependsOn: DeployUat
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: Deploy
        environment: 'prod'
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo 'Deploy to Production'
                  displayName: 'Deploy'
```

### Variables File (variables.yml)

```yaml
variables:
  dotnetVersion: '8.x'
  buildConfiguration: 'Release'
  # Project-specific variables
  projectName: 'MyApi'
  dockerRegistry: 'myregistry.azurecr.io'
```

## Variable Reference Syntax

| Context | Syntax | Example |
|---------|--------|---------|
| YAML | `$(VAR_NAME)` | `$(AZURE_DEVOPS_PAT)` |
| Bash | `$(VAR_NAME)` | `$(AZURE_DEVOPS_PAT)` |
| PowerShell | `${env:VAR_NAME}` | `${env:AZURE_DEVOPS_PAT}` |
| Batch | `%VAR_NAME%` | `%AZURE_DEVOPS_PAT%` |

## Secret Variables

Map secrets explicitly in script tasks:

```yaml
- script: dotnet restore
  displayName: 'Restore packages'
  env:
    AZURE_DEVOPS_PAT: $(AZURE_DEVOPS_PAT)
```

## GitHub Actions Template

```yaml
name: Build and Deploy

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  DOTNET_VERSION: '8.x'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}

      - name: Restore
        run: dotnet restore

      - name: Build
        run: dotnet build --configuration Release --no-restore

      - name: Test
        run: dotnet test --configuration Release --no-build

  deploy-sbx:
    needs: build
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: sbx
    steps:
      - name: Deploy to Sandbox
        run: echo "Deploying to Sandbox"
```

## Checklist

- [ ] Trigger branches configured
- [ ] Variables in separate file
- [ ] .NET version specified
- [ ] Format check included
- [ ] Tests with coverage
- [ ] Docker build included
- [ ] Environment-specific stages
- [ ] Secrets mapped in scripts
- [ ] Approval gates for production
