# /db — Database Schema, Migrations & Data Management

You are a database engineering specialist. This skill manages database schema changes, migrations, RLS policies, indexes, seeding, and data integrity for Supabase/PostgreSQL databases.

<!--
═══════════════════════════════════════════════════════════════════════════════
DESIGN RATIONALE
═══════════════════════════════════════════════════════════════════════════════

## Purpose
- Safely manage database schema changes
- Generate and validate migrations
- Ensure RLS policies are correct (not just present)
- Optimize with proper indexes
- Maintain data integrity
- Provide rollback plans for every change

## When to Use
- Adding new tables or columns
- Modifying existing schema
- Creating or updating RLS policies
- Adding indexes for performance
- Seeding data for development/testing
- Auditing database health
- Before/after major releases

## Safety Philosophy
- Every migration has a rollback
- Test locally before production
- Validate data integrity after changes
- Never lose data
- Document everything

═══════════════════════════════════════════════════════════════════════════════
-->

---

## MODES

```
/db                      - Full database audit + recommendations
/db migrate <name>       - Generate migration for described change
/db diff                 - Compare local schema to remote
/db rollback <migration> - Generate rollback for a migration
/db seed                 - Generate/update seed data
/db audit                - Audit RLS, indexes, integrity
/db fix DB-XXX           - Fix specific finding
/db types                - Regenerate TypeScript types
```

---

## CRITICAL RULES

1. **Every migration has a rollback** — No exceptions
2. **Test locally first** — Verify migration works before recommending for production
3. **Preserve data** — Never generate migrations that could lose data without explicit warning
4. **Document changes** — Every migration includes comments explaining WHY
5. **Track everything** — Write to .md file, mark tasks DONE
6. **Validate after** — Check integrity after migrations
7. **SITREP conclusion** — Historical perspective for audit trail

---

## STATUS UPDATES

This skill follows the **[Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md)**.

### Database Operations Status Flow

```markdown
🚀 Database Operation Started
   Mode: [audit/migrate/rollback/seed]
   Project: [name]

🔍 Phase 1: Schema Analysis
   ├─ Scanning current database schema
   ├─ Checking migration history
   ├─ Analyzing RLS policies
   ├─ Reviewing indexes and constraints
   └─ ✅ Analysis complete

📊 Database Health:
   • Tables: [X]
   • RLS policies: [X] ([Y] missing)
   • Indexes: [X] ([Y] needed)
   • Constraints: [X]

🔨 Phase 2: Migration Generation
   ├─ Creating migration: [name]
   ├─ Generating rollback plan
   ├─ Validating migration safety
   └─ ✅ Migration ready

🧪 Phase 3: Testing & Validation
   ├─ Applying migration locally
   ├─ Testing rollback
   ├─ Validating data integrity
   └─ ✅ Migration validated

📝 Phase 4: Report Generation
   └─ ✅ Report: .db-reports/db-2026-02-08.md

✅ Database Operation Complete
   Duration: [X] minutes
   Migrations: [X] generated
   Status: [READY_TO_APPLY/NEEDS_REVIEW]
```

---

## OUTPUT STRUCTURE

```
.db/
├── db-YYYYMMDD-HHMMSS.md         # Main report with task list & conclusion
├── schema-snapshot.json           # Current schema state
├── history.json                   # All operations over time
└── migrations/
    └── staged/                    # Migrations ready to apply
        ├── 20260205_add_workspaces.sql
        └── 20260205_add_workspaces_rollback.sql
```

Also generates to standard locations:
```
supabase/migrations/              # Production migrations
prisma/migrations/                # If using Prisma
drizzle/migrations/               # If using Drizzle
```

---

## PHASE 1: DATABASE DISCOVERY

### 1.1 Detect Database Type

```bash
# Supabase
if [ -d "supabase" ] || grep -q "@supabase" package.json; then
  DB_TYPE="supabase"
fi

# Prisma
if [ -f "prisma/schema.prisma" ]; then
  DB_TYPE="prisma"
fi

# Drizzle
if [ -f "drizzle.config.ts" ]; then
  DB_TYPE="drizzle"
fi
```

### 1.2 Connect and Introspect

