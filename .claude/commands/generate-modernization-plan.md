---
allowed-tools: Read, Edit, Write, Glob, Grep
description: Generate complete modernization plan with PRDs from legacy analysis
---

# Generate Modernization Plan

Generate a complete modernization plan with separate PRDs for frontend and backend applications.

## Prerequisites

Before running this command, ensure you have:

1. **Legacy Analysis Report** - Run `/analyze-legacy` first
2. **Database Schema** - DDL or query results
3. **Business Requirements** - Key features and workflows
4. **SQL Query Results** - Table inventory, relationships, stored procs

## Required Inputs Checklist

Ask the user for any missing items:

- [ ] Path to legacy analysis report (or run `/analyze-legacy` first)
- [ ] Database schema export or query results
- [ ] List of user roles/personas
- [ ] Priority features (must-have vs nice-to-have)
- [ ] Known technical constraints
- [ ] Timeline expectations (if any)
- [ ] Team size/skills

## Target Architecture (Constant)

All plans target this architecture:

### Frontend Application
```
Technology: React 18+ with Next.js 14+ (App Router)
Testing: Playwright (E2E), Jest (unit)
Styling: Tailwind CSS
State: React Query + Zustand
Forms: React Hook Form + Zod
```

### Backend API Application
```
Technology: .NET 8 (ASP.NET Core Web API)
Database: PostgreSQL
Storage: Azure Blob Storage
ORM: Entity Framework Core 8
Auth: JWT with refresh tokens
Docs: OpenAPI/Swagger
```

## Output Structure

Generate the following files:

```
docs/
├── analysis/
│   └── legacy-analysis-report.md    # From /analyze-legacy
├── prd/
│   ├── api-prd.md                   # Backend API specification
│   ├── frontend-prd.md              # Frontend application specification
│   └── shared-requirements.md       # Cross-cutting concerns
└── migration/
    ├── migration-plan.md            # Phased implementation plan
    ├── data-migration.md            # Database migration strategy
    └── risk-assessment.md           # Risk analysis and mitigation
```

## Generation Process

### Step 1: Validate Prerequisites

Check that required information is available:
- Legacy analysis complete
- Domains identified
- Database schema documented
- Business requirements understood

### Step 2: Generate Shared Requirements

Create `docs/prd/shared-requirements.md`:
- Authentication/authorization requirements
- Cross-cutting concerns (logging, monitoring)
- Integration requirements
- Non-functional requirements
- Compliance/regulatory needs

### Step 3: Generate API PRD

Create `docs/prd/api-prd.md` covering:

1. **Domain Model**
   - All entities with properties
   - Relationships (ER diagram)
   - Audit columns (CreatedOn, CreatedBy, UpdatedOn, UpdatedBy)

2. **API Endpoints**
   - Grouped by domain
   - Full REST specification (GET, POST, PUT, PATCH, DELETE)
   - Request/response schemas
   - Error responses
   - Pagination patterns

3. **Authentication**
   - JWT flow
   - Roles and permissions
   - Auth endpoints

4. **Data Migration**
   - Source to target mapping
   - Transformation rules
   - Validation queries

5. **Integrations**
   - External services
   - Azure services (Blob, App Config, Key Vault)

### Step 4: Generate Frontend PRD

Create `docs/prd/frontend-prd.md` covering:

1. **User Personas**
   - Based on legacy user types
   - Goals and workflows

2. **Information Architecture**
   - Site map
   - Navigation structure
   - Page hierarchy

3. **Page Specifications**
   - For each page: route, layout, components, API deps
   - User stories per page

4. **Component Library**
   - Core components needed
   - Form components with validation

5. **State Management**
   - React Query for server state
   - Zustand for client state

6. **Testing Requirements**
   - Playwright E2E test suites
   - Critical user journeys

### Step 5: Generate Migration Plan

Create `docs/migration/migration-plan.md` covering:

