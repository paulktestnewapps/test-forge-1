---
name: iac-module-generator
description: Generate Infrastructure as Code modules using Bicep or Terraform. Use when creating cloud resource definitions, parameterized templates, or deployment configurations.
allowed-tools: Read, Edit, Write, Glob, Grep
---

# IaC Module Generator

## Purpose

Generates modular, reusable Infrastructure as Code templates for Azure deployments.

## When to Use

- Creating new Azure resources
- Parameterizing existing infrastructure
- Setting up environment-specific configurations
- Standardizing IaC across projects

## Supported Tools

- Bicep (primary for Azure)
- Terraform

## Standard File Structure

```
infra/
  main.bicep                    # Main deployment orchestration
  modules/
    app-service.bicep           # App Service module
    sql-database.bicep          # SQL Database module
    app-configuration.bicep     # App Configuration module
    container-app.bicep         # Container App module
    key-vault.bicep             # Key Vault module
  parameters/
    sbx.parameters.json         # Sandbox parameters
    uat.parameters.json         # UAT parameters
    prod.parameters.json        # Production parameters
```

## Bicep Templates

### Main Deployment (main.bicep)

```bicep
targetScope = 'resourceGroup'

@description('Environment name')
@allowed(['sbx', 'uat', 'prod'])
param environment string

@description('Location for resources')
param location string = resourceGroup().location

@description('Application name')
param appName string

// Naming convention
var resourcePrefix = '${appName}-${environment}'

// App Service
module appService 'modules/app-service.bicep' = {
  name: 'appService-${uniqueString(resourceGroup().id)}'
  params: {
    name: '${resourcePrefix}-app'
    location: location
    environment: environment
  }
}

// SQL Database
module sqlDatabase 'modules/sql-database.bicep' = {
  name: 'sqlDatabase-${uniqueString(resourceGroup().id)}'
  params: {
    name: '${resourcePrefix}-sql'
    location: location
    environment: environment
  }
}

// Outputs
output appServiceUrl string = appService.outputs.url
output sqlServerFqdn string = sqlDatabase.outputs.serverFqdn
```

### App Service Module (modules/app-service.bicep)

```bicep
@description('App Service name')
param name string

@description('Location')
param location string

@description('Environment')
@allowed(['sbx', 'uat', 'prod'])
param environment string

// SKU based on environment
var skuName = environment == 'prod' ? 'P1v3' : 'B1'

resource appServicePlan 'Microsoft.Web/serverfarms@2022-09-01' = {
  name: '${name}-plan'
  location: location
  sku: {
    name: skuName
  }
  properties: {
    reserved: true // Linux
  }
  tags: {
    environment: environment
  }
}

resource appService 'Microsoft.Web/sites@2022-09-01' = {
  name: name
  location: location
  properties: {
    serverFarmId: appServicePlan.id
    siteConfig: {
      linuxFxVersion: 'DOTNETCORE|8.0'
      alwaysOn: environment == 'prod'
    }
    httpsOnly: true
  }
  identity: {
    type: 'SystemAssigned'
  }
  tags: {
    environment: environment
  }
}

output url string = 'https://${appService.properties.defaultHostName}'
output principalId string = appService.identity.principalId
```

### Container App Module (modules/container-app.bicep)

```bicep
@description('Container App name')
param name string

@description('Location')
param location string

@description('Environment')
@allowed(['sbx', 'uat', 'prod'])
param environment string

@description('Container image')
param containerImage string

@description('Container App Environment ID')
param containerAppEnvironmentId string

// CPU and memory based on environment
var cpu = environment == 'prod' ? '1.0' : '0.5'
var memory = environment == 'prod' ? '2Gi' : '1Gi'

resource containerApp 'Microsoft.App/containerApps@2023-05-01' = {
  name: name
  location: location
  properties: {
    managedEnvironmentId: containerAppEnvironmentId
    configuration: {
      ingress: {
        external: true
        targetPort: 8080
        transport: 'http'
      }
    }
    template: {
      containers: [
        {
          name: name
          image: containerImage
          resources: {
            cpu: json(cpu)
            memory: memory
          }
        }
      ]
      scale: {
        minReplicas: environment == 'prod' ? 2 : 1
        maxReplicas: environment == 'prod' ? 10 : 3
      }
    }
  }
  tags: {
    environment: environment
  }
}

output fqdn string = containerApp.properties.configuration.ingress.fqdn
```

### Parameters File (parameters/sbx.parameters.json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "environment": {
      "value": "sbx"
    },
    "appName": {
      "value": "myapi"
    },
    "location": {
      "value": "eastus"
    }
  }
}
```

## Terraform Templates

### Main Configuration (main.tf)

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

variable "environment" {
  type        = string
  description = "Environment name"
  validation {
    condition     = contains(["sbx", "uat", "prod"], var.environment)
    error_message = "Environment must be sbx, uat, or prod."
  }
}

variable "app_name" {
  type        = string
  description = "Application name"
}

variable "location" {
  type        = string
  default     = "eastus"
  description = "Azure region"
}

locals {
  resource_prefix = "${var.app_name}-${var.environment}"
}

module "app_service" {
  source      = "./modules/app-service"
  name        = "${local.resource_prefix}-app"
  location    = var.location
  environment = var.environment
}

output "app_service_url" {
  value = module.app_service.url
}
```

## Best Practices

1. **Modular Design**: One resource type per module
2. **Parameterization**: Environment-specific values in parameter files
3. **Naming Conventions**: Use consistent `{app}-{env}-{resource}` pattern
4. **Tagging**: Always include environment tag
5. **Outputs**: Export values needed by other modules or pipelines
6. **Managed Identity**: Use SystemAssigned identity for Azure services

## Checklist

- [ ] Modules are reusable and parameterized
- [ ] Environment-specific parameter files exist
- [ ] Naming convention followed
- [ ] Tags applied to all resources
- [ ] Outputs defined for integration
- [ ] Managed identities used
- [ ] SKUs appropriate for environment