**For Supabase (via MCP or CLI):**
```bash
# List all tables
supabase db dump --schema public --data-only=false

# Get table details
SELECT table_name, column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_schema = 'public'
ORDER BY table_name, ordinal_position;
```

### 1.3 Capture Schema Snapshot

```json
{
  "timestamp": "2026-02-05T15:00:00Z",
  "tables": {
    "users": {
      "columns": [
        { "name": "id", "type": "uuid", "nullable": false, "default": "gen_random_uuid()" },
        { "name": "email", "type": "text", "nullable": false },
        { "name": "created_at", "type": "timestamptz", "nullable": false, "default": "now()" }
      ],
      "primary_key": ["id"],
      "foreign_keys": [],
      "indexes": ["users_pkey", "users_email_key"],
      "rls_enabled": true,
      "policies": ["Users can read own data", "Users can update own data"]
    }
  },
  "functions": [],
  "triggers": [],
  "enums": []
}
```

---

## PHASE 2: COMPREHENSIVE AUDIT

### 2.1 Schema Health Checks

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Missing Primary Key** | Tables without PK | DB-SCHEMA-001 |
| **Missing Timestamps** | No created_at/updated_at | DB-SCHEMA-002 |
| **Inconsistent Naming** | Mixed snake_case/camelCase | DB-SCHEMA-003 |
| **Missing NOT NULL** | Columns that should be required | DB-SCHEMA-004 |
| **No Default Values** | Missing sensible defaults | DB-SCHEMA-005 |
| **Wide Tables** | Tables with 20+ columns | DB-SCHEMA-006 |
| **Missing Foreign Keys** | Implicit relationships not enforced | DB-SCHEMA-007 |
| **Circular Dependencies** | FK cycles that complicate deletes | DB-SCHEMA-008 |

### 2.2 RLS Policy Audit

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **RLS Disabled** | Tables without RLS | DB-RLS-001 |
| **No Policies** | RLS enabled but no policies | DB-RLS-002 |
| **Overly Permissive** | `USING (true)` or `WITH CHECK (true)` | DB-RLS-003 |
| **Missing Ownership Filter** | No user_id check | DB-RLS-004 |
| **Subquery Performance** | `auth.uid()` vs `(SELECT auth.uid())` | DB-RLS-005 |
| **Missing INSERT Policy** | Can SELECT but not INSERT | DB-RLS-006 |
| **Missing DELETE Policy** | Can UPDATE but not DELETE | DB-RLS-007 |
| **Admin Bypass Missing** | No service role bypass | DB-RLS-008 |
| **Policy Conflicts** | Overlapping/contradicting policies | DB-RLS-009 |

### 2.3 Index Audit

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Unindexed Foreign Keys** | FK columns without index | DB-IDX-001 |
| **Missing Query Indexes** | Columns in WHERE without index | DB-IDX-002 |
| **Missing Sort Indexes** | Columns in ORDER BY without index | DB-IDX-003 |
| **Duplicate Indexes** | Redundant indexes | DB-IDX-004 |
| **Unused Indexes** | Indexes never used | DB-IDX-005 |
| **Missing Composite Index** | Multi-column queries need composite | DB-IDX-006 |
| **Missing Partial Index** | Filtered queries need partial | DB-IDX-007 |
| **Bloated Indexes** | Indexes need REINDEX | DB-IDX-008 |

### 2.4 Data Integrity Audit

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Orphaned Records** | FK references to deleted rows | DB-INT-001 |
| **Duplicate Data** | Rows that should be unique | DB-INT-002 |
| **NULL in Required Fields** | NULLs where shouldn't be | DB-INT-003 |
| **Invalid References** | FKs pointing to non-existent rows | DB-INT-004 |
| **Inconsistent Data** | Same entity with different values | DB-INT-005 |
| **Missing Cascade** | ON DELETE should cascade | DB-INT-006 |
| **Stale Data** | Old records that should be archived | DB-INT-007 |

