---
name: secrets-wiring
description: Configure secret management, Key Vault references, and secure variable bindings. Use when setting up application secrets, pipeline credentials, or secure configuration.
allowed-tools: Read, Edit, Write, Glob, Grep
---

# Secrets Wiring

## Purpose

Documents and configures secure secret management patterns for .NET applications with Azure DevOps.

## When to Use

- Setting up pipeline secrets
- Configuring Key Vault references
- Documenting secret storage locations
- Wiring application configuration to secrets

## Secret Storage Locations

| Environment | Storage | Access Method |
|-------------|---------|---------------|
| Local Dev | User Secrets | `dotnet user-secrets` |
| Pipeline | Azure DevOps Variables | Pipeline variable groups |
| Runtime | Azure Key Vault | Managed Identity |
| App Config | Azure App Configuration | Key Vault references |

## Local Development Secrets

### Initialize User Secrets

```bash
# From project directory
dotnet user-secrets init
```

### Add Secrets

```bash
# Connection string
dotnet user-secrets set "Database:ConnectionString" "<connection-string>"

# API keys
dotnet user-secrets set "ExternalService:ApiKey" "<api-key>"
```

### Access in Code

```csharp
// Program.cs
builder.Configuration.AddUserSecrets<Program>();

// appsettings.json (placeholder)
{
  "Database": {
    "ConnectionString": ""  // Empty - value from user secrets
  }
}
```

## Azure DevOps Pipeline Secrets

### AZURE_DEVOPS_PAT

For NuGet feed authentication:

1. Generate PAT with **only** `Packaging: Read & Write` scope
2. Add to pipeline variables as secret
3. Map in scripts:

```yaml
- script: dotnet restore
  displayName: 'Restore packages'
  env:
    AZURE_DEVOPS_PAT: $(AZURE_DEVOPS_PAT)
```

### Variable Reference Syntax

| Context | Syntax | Example |
|---------|--------|---------|
| YAML | `$(VAR_NAME)` | `$(AZURE_DEVOPS_PAT)` |
| Bash | `$(VAR_NAME)` | `$(AZURE_DEVOPS_PAT)` |
| PowerShell | `${env:VAR_NAME}` | `${env:AZURE_DEVOPS_PAT}` |
| Batch | `%VAR_NAME%` | `%AZURE_DEVOPS_PAT%` |

### Variable Groups

Link to Azure Key Vault for centralized secrets:

```yaml
variables:
  - group: 'KeyVault-Secrets'  # Linked to Key Vault
  - template: variables.yml
```

## Azure Key Vault Integration

### Bicep Configuration

```bicep
resource keyVault 'Microsoft.KeyVault/vaults@2023-02-01' = {
  name: '${resourcePrefix}-kv'
  location: location
  properties: {
    tenantId: subscription().tenantId
    sku: {
      family: 'A'
      name: 'standard'
    }
    enableRbacAuthorization: true
    enableSoftDelete: true
    softDeleteRetentionInDays: 90
  }
}

// Grant App Service access
resource keyVaultRoleAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(keyVault.id, appService.id, 'Key Vault Secrets User')
  scope: keyVault
  properties: {
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', '4633458b-17de-408a-b874-0445c86b69e6') // Key Vault Secrets User
    principalId: appService.identity.principalId
    principalType: 'ServicePrincipal'
  }
}
```

### App Configuration with Key Vault References

```bicep
resource appConfigKeyVaultRef 'Microsoft.AppConfiguration/configurationStores/keyValues@2023-03-01' = {
  parent: appConfiguration
  name: 'Database:ConnectionString'
  properties: {
    value: '{"uri":"${keyVault.properties.vaultUri}secrets/db-connection-string"}'
    contentType: 'application/vnd.microsoft.appconfig.keyvaultref+json;charset=utf-8'
  }
}
```

### Application Configuration

```csharp
// Program.cs
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .ConfigureKeyVault(kv =>
           {
               kv.SetCredential(new DefaultAzureCredential());
           });
});
```

## appsettings.json Pattern

```json
{
  "Database": {
    "ConnectionString": ""
  },
  "ExternalService": {
    "ApiKey": "",
    "BaseUrl": "https://api.example.com"
  },
  "Auth": {
    "Issuer": "",
    "Audience": "",
    "JwksUri": ""
  }
}
```

**Rules:**
- Include key with empty string for secrets
- Non-secret values can have defaults
- Use sensible namespace nesting
- Document required secrets in README

## Secret Rotation

### Key Vault Secret Versioning

```bicep
resource secret 'Microsoft.KeyVault/vaults/secrets@2023-02-01' = {
  parent: keyVault
  name: 'db-connection-string'
  properties: {
    value: connectionString
    attributes: {
      exp: dateTimeToEpoch(dateTimeAdd(utcNow(), 'P90D')) // 90 day expiry
    }
  }
}
```

### Application Handling

```csharp
// Refresh configuration periodically
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.ConfigureRefresh(refresh =>
    {
        refresh.Register("Sentinel", refreshAll: true)
               .SetCacheExpiration(TimeSpan.FromMinutes(5));
    });
});
```

## Security Checklist

- [ ] No secrets in source code
- [ ] No secrets in appsettings.json values
- [ ] User secrets configured for local dev
- [ ] Pipeline secrets marked as secret
- [ ] Key Vault RBAC enabled
- [ ] Managed Identity used for Key Vault access
- [ ] Secret expiration configured
- [ ] Rotation policy documented

## Documentation Template

When setting up secrets, document:

```markdown
## Required Secrets

| Secret | Location | How to Obtain |
|--------|----------|---------------|
| Database:ConnectionString | Key Vault | Azure Portal > SQL Database > Connection Strings |
| Auth:JwksUri | App Config | Identity provider configuration |
| AZURE_DEVOPS_PAT | Pipeline | Azure DevOps > User Settings > PAT |
```
