---
name: legacy-code-analyzer
description: Analyze legacy C# monolith codebases to identify structure, patterns, domains, and technical debt. Use when assessing existing applications for modernization.
allowed-tools: Read, Glob, Grep
---

# Legacy Code Analyzer

## Purpose

Systematically analyzes legacy .NET monolith applications to understand structure, identify patterns, map domains, and catalog technical debt for modernization planning.

## When to Use

- Assessing legacy applications before modernization
- Understanding unfamiliar codebases
- Identifying extraction candidates for microservices
- Documenting existing system architecture

## Analysis Checklist

### 1. Solution Structure Analysis

```
Patterns to look for:
├── *.sln files (solution entry points)
├── *.csproj files (project structure)
├── web.config / appsettings.json (configuration)
├── packages.config / *.csproj PackageReferences (dependencies)
├── Global.asax / Startup.cs / Program.cs (bootstrapping)
└── Folder patterns indicating architecture
```

### 2. Project Classification

| Project Type | Indicators | Modern Equivalent |
|--------------|------------|-------------------|
| Web Forms | .aspx, .aspx.cs, .ascx | Next.js pages |
| MVC | Controllers/, Views/, Models/ | API + React |
| Web API | ApiController, [Route] | .NET 8 Web API |
| Class Library | No web references | Shared libraries |
| Console App | Main() entry, no web | Background workers |
| Windows Service | ServiceBase | Azure Functions |

### 3. Frontend Technology Detection

**ASP.NET Web Forms**
```
- *.aspx files with runat="server"
- Code-behind (*.aspx.cs)
- UserControls (*.ascx)
- Master pages (*.master)
- ViewState usage
```

**ASP.NET MVC Razor**
```
- *.cshtml files
- @model directives
- @Html helpers
- _Layout.cshtml
- _ViewStart.cshtml
```

**Embedded JavaScript**
```
- <script> blocks in views
- jQuery patterns
- AJAX calls to same-origin
- Knockout.js, Angular 1.x bindings
```

### 4. Backend Pattern Detection

**Data Access Patterns**

| Pattern | Files/Keywords | Modernization |
|---------|----------------|---------------|
| ADO.NET | SqlConnection, SqlCommand | EF Core |
| Entity Framework 6 | DbContext, edmx | EF Core 8 |
| Dapper | SqlMapper, Query<T> | Keep or EF Core |
| Stored Procedures | EXEC, sp_ prefix | Evaluate case-by-case |
| Repository Pattern | IRepository, Repository | Keep pattern |
| Active Record | Entity.Save() | Convert to Repository |

**Business Logic Location**

| Location | Indicator | Extraction |
|----------|-----------|------------|
| Controllers | Fat controllers | Move to services |
| Code-behind | *.aspx.cs logic | Extract to services |
| Stored Procs | Business rules in SQL | Move to application |
| Entity classes | Methods on entities | Domain services |
| Service classes | IService, Service suffix | Keep and modernize |

### 5. Domain Identification

**By Folder Structure**
```csharp
// Look for domain-organized folders
/Features/Orders/
/Features/Customers/
/Features/Inventory/

// Or layer-organized with domain hints
/Models/OrderModels/
/Services/CustomerService.cs
/Repositories/InventoryRepository.cs
```

**By Namespace Analysis**
```csharp
// Extract domain from namespaces
Company.App.Orders.OrderService
Company.App.Customers.CustomerRepository
Company.App.Shared.BaseEntity
```

**By Database Schema**
```sql
-- Group tables by prefix or schema
dbo.Order, dbo.OrderLine, dbo.OrderStatus → Orders domain
dbo.Customer, dbo.CustomerAddress → Customers domain
dbo.Product, dbo.ProductCategory → Catalog domain
```

### 6. Dependency Analysis

**External Integrations**
```csharp
// Look for:
- HttpClient usage (external APIs)
- WCF service references
- SOAP clients
- Message queue connections (RabbitMQ, Azure Service Bus)
- Email services (SMTP, SendGrid)
- Payment gateways
- File storage operations
```

**Inter-Project Dependencies**
```xml
<!-- In .csproj files -->
<ProjectReference Include="..\SharedLib\SharedLib.csproj" />
```

### 7. Technical Debt Catalog

| Category | What to Look For | Severity |
|----------|------------------|----------|
| Outdated Framework | .NET Framework 4.x | High |
| Deprecated APIs | WebClient, ConfigurationManager | Medium |
| Security Issues | SQL concatenation, no HTTPS | Critical |
| Missing Tests | No test projects | High |
| Hard-coded Config | Connection strings in code | Medium |
| God Classes | Classes > 1000 lines | High |
| Circular Dependencies | Projects referencing each other | High |

### 8. Code Metrics Collection

**Commands to Run**
```bash
# Count lines by file type
find . -name "*.cs" | xargs wc -l | sort -n | tail -20

# Find largest files (potential god classes)
find . -name "*.cs" -exec wc -l {} \; | sort -rn | head -20

# Count files by extension
find . -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn
```