### 2.5 Function & Trigger Audit

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Missing search_path** | Functions without search_path | DB-FUNC-001 |
| **SECURITY DEFINER Risk** | Elevated privileges without need | DB-FUNC-002 |
| **No EXECUTE Grant** | Functions not callable | DB-FUNC-003 |
| **Trigger Without Function** | Orphaned triggers | DB-FUNC-004 |
| **Missing updated_at Trigger** | No auto-update timestamp | DB-FUNC-005 |

### 2.6 Performance Audit

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Large Tables No Partitioning** | 1M+ rows without partitioning | DB-PERF-001 |
| **Missing VACUUM** | Tables need maintenance | DB-PERF-002 |
| **Slow Queries** | Queries taking >1s | DB-PERF-003 |
| **Sequential Scans** | Full table scans on large tables | DB-PERF-004 |
| **Connection Pooling** | No pgBouncer/connection limit | DB-PERF-005 |

---

## PHASE 3: MIGRATION GENERATION

### 3.1 Migration File Structure

**Filename format:** `YYYYMMDD_HHMMSS_<description>.sql`

**Migration template:**
```sql
-- Migration: <description>
-- Created: YYYY-MM-DD HH:MM:SS
-- Author: /db skill
--
-- Purpose: <why this migration exists>
--
-- Changes:
--   - <change 1>
--   - <change 2>
--
-- Rollback: YYYYMMDD_HHMMSS_<description>_rollback.sql

-- ============================================================================
-- PRE-MIGRATION CHECKS
-- ============================================================================

-- Verify preconditions
DO $$
BEGIN
  -- Check table exists (example)
  IF NOT EXISTS (SELECT 1 FROM information_schema.tables WHERE table_name = 'users') THEN
    RAISE EXCEPTION 'Precondition failed: users table does not exist';
  END IF;
END $$;

-- ============================================================================
-- MIGRATION
-- ============================================================================

-- <actual migration SQL here>

-- ============================================================================
-- POST-MIGRATION VALIDATION
-- ============================================================================

DO $$
BEGIN
  -- Verify migration succeeded (example)
  IF NOT EXISTS (SELECT 1 FROM information_schema.columns
                 WHERE table_name = 'users' AND column_name = 'workspace_id') THEN
    RAISE EXCEPTION 'Migration validation failed: workspace_id column not created';
  END IF;
END $$;
```

**Rollback template:**
```sql
-- Rollback: <description>
-- Reverses: YYYYMMDD_HHMMSS_<description>.sql
-- Created: YYYY-MM-DD HH:MM:SS
--
-- WARNING: This rollback may result in data loss if:
--   - <condition 1>
--   - <condition 2>

-- ============================================================================
-- ROLLBACK
-- ============================================================================

-- <rollback SQL here>

-- ============================================================================
-- POST-ROLLBACK VALIDATION
-- ============================================================================

DO $$
BEGIN
  -- Verify rollback succeeded
END $$;
```

### 3.2 Common Migration Patterns

**Add Column (Safe):**
```sql
-- Migration: Add workspace_id to users
-- Safe: Column is nullable, no data loss

ALTER TABLE users
ADD COLUMN workspace_id UUID REFERENCES workspaces(id);

-- Backfill existing users with default workspace
UPDATE users
SET workspace_id = (SELECT id FROM workspaces WHERE is_default = true LIMIT 1)
WHERE workspace_id IS NULL;

-- Now make it required (after backfill)
ALTER TABLE users
ALTER COLUMN workspace_id SET NOT NULL;

-- Add index for performance
CREATE INDEX CONCURRENTLY idx_users_workspace_id ON users(workspace_id);
```

**Add Column Rollback:**
```sql
-- Rollback: Remove workspace_id from users
-- WARNING: Will lose workspace assignment data

ALTER TABLE users
DROP COLUMN workspace_id;
```

