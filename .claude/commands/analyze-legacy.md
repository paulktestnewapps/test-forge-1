---
allowed-tools: Read, Glob, Grep, Bash
argument-hint: [path-to-legacy-solution]
description: Analyze a legacy C# monolith codebase for modernization planning
---

# Legacy Codebase Analysis

You are analyzing a legacy C# monolith application for modernization. The goal is to understand the codebase structure, identify domains, catalog technical debt, and prepare for migration to a modern architecture.

## Target Architecture (Constant)

The modernized application will consist of:

**Frontend (Separate Codebase)**
- React with Next.js 14+
- Playwright for E2E testing
- Tailwind CSS

**Backend API (Separate Codebase)**
- .NET 8 Web API
- PostgreSQL database
- Azure Blob Storage
- Entity Framework Core 8

## Analysis Path

Analyze the legacy solution at: $ARGUMENTS

## Step 1: Gather Required Information

Before proceeding with deep analysis, ask the user to provide:

### Required Documents
1. **Database Schema** - DDL export or access to run schema queries
2. **Architecture Diagrams** - Any existing documentation
3. **Business Requirements** - Key features and workflows
4. **User Types** - Roles and their primary activities

### Required SQL Query Results

Ask the user to run these queries against their database:

**For SQL Server:**
```sql
-- Table inventory with row counts
SELECT
    s.name AS schema_name,
    t.name AS table_name,
    p.rows AS row_count
FROM sys.tables t
INNER JOIN sys.schemas s ON t.schema_id = s.schema_id
INNER JOIN sys.partitions p ON t.object_id = p.object_id
WHERE p.index_id IN (0, 1)
ORDER BY p.rows DESC;

-- Foreign key relationships
SELECT
    OBJECT_NAME(fk.parent_object_id) AS child_table,
    COL_NAME(fkc.parent_object_id, fkc.parent_column_id) AS child_column,
    OBJECT_NAME(fk.referenced_object_id) AS parent_table,
    COL_NAME(fkc.referenced_object_id, fkc.referenced_column_id) AS parent_column
FROM sys.foreign_keys fk
INNER JOIN sys.foreign_key_columns fkc ON fk.object_id = fkc.constraint_object_id
ORDER BY child_table, parent_table;

-- Stored procedures
SELECT name, create_date, modify_date
FROM sys.procedures
ORDER BY name;

-- Views
SELECT name, create_date
FROM sys.views
ORDER BY name;
```

## Step 2: Solution Structure Analysis

Analyze the solution structure:

1. Find all `.sln` files
2. Parse `.csproj` files for:
   - Target framework version
   - Project references
   - Package references
3. Identify project types:
   - Web (MVC, Web Forms, API)
   - Class Libraries
   - Test Projects
   - Console/Windows Services

## Step 3: Frontend Technology Detection

Look for indicators:

| Pattern | Technology |
|---------|------------|
| `.aspx`, `.aspx.cs` | Web Forms |
| `.cshtml`, `Views/` folder | MVC Razor |
| `.master` files | Master Pages |
| `angular.json` | Angular |
| `package.json` with React | React |

## Step 4: Backend Pattern Detection

Identify data access patterns:
- Entity Framework (`.edmx`, `DbContext`)
- ADO.NET (`SqlConnection`, `SqlCommand`)
- Dapper (`SqlMapper`, `.Query<>`)
- Stored Procedures (`EXEC`, `sp_`)

Identify business logic location:
- Controllers (fat vs thin)
- Service classes
- Code-behind files
- Entity methods

## Step 5: Domain Identification

Group code by business domain:

1. Analyze folder structure for domain hints
2. Analyze namespace patterns
3. Group related tables by foreign keys
4. Identify shared/core entities

## Step 6: Technical Debt Catalog

Document issues:

| Category | What to Find |
|----------|--------------|
| Security | SQL string concatenation, missing auth |
| Performance | N+1 queries, missing indexes |
| Maintainability | God classes, circular dependencies |
| Outdated | Deprecated APIs, old packages |
| Missing | No tests, no logging, no docs |

## Output

Generate a comprehensive analysis report:

```markdown
# Legacy Analysis Report: [Solution Name]

## Executive Summary
- Framework: .NET [version]
- Solution Structure: [X] projects
- Frontend: [Technology]
- Database: [Type], [X] tables
- Lines of Code: ~[estimate]

## Solution Map
[Project dependency diagram]

## Identified Domains
| Domain | Projects | Tables | Key Entities |
|--------|----------|--------|--------------|

## Technology Stack
| Layer | Current | Notes |
|-------|---------|-------|

## Technical Debt
| Issue | Location | Severity | Effort |
|-------|----------|----------|--------|

## Modernization Candidates
| Component | Priority | Rationale |
|-----------|----------|-----------|

## Data Requirements
| Query | Status | Results |
|-------|--------|---------|

## Next Steps
1. Provide missing information: [list]
2. Run database queries: [list pending]
3. Proceed to `/generate-modernization-plan`
```

## Important

- Ask clarifying questions before making assumptions
- Document what information is missing
- Flag security concerns immediately
- Note any blocking dependencies for modernization