1. **Strategy Selection**
   - Recommend Strangler Fig pattern
   - Justify approach

2. **Phase Breakdown**
   - Phase 0: Foundation (infra, project setup)
   - Phase 1-N: Domain-by-domain migration
   - Final Phase: Legacy decommission

3. **Dependency Order**
   - Based on domain relationships
   - Technical dependencies

4. **Risk Assessment**
   - Technical risks
   - Business risks
   - Mitigation strategies

5. **Rollback Plan**
   - Per-phase rollback
   - Decision criteria

### Step 6: Generate Data Migration Plan

Create `docs/migration/data-migration.md` covering:

1. **Schema Mapping**
   - Legacy table → PostgreSQL table
   - Column transformations

2. **Migration Scripts**
   - SQL for initial load
   - Delta sync approach

3. **Validation**
   - Row count comparisons
   - Checksum validations
   - Business rule validations

## SQL Queries to Include

If not already provided, ask user to run:

### Schema Discovery (SQL Server)
```sql
-- Complete table listing with columns
SELECT
    t.TABLE_SCHEMA,
    t.TABLE_NAME,
    c.COLUMN_NAME,
    c.DATA_TYPE,
    c.CHARACTER_MAXIMUM_LENGTH,
    c.IS_NULLABLE,
    c.COLUMN_DEFAULT
FROM INFORMATION_SCHEMA.TABLES t
JOIN INFORMATION_SCHEMA.COLUMNS c
    ON t.TABLE_NAME = c.TABLE_NAME
WHERE t.TABLE_TYPE = 'BASE TABLE'
ORDER BY t.TABLE_SCHEMA, t.TABLE_NAME, c.ORDINAL_POSITION;
```

### Relationship Discovery
```sql
-- Foreign keys with full details
SELECT
    fk.name AS constraint_name,
    OBJECT_SCHEMA_NAME(fk.parent_object_id) AS child_schema,
    OBJECT_NAME(fk.parent_object_id) AS child_table,
    COL_NAME(fkc.parent_object_id, fkc.parent_column_id) AS child_column,
    OBJECT_SCHEMA_NAME(fk.referenced_object_id) AS parent_schema,
    OBJECT_NAME(fk.referenced_object_id) AS parent_table,
    COL_NAME(fkc.referenced_object_id, fkc.referenced_column_id) AS parent_column
FROM sys.foreign_keys fk
JOIN sys.foreign_key_columns fkc ON fk.object_id = fkc.constraint_object_id;
```

### Stored Procedure Analysis
```sql
-- Stored procedures with dependencies
SELECT
    p.name AS procedure_name,
    d.referenced_entity_name AS depends_on
FROM sys.procedures p
LEFT JOIN sys.sql_expression_dependencies d
    ON p.object_id = d.referencing_id
ORDER BY p.name;
```

## Output Summary

After generation, provide summary:

```markdown
## Generated Documents

| Document | Location | Status |
|----------|----------|--------|
| API PRD | docs/prd/api-prd.md | Created |
| Frontend PRD | docs/prd/frontend-prd.md | Created |
| Shared Requirements | docs/prd/shared-requirements.md | Created |
| Migration Plan | docs/migration/migration-plan.md | Created |
| Data Migration | docs/migration/data-migration.md | Created |

## Key Findings

- **Domains**: [X] identified
- **API Endpoints**: [X] planned
- **Frontend Pages**: [X] specified
- **Migration Phases**: [X] phases
- **Estimated Effort**: [X] sprints

## Missing Information

[List any gaps that should be filled]

## Recommended Next Steps

1. Review PRDs with stakeholders
2. Validate data migration mapping
3. Set up new repositories
4. Begin Phase 0 (Foundation)
```

## Important Notes

- Always generate BOTH frontend and backend PRDs (never combined)
- Target stack is constant - do not deviate
- Ask for missing information rather than assuming
- Flag any blocking issues immediately
- Include Mermaid diagrams for visual documentation