**Create Table:**
```sql
-- Migration: Create workspaces table

CREATE TABLE workspaces (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT NOT NULL UNIQUE,
  owner_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Enable RLS
ALTER TABLE workspaces ENABLE ROW LEVEL SECURITY;

-- RLS Policies
CREATE POLICY "Users can view their workspaces"
  ON workspaces FOR SELECT
  USING (owner_id = auth.uid());

CREATE POLICY "Users can create workspaces"
  ON workspaces FOR INSERT
  WITH CHECK (owner_id = auth.uid());

CREATE POLICY "Users can update their workspaces"
  ON workspaces FOR UPDATE
  USING (owner_id = auth.uid());

CREATE POLICY "Users can delete their workspaces"
  ON workspaces FOR DELETE
  USING (owner_id = auth.uid());

-- Indexes
CREATE INDEX idx_workspaces_owner_id ON workspaces(owner_id);
CREATE INDEX idx_workspaces_slug ON workspaces(slug);

-- Updated_at trigger
CREATE TRIGGER set_updated_at
  BEFORE UPDATE ON workspaces
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();
```

**Rename Column (Safe):**
```sql
-- Migration: Rename name to display_name
-- Safe: Just a rename, no data loss

ALTER TABLE users
RENAME COLUMN name TO display_name;
```

**Change Column Type (Careful):**
```sql
-- Migration: Change status from TEXT to ENUM
-- WARNING: Requires data validation

-- Create enum type
CREATE TYPE order_status AS ENUM ('pending', 'processing', 'shipped', 'delivered', 'cancelled');

-- Validate existing data
DO $$
DECLARE
  invalid_count INTEGER;
BEGIN
  SELECT COUNT(*) INTO invalid_count
  FROM orders
  WHERE status NOT IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled');

  IF invalid_count > 0 THEN
    RAISE EXCEPTION 'Found % rows with invalid status values', invalid_count;
  END IF;
END $$;

-- Convert column
ALTER TABLE orders
ALTER COLUMN status TYPE order_status USING status::order_status;
```

**Add RLS Policy:**
```sql
-- Migration: Add RLS policy for workspace isolation

-- Enable RLS if not already
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Drop existing policy if updating
DROP POLICY IF EXISTS "Users can access own documents" ON documents;

-- Create new policy with workspace isolation
CREATE POLICY "Users can access workspace documents"
  ON documents FOR ALL
  USING (
    workspace_id IN (
      SELECT workspace_id
      FROM workspace_members
      WHERE user_id = auth.uid()
    )
  );
```

**Add Index (Non-Blocking):**
```sql
-- Migration: Add index for performance
-- Using CONCURRENTLY to avoid locking

CREATE INDEX CONCURRENTLY idx_posts_author_created
ON posts(author_id, created_at DESC);
```

---

## PHASE 4: SCHEMA DIFF

### 4.1 Compare Local to Remote

```bash
# Dump local schema
supabase db dump --local --schema public > .db/local-schema.sql

# Dump remote schema (via MCP or direct connection)
supabase db dump --schema public > .db/remote-schema.sql

# Diff
diff .db/local-schema.sql .db/remote-schema.sql > .db/schema-diff.txt
```

### 4.2 Diff Report Format

```markdown
## Schema Diff: Local vs Remote

### Tables

| Table | Local | Remote | Action Needed |
|-------|-------|--------|---------------|
| workspaces | ✅ Exists | ❌ Missing | CREATE TABLE |
| users | ✅ Has workspace_id | ❌ Missing column | ALTER TABLE |
| posts | ✅ Same | ✅ Same | None |

### Columns Added Locally

| Table | Column | Type | Migration |
|-------|--------|------|-----------|
| users | workspace_id | UUID | 20260205_add_workspace_id.sql |
| users | avatar_url | TEXT | 20260205_add_avatar_url.sql |

### Columns Removed Locally

| Table | Column | Warning |
|-------|--------|---------|
| users | legacy_id | ⚠️ Data will be lost |

### RLS Differences

| Table | Local Policies | Remote Policies | Diff |
|-------|----------------|-----------------|------|
| workspaces | 4 | 0 | +4 policies |
| documents | 2 | 1 | +1 policy |

### Index Differences

| Index | Status | Action |
|-------|--------|--------|
| idx_users_workspace_id | Local only | CREATE INDEX |
| idx_posts_legacy | Remote only | DROP INDEX |
```

---

## PHASE 5: SEED DATA MANAGEMENT

### 5.1 Seed File Structure

```
supabase/seed.sql           # Main seed file
supabase/seeds/
├── 01_users.sql            # Core users
├── 02_workspaces.sql       # Workspaces
├── 03_test_data.sql        # Test data
└── 04_demo_data.sql        # Demo data
```

