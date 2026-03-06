---
allowed-tools: Read, Edit, Write, Glob, Grep
argument-hint: --db [postgres|sqlserver|mysql|sqlite]
description: Generate SQL migration scripts for the specified database
---

## Required References

Before generating, read these project guides:
- [Database Design Guide](docs/Guides/database_design_guide.md)
- [Primary Key Decision Guide](docs/Guides/pk_type_decision_guide.md)

## Context

Review existing:
- Entity definitions
- DbContext configurations
- Previous migrations

## Task

Generate SQL scripts for database: $ARGUMENTS

1. Create table definitions with proper types for the target database
2. Generate foreign key constraints
3. Add indexes from entity configurations
4. Create migration script with up/down operations

## Required Table Standards

### Primary Keys
- Default to `SERIAL` (Postgres) / `INT IDENTITY` (SQL Server) / `INTEGER AUTOINCREMENT` (SQLite)
- Use `UUID` / `UNIQUEIDENTIFIER` only for API-exposed sensitive data tables

### Audit Columns (required on all tables)
```sql
-- Postgres example
CreatedOn TIMESTAMP NOT NULL DEFAULT (NOW() AT TIME ZONE 'UTC'),
CreatedBy VARCHAR(255) NOT NULL,
UpdatedOn TIMESTAMP NULL,
UpdatedBy VARCHAR(255) NULL
```

### Soft Delete Column
Add for tables with DELETE operations:
```sql
IsActive BOOLEAN NOT NULL DEFAULT TRUE
```

### Naming
- Use casing consistent with existing tables or database provider conventions
- No "Bunzl" prefix unless duplicate name exists

Output files:
- migrations/[timestamp]_[description].sql
- migrations/[timestamp]_[description]_rollback.sql
