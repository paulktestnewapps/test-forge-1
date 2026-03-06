---
name: environment-matrix
description: Define and document environment configurations, promotion workflows, and environment-specific settings. Use when planning deployments across sbx, uat, and prod environments.
allowed-tools: Read, Edit, Write, Glob, Grep
---

# Environment Matrix

## Purpose

Documents environment configurations, differences, and promotion workflows for multi-environment deployments.

## When to Use

- Planning environment strategy
- Documenting environment differences
- Creating environment-specific configurations
- Setting up promotion workflows

## Standard Environments

| Environment | Purpose | Deployment Trigger | Approval |
|-------------|---------|-------------------|----------|
| sbx (Sandbox) | Development testing | Push to develop | None |
| uat (UAT) | User acceptance testing | After sbx success | Optional |
| prod (Production) | Live system | After uat success | Required |

## Environment Configuration Matrix

### Resource Sizing

| Resource | sbx | uat | prod |
|----------|-----|-----|------|
| App Service SKU | B1 | B2 | P1v3 |
| SQL DTU | 10 | 50 | 100 |
| Container CPU | 0.5 | 0.5 | 1.0 |
| Container Memory | 1Gi | 1Gi | 2Gi |
| Min Replicas | 1 | 1 | 2 |
| Max Replicas | 2 | 3 | 10 |

### Feature Flags

| Feature | sbx | uat | prod |
|---------|-----|-----|------|
| Debug Logging | ON | ON | OFF |
| Swagger UI | ON | ON | OFF |
| Detailed Errors | ON | ON | OFF |
| Rate Limiting | OFF | ON | ON |
| Caching | OFF | ON | ON |

### Connection Endpoints

| Service | sbx | uat | prod |
|---------|-----|-----|------|
| Database | sbx-sql.database.windows.net | uat-sql.database.windows.net | prod-sql.database.windows.net |
| Cache | sbx-redis.redis.cache.windows.net | uat-redis.redis.cache.windows.net | prod-redis.redis.cache.windows.net |
| App Config | sbx-appconfig.azconfig.io | uat-appconfig.azconfig.io | prod-appconfig.azconfig.io |

## Pipeline Variables Template

### variables.yml

```yaml
variables:
  # Shared
  dotnetVersion: '8.x'
  buildConfiguration: 'Release'

  # Environment-specific (override in stages)
  ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/develop') }}:
    environment: 'sbx'
    resourceGroup: 'rg-myapi-sbx'
    appServiceName: 'myapi-sbx-app'

  ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/main') }}:
    environment: 'prod'
    resourceGroup: 'rg-myapi-prod'
    appServiceName: 'myapi-prod-app'
```

### Environment-Specific Stage

```yaml
- stage: DeploySbx
  displayName: 'Deploy to Sandbox'
  variables:
    environment: 'sbx'
    resourceGroup: 'rg-myapi-sbx'
    appServiceName: 'myapi-sbx-app'
    appConfigEndpoint: 'https://myapi-sbx-appconfig.azconfig.io'
  jobs:
    - deployment: Deploy
      environment: 'sbx'
      strategy:
        runOnce:
          deploy:
            steps:
              - task: AzureWebApp@1
                inputs:
                  azureSubscription: 'Azure-Sbx'
                  appName: $(appServiceName)
                  package: '$(Pipeline.Workspace)/drop/*.zip'
```

## Bicep Parameter Files

### sbx.parameters.json

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "environment": { "value": "sbx" },
    "appName": { "value": "myapi" },
    "location": { "value": "eastus" },
    "appServiceSku": { "value": "B1" },
    "sqlDtu": { "value": 10 },
    "enableSwagger": { "value": true },
    "enableDetailedErrors": { "value": true }
  }
}
```

### prod.parameters.json

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "environment": { "value": "prod" },
    "appName": { "value": "myapi" },
    "location": { "value": "eastus" },
    "appServiceSku": { "value": "P1v3" },
    "sqlDtu": { "value": 100 },
    "enableSwagger": { "value": false },
    "enableDetailedErrors": { "value": false }
  }
}
```

## Promotion Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Build     │───►│     Sbx     │───►│     UAT     │
│             │    │ (Auto)      │    │ (Auto)      │
└─────────────┘    └─────────────┘    └──────┬──────┘
                                             │
                                      [Manual Approval]
                                             │
                                      ┌──────▼──────┐
                                      │    Prod     │
                                      │ (Approved)  │
                                      └─────────────┘
```

### Azure DevOps Environment Approvals

Configure in Azure DevOps > Environments > prod > Approvals and checks:

1. Add approvers (team leads, managers)
2. Set timeout (e.g., 24 hours)
3. Optional: Add business hours restriction

## appsettings per Environment

### appsettings.json (base)

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}
```

### appsettings.Development.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  },
  "Swagger": {
    "Enabled": true
  }
}
```

### appsettings.Production.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "Swagger": {
    "Enabled": false
  }
}
```

## Azure App Configuration Labels

Use labels to separate environment configurations:

```
Key                          | Label | Value
-----------------------------|-------|------------------
Database:ConnectionString    | sbx   | Server=sbx-sql...
Database:ConnectionString    | uat   | Server=uat-sql...
Database:ConnectionString    | prod  | Server=prod-sql...
```

Load with label filter:

```csharp
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .Select(KeyFilter.Any, environment); // "sbx", "uat", or "prod"
});
```

## Documentation Template

```markdown
## Environment Matrix

| Setting | Sandbox | UAT | Production |
|---------|---------|-----|------------|
| URL | https://myapi-sbx.azurewebsites.net | https://myapi-uat.azurewebsites.net | https://api.mycompany.com |
| Resource Group | rg-myapi-sbx | rg-myapi-uat | rg-myapi-prod |
| Subscription | Dev/Test | Dev/Test | Production |
| App Service Plan | B1 | B2 | P1v3 |
| Auto-scale | No | No | Yes (2-10) |
| Swagger | Enabled | Enabled | Disabled |
| Logging | Debug | Info | Warning |

### Access

| Environment | Deployment | Logs | Database |
|-------------|------------|------|----------|
| Sandbox | All devs | All devs | All devs |
| UAT | CI/CD only | All devs | Read-only |
| Production | CI/CD only | On-call team | DBA only |
```

## Checklist

- [ ] All environments documented
- [ ] Resource sizing defined per environment
- [ ] Feature flags mapped
- [ ] Parameter files created
- [ ] Promotion workflow configured
- [ ] Approval gates set for production
- [ ] Access controls documented