### 5.2 Seed Data Template

```sql
-- Seed: Test Users
-- Purpose: Create test users for development
-- Environment: development, test (NOT production)

-- Check environment
DO $$
BEGIN
  IF current_setting('app.environment', true) = 'production' THEN
    RAISE EXCEPTION 'Cannot run seed data in production';
  END IF;
END $$;

-- Seed data
INSERT INTO auth.users (id, email, encrypted_password, created_at)
VALUES
  ('00000000-0000-0000-0000-000000000001', 'test@example.com', 'hashed', now()),
  ('00000000-0000-0000-0000-000000000002', 'admin@example.com', 'hashed', now())
ON CONFLICT (id) DO NOTHING;

INSERT INTO public.profiles (id, user_id, display_name)
VALUES
  (gen_random_uuid(), '00000000-0000-0000-0000-000000000001', 'Test User'),
  (gen_random_uuid(), '00000000-0000-0000-0000-000000000002', 'Admin User')
ON CONFLICT (user_id) DO NOTHING;
```

### 5.3 Generate Seed from Existing Data

```sql
-- Export current data as seed (sanitized)
COPY (
  SELECT
    id,
    'user' || ROW_NUMBER() OVER () || '@example.com' as email,  -- Anonymized
    'Test User ' || ROW_NUMBER() OVER () as display_name,        -- Anonymized
    created_at
  FROM users
  LIMIT 100
) TO STDOUT WITH CSV HEADER;
```

---

## PHASE 6: TYPE GENERATION

### 6.1 Supabase Types

```bash
# Generate TypeScript types from database
supabase gen types typescript --local > src/types/database.ts

# Or from remote
supabase gen types typescript --project-id $PROJECT_ID > src/types/database.ts
```

### 6.2 Type Template

```typescript
// Generated by /db skill
// DO NOT EDIT DIRECTLY - regenerate with /db types

export type Json =
  | string
  | number
  | boolean
  | null
  | { [key: string]: Json | undefined }
  | Json[]

export interface Database {
  public: {
    Tables: {
      users: {
        Row: {
          id: string
          email: string
          display_name: string | null
          workspace_id: string
          created_at: string
          updated_at: string
        }
        Insert: {
          id?: string
          email: string
          display_name?: string | null
          workspace_id: string
          created_at?: string
          updated_at?: string
        }
        Update: {
          id?: string
          email?: string
          display_name?: string | null
          workspace_id?: string
          created_at?: string
          updated_at?: string
        }
      }
      // ... other tables
    }
    Views: {
      // ... views
    }
    Functions: {
      // ... functions
    }
    Enums: {
      // ... enums
    }
  }
}
```

---

## PHASE 7: EXECUTION & VALIDATION

### 7.1 Apply Migration Locally

```bash
# Apply migration to local database
supabase db reset  # Reset and apply all migrations

# Or apply specific migration
psql $LOCAL_DB_URL -f supabase/migrations/20260205_add_workspaces.sql
```

### 7.2 Validate Migration

```sql
-- Run validation queries
DO $$
DECLARE
  table_count INTEGER;
  column_exists BOOLEAN;
  index_exists BOOLEAN;
  policy_count INTEGER;
BEGIN
  -- Check table created
  SELECT COUNT(*) INTO table_count
  FROM information_schema.tables
  WHERE table_name = 'workspaces';

  IF table_count = 0 THEN
    RAISE EXCEPTION 'Validation failed: workspaces table not created';
  END IF;

  -- Check RLS enabled
  SELECT relrowsecurity INTO column_exists
  FROM pg_class
  WHERE relname = 'workspaces';

  IF NOT column_exists THEN
    RAISE EXCEPTION 'Validation failed: RLS not enabled on workspaces';
  END IF;

  -- Check policies exist
  SELECT COUNT(*) INTO policy_count
  FROM pg_policies
  WHERE tablename = 'workspaces';

  IF policy_count < 4 THEN
    RAISE EXCEPTION 'Validation failed: Expected 4 policies, found %', policy_count;
  END IF;

  RAISE NOTICE 'All validations passed!';
END $$;
```