**Grep Patterns**
```bash
# Find controllers
grep -r "Controller" --include="*.cs" -l

# Find database access
grep -r "SqlConnection\|DbContext\|IDbConnection" --include="*.cs" -l

# Find external HTTP calls
grep -r "HttpClient\|WebClient\|RestClient" --include="*.cs" -l

# Find configuration access
grep -r "ConfigurationManager\|IConfiguration" --include="*.cs" -l
```

## Output Format

### Analysis Report Structure

```markdown
# Legacy Codebase Analysis Report

## Executive Summary
- Solution: [Name]
- Framework: [.NET version]
- Projects: [count]
- Estimated Lines of Code: [count]
- Primary Architecture: [MVC/Web Forms/Mixed]

## Solution Structure
[Diagram of projects and dependencies]

## Domain Map
| Domain | Projects | Tables | Key Entities |
|--------|----------|--------|--------------|
| Orders | 2 | 5 | Order, OrderLine |
| Customers | 1 | 3 | Customer, Address |

## Technology Stack
- Frontend: [Web Forms/MVC Razor/etc.]
- Backend: [.NET Framework 4.8]
- Database: [SQL Server]
- ORM: [EF6/ADO.NET/Dapper]
- External Services: [list]

## Technical Debt
| Issue | Location | Severity | Effort |
|-------|----------|----------|--------|
| SQL Injection risk | OrderRepository.cs:45 | Critical | Low |
| God class | CustomerService.cs | High | Medium |

## Modernization Candidates
| Component | Current | Proposed | Priority |
|-----------|---------|----------|----------|
| Customer UI | Web Forms | React/Next.js | High |
| Order API | Controller code | REST API | High |

## Risk Assessment
- [List of migration risks]
- [Dependency concerns]
- [Data migration challenges]
```

## SQL Queries for Database Analysis

Provide these to the user to run against legacy database:

### SQL Server Queries

```sql
-- Table inventory
SELECT
    s.name AS schema_name,
    t.name AS table_name,
    p.rows AS row_count
FROM sys.tables t
INNER JOIN sys.schemas s ON t.schema_id = s.schema_id
INNER JOIN sys.partitions p ON t.object_id = p.object_id
WHERE p.index_id IN (0, 1)
ORDER BY s.name, t.name;

-- Foreign key relationships
SELECT
    fk.name AS constraint_name,
    OBJECT_NAME(fk.parent_object_id) AS child_table,
    COL_NAME(fkc.parent_object_id, fkc.parent_column_id) AS child_column,
    OBJECT_NAME(fk.referenced_object_id) AS parent_table,
    COL_NAME(fkc.referenced_object_id, fkc.referenced_column_id) AS parent_column
FROM sys.foreign_keys fk
INNER JOIN sys.foreign_key_columns fkc ON fk.object_id = fkc.constraint_object_id
ORDER BY child_table, parent_table;

-- Stored procedures with line counts
SELECT
    name,
    LEN(OBJECT_DEFINITION(object_id)) - LEN(REPLACE(OBJECT_DEFINITION(object_id), CHAR(10), '')) AS line_count
FROM sys.procedures
ORDER BY line_count DESC;

-- Find tables with no foreign keys (root entities)
SELECT t.name
FROM sys.tables t
WHERE NOT EXISTS (
    SELECT 1 FROM sys.foreign_keys fk
    WHERE fk.parent_object_id = t.object_id
)
ORDER BY t.name;

-- Column details for key tables
SELECT
    c.TABLE_NAME,
    c.COLUMN_NAME,
    c.DATA_TYPE,
    c.CHARACTER_MAXIMUM_LENGTH,
    c.IS_NULLABLE,
    CASE WHEN pk.COLUMN_NAME IS NOT NULL THEN 'PK' ELSE '' END AS is_pk
FROM INFORMATION_SCHEMA.COLUMNS c
LEFT JOIN (
    SELECT ku.TABLE_NAME, ku.COLUMN_NAME
    FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS tc
    JOIN INFORMATION_SCHEMA.KEY_COLUMN_USAGE ku ON tc.CONSTRAINT_NAME = ku.CONSTRAINT_NAME
    WHERE tc.CONSTRAINT_TYPE = 'PRIMARY KEY'
) pk ON c.TABLE_NAME = pk.TABLE_NAME AND c.COLUMN_NAME = pk.COLUMN_NAME
WHERE c.TABLE_NAME IN ('Orders', 'Customers', 'Products') -- Replace with key tables
ORDER BY c.TABLE_NAME, c.ORDINAL_POSITION;
```

## Checklist

- [ ] Solution file(s) located and parsed
- [ ] All projects classified by type
- [ ] Framework version identified
- [ ] Frontend technology detected
- [ ] Data access patterns cataloged
- [ ] Business logic locations mapped
- [ ] Domains identified
- [ ] External integrations listed
- [ ] Technical debt documented
- [ ] Database schema analyzed