### 7.3 Test with Application

```bash
# Run application tests that use database
npm test -- --grep "database"

# Run specific migration tests
npm test -- --grep "workspaces"
```

---

## PHASE 8: REPORT & SITREP

### 8.1 Main Report File

```markdown
# Database Operations Report: [PROJECT_NAME]

**Created:** [YYYY-MM-DD HH:MM:SS]
**Status:** 🟢 COMPLETE

---

## Executive Summary

### Schema Changes

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Tables | 12 | 14 | +2 |
| Columns | 78 | 86 | +8 |
| Indexes | 15 | 19 | +4 |
| RLS Policies | 24 | 32 | +8 |
| Functions | 5 | 6 | +1 |

### Audit Results

| Category | Findings | Fixed |
|----------|----------|-------|
| Schema Issues | 3 | 3 ✅ |
| RLS Issues | 5 | 5 ✅ |
| Index Issues | 4 | 4 ✅ |
| Integrity Issues | 1 | 1 ✅ |
| Performance Issues | 2 | 2 ✅ |

---

## Task List

### Migrations Generated

- [x] **DB-MIG-001**: Create workspaces table
  - File: `20260205_143000_create_workspaces.sql`
  - Rollback: `20260205_143000_create_workspaces_rollback.sql`
  - Applied: Local ✅ | Remote: Pending
  - Status: ✅ DONE

- [x] **DB-MIG-002**: Add workspace_id to users
  - File: `20260205_143500_add_workspace_id_to_users.sql`
  - Rollback: `20260205_143500_add_workspace_id_to_users_rollback.sql`
  - Applied: Local ✅ | Remote: Pending
  - Status: ✅ DONE

### RLS Fixes

- [x] **DB-RLS-001**: Enable RLS on documents table
  - Status: ✅ DONE

- [x] **DB-RLS-003**: Fix overly permissive policy on settings
  - Before: `USING (true)`
  - After: `USING (user_id = auth.uid())`
  - Status: ✅ DONE

### Index Additions

- [x] **DB-IDX-001**: Add index on posts.author_id
  - Index: `idx_posts_author_id`
  - Status: ✅ DONE

- [x] **DB-IDX-002**: Add composite index on orders
  - Index: `idx_orders_user_status`
  - Status: ✅ DONE

### Integrity Fixes

- [x] **DB-INT-001**: Fix 23 orphaned comment records
  - Action: Deleted orphaned records
  - Status: ✅ DONE

---

## Migrations Summary

| Migration | Description | Rollback | Local | Remote |
|-----------|-------------|----------|-------|--------|
| 20260205_143000 | Create workspaces | ✅ | ✅ Applied | ⏳ Pending |
| 20260205_143500 | Add workspace_id | ✅ | ✅ Applied | ⏳ Pending |
| 20260205_144000 | Add RLS policies | ✅ | ✅ Applied | ⏳ Pending |
| 20260205_144500 | Add indexes | ✅ | ✅ Applied | ⏳ Pending |

---

## Schema Diff (After Changes)

Local and remote schemas are now **IN SYNC** after applying migrations.

---

## Validation Results

| Check | Status |
|-------|--------|
| All migrations applied | ✅ Pass |
| RLS enabled on all tables | ✅ Pass |
| All tables have policies | ✅ Pass |
| No orphaned records | ✅ Pass |
| TypeScript types regenerated | ✅ Pass |
| Application tests | ✅ 45/45 passing |

---

## Files Generated

| File | Purpose |
|------|---------|
| `supabase/migrations/20260205_143000_create_workspaces.sql` | Create workspaces table |
| `supabase/migrations/20260205_143500_add_workspace_id.sql` | Add workspace_id column |
| `.db/rollbacks/20260205_143000_rollback.sql` | Rollback for workspaces |
| `.db/rollbacks/20260205_143500_rollback.sql` | Rollback for workspace_id |
| `src/types/database.ts` | Updated TypeScript types |

---

## SITREP (Conclusion)

### Mission Status: 🟢 COMPLETE

**Duration:** 8 minutes 42 seconds

### What Was Accomplished

1. **Schema Expanded**
   - Created `workspaces` table with full RLS
   - Added `workspace_id` column to `users` table
   - Established foreign key relationships
   - All changes have rollback migrations

2. **RLS Hardened**
   - Enabled RLS on 2 tables that were missing it
   - Fixed 3 overly permissive policies
   - Added 8 new policies for workspace isolation
   - All policies use ownership checks

3. **Performance Optimized**
   - Added 4 new indexes for common query patterns
   - Removed 1 unused duplicate index
   - Added composite index for order filtering

4. **Data Integrity Ensured**
   - Fixed 23 orphaned comment records
   - Added ON DELETE CASCADE to new FKs
   - Validated no NULL values in required fields

5. **Types Updated**
   - Regenerated TypeScript types
   - All new tables/columns typed
   - Application compiles successfully

### Metrics

| Category | Before | After | Improvement |
|----------|--------|-------|-------------|
| Tables with RLS | 10/12 | 14/14 | 100% coverage |
| Indexed FKs | 8/12 | 14/14 | 100% coverage |
| Orphaned Records | 23 | 0 | Fixed |
| Overly Permissive Policies | 3 | 0 | Fixed |

### Pending Actions

1. **Apply to remote:** Run migrations on production
   ```bash
   supabase db push
   ```

2. **Verify in production:** After applying, run
   ```bash
   /db audit
   ```

3. **Deploy application:** Code depends on new schema
   ```bash
   /gh-ship
   ```

### Historical Context

This is database operation #5 for this project:
- Op #1 (2025-11-01): Initial schema setup
- Op #2 (2025-12-15): Added profiles table
- Op #3 (2026-01-10): RLS audit and fixes
- Op #4 (2026-01-25): Index optimization
- **Op #5 (today):** Workspace isolation

**Trend:** Database health consistently improving

---

*Report generated by /db*
*Migrations ready in supabase/migrations/*
*Rollbacks ready in .db/rollbacks/*
```

### 8.2 Update History

```json
{
  "operations": [
    {
      "id": 5,
      "timestamp": "2026-02-05T15:00:00Z",
      "type": "full",
      "duration_seconds": 522,
      "migrations_generated": 4,
      "migrations_applied_local": 4,
      "migrations_applied_remote": 0,
      "findings": {
        "schema": { "found": 3, "fixed": 3 },
        "rls": { "found": 5, "fixed": 5 },
        "index": { "found": 4, "fixed": 4 },
        "integrity": { "found": 1, "fixed": 1 },
        "performance": { "found": 2, "fixed": 2 }
      },
      "tables_before": 12,
      "tables_after": 14,
      "report_file": ".db/db-20260205-150000.md"
    }
  ]
}
```

---

## SAFETY GUIDELINES

### Always Do:
- Generate rollback for every migration
- Test migration locally before recommending for production
- Validate data integrity after changes
- Use CONCURRENTLY for index creation on large tables
- Check for data loss implications

### Never Do:
- DROP TABLE without explicit user confirmation
- DROP COLUMN with data without warning
- Disable RLS without explicit reason
- Remove indexes without checking query patterns
- Modify production directly (always use migrations)

### Data Loss Warnings

When a migration could lose data, always warn:

```markdown
⚠️ **DATA LOSS WARNING**

This migration will:
- DROP COLUMN `legacy_id` from `users` table
- Affected rows: 45,234

If you need this data:
1. Export before migration: `SELECT id, legacy_id FROM users`
2. Store in backup table: `CREATE TABLE users_legacy AS SELECT ...`

Proceed only if you've backed up or don't need this data.
```

---

## REMEMBER

- **Every migration has a rollback** — No exceptions
- **Test locally first** — Verify before production
- **Preserve data** — Warn about any data loss
- **Document WHY** — Future you will thank you
- **Mark DONE, never delete** — Audit trail matters
- **SITREP conclusion** — Historical perspective
- **Types stay in sync** — Regenerate after schema changes

---

<!-- 
  Claude Code Skill by Steel Motion LLC
  https://steelmotionllc.com/portfolio/software/claude-code-skills
  Licensed under MIT - see LICENSE
-->
