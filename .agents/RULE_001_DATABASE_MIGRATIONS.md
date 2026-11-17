# 🔄 RULE 001: Database Migrations Protocol

**Version:** 1.0  
**Date:** November 17, 2026  
**Owner:** Backend Team Lead  
**Status:** ACTIVE - Mandatory Compliance Required

---

## 📌 Executive Summary

All database schema changes in HealthCore Digital must follow strict versioning and rollback protocols. This rule prevents data loss, ensures compliance auditing, and maintains system stability.

---

## 🎯 Scope of Application

### **APPLIES TO:**

| Category | Details |
|----------|---------|
| **Code Locations** | `services/api/migrations/`, `services/api/models/`, any `*.sql` files |
| **Team Members** | Backend developers, DBAs, all agents modifying database schemas |
| **Technologies** | Alembic (SQLAlchemy), Django ORM, raw SQL scripts |
| **Branches** | ALL branches (main, develop, feature/*, hotfix/*) |
| **Repositories** | HealthCore Digital monorepo |
| **Data Types** | Patient records (PHI), appointments, clinic data, audit logs |
| **Environments** | LOCAL DEV → STAGING → PRODUCTION (must pass each level) |

### **DOES NOT APPLY TO:**

- ❌ Read-only SELECT queries (don't require migrations)
- ❌ Index creation on existing columns (create separately, documented)
- ❌ Configuration changes (use feature flags)
- ❌ Documentation-only updates

---

## 📋 Requirements

### **STEP 1: Pre-Migration Planning (Before Writing Code)**

```yaml
Requirement: Every migration must have approval before implementation

Checklist:
  ✓ Create GitHub issue with tag: [MIGRATION] in title
  ✓ Document:
    - What schema is changing (table, column, constraint)
    - Why (feature requirement, bug fix, compliance)
    - Rollback strategy (how to undo if something breaks)
    - Estimated downtime (must be ≤30 seconds)
    - Data validation plan
  ✓ Get DBAs approval (comment in GitHub issue: "@dba-lead review needed")
  ✓ Wait for approval before proceeding to STEP 2

Violation: Skipping GitHub issue = automatic rejection on PR review
```

**Example:** `issues/#234 - [MIGRATION] Add HIPAA audit_log table`

---

### **STEP 2: Migration Code Generation (Using Framework Tools)**

```bash
# Using Alembic (SQLAlchemy)
cd services/api
alembic revision --autogenerate -m "Add audit_log table with HIPAA compliance"

# Using Django migrations
cd services/api
python manage.py makemigrations --name add_audit_log_table

# Manual migrations MUST include:
# - Version number (alembic: auto-generated, manual: YYYYMMDD_HHmm_descriptive_name)
# - Upgrade function (apply changes)
# - Downgrade function (rollback changes)
# - Detailed comments explaining every step
```

**Non-Negotiable Rules:**
- ❌ NO inline SQL without comments
- ❌ NO schema changes without down() function (rollback path)
- ❌ NO migrations that touch >3 tables (break into smaller migrations)
- ✅ ALWAYS include data validation in migration

**Example Migration File:**
```python
# File: alembic/versions/20261117_1234_add_audit_log_hipaa.py

def upgrade() -> None:
    # HIPAA COMPLIANCE: Create audit_log table
    # This table tracks all PHI access and modifications per HIPAA §164.312(b)
    op.create_table(
        'audit_log',
        sa.Column('id', sa.UUID(), primary_key=True),
        sa.Column('patient_id', sa.UUID(), sa.ForeignKey('patients.id'), nullable=False),
        sa.Column('action', sa.String(50), nullable=False),  # 'read', 'update', 'delete'
        sa.Column('performed_by', sa.UUID(), sa.ForeignKey('users.id'), nullable=False),
        sa.Column('timestamp', sa.DateTime(timezone=True), server_default=sa.func.now()),
        sa.Column('ip_address', sa.String(45), nullable=True),  # IPv4 or IPv6
        # Encryption: ip_address is encrypted at application layer
    )
    # Create index for performance on queries
    op.create_index('idx_audit_log_patient_timestamp', 'audit_log', 
                    ['patient_id', 'timestamp'], unique=False)

def downgrade() -> None:
    # Rollback: Remove audit_log table
    # Data loss warning: This will delete all audit records!
    op.drop_index('idx_audit_log_patient_timestamp')
    op.drop_table('audit_log')
```

---

### **STEP 3: Testing in Local Development**

```bash
# Local testing BEFORE pushing to GitHub

# 1. Apply migration
alembic upgrade head  # or django migrate

# 2. Verify schema matches expectation
psql healthcore_dev -c "\d audit_log"  # PostgreSQL descriptor

# 3. Test rollback works
alembic downgrade -1
alembic upgrade head  # re-apply to verify idempotency

# 4. Data validation test
python scripts/validate_migration.py --migration add_audit_log_hipaa

# 5. Performance check (if adding index)
psql healthcore_dev -c "ANALYZE audit_log; EXPLAIN ANALYZE SELECT * FROM audit_log WHERE patient_id = 'xyz';"

# Non-Negotiable:
# ❌ NEVER commit without testing downgrade
# ❌ NEVER assume migration will work in production without testing up→down→up cycle
```

---

### **STEP 4: Code Review & PR Submission**

```yaml
PR Checklist (before submitting):

Code Quality:
  ✓ Migration file has clear comments explaining EVERY change
  ✓ Alembic/Django auto-generated code is reviewed for correctness
  ✓ No hardcoded values or magic numbers
  ✓ Type hints used (if applicable)

Compliance:
  ✓ PHI data is encrypted (comment: "# HIPAA §164.312(e)(2)(ii) encryption")
  ✓ Audit trail included for sensitive operations
  ✓ GDPR compliance reviewed if EU data involved
  ✓ Linked to GitHub issue with [MIGRATION] tag

Performance:
  ✓ Schema change doesn't require long locks (downtime ≤30 seconds)
  ✓ Indexes created for new foreign keys
  ✓ No N+1 query vulnerabilities introduced

Rollback:
  ✓ downgrade() function tested and works
  ✓ Rollback procedure documented in PR description
  ✓ Estimated rollback time calculated (<5 minutes)

PR Description Template:
---
## [MIGRATION] Add audit_log table with HIPAA compliance

### What's Changing
- Adds `audit_log` table to track PHI access

### Why
- Requirement: HIPAA §164.312(b) audit controls
- Linked issue: #234

### Rollback Plan
- Run: `alembic downgrade -1`
- Verify: `SELECT COUNT(*) FROM audit_log` returns 0
- Estimated time: 2 minutes

### Testing Evidence
- Local up/down/up cycle: ✅ PASSED
- Performance test: ✅ Query <100ms with 100k rows
- Data validation: ✅ No orphaned records
---
```

---

### **STEP 5: Staging Environment Validation (Pre-Production)**

```bash
# After PR is merged to develop/staging branch

# 1. Deploy migration to staging
kubectl rollout status deployment/api-service -n staging
# Verify: Check migration logs

# 2. Smoke test
curl -s http://api-staging.healthcore.local/health | jq .
# Expected: {"status": "healthy", "db_migration": "current"}

# 3. Query validation
psql healthcore_staging -U staging_user -c "SELECT COUNT(*) FROM audit_log;"
# Expected: Results match production schema

# 4. Monitoring setup
# Create alert in Datadog/New Relic for audit_log table growth
# Alert threshold: Warn if >10,000 rows/hour (data anomaly detection)

# Non-Negotiable:
# ❌ NEVER deploy to production without staging validation
# ❌ NEVER skip monitoring setup
```

---

### **STEP 6: Production Deployment**

```bash
# Deployment window: Maintenance window only (Friday 2-4 AM UTC)
# Requires: 2 approvals (backend lead + DBA)

# Commands for production:
kubectl set image deployment/api-service api-service=healthcore/api:v1.2.3-migrations \
  -n production \
  --record

# Monitor migration progress
kubectl logs -n production -l app=api-service -f | grep -i "migration"

# Verify production data integrity
psql healthcore_prod -U prod_user -c "SELECT version() FROM schema_version ORDER BY version DESC LIMIT 1;"

# Alert DevOps if anything fails
# Slack: #devops-alerts
```

---

## 🚫 Violations & Enforcement

| Violation | Consequence | Escalation |
|-----------|------------|-----------|
| Skipping GitHub issue approval | PR blocked, manual rejection | PR author must add issue link |
| Migration without downgrade function | PR rejected, requires rewrite | Backend team lead + CTO review |
| Data loss in migration (>1% data affected) | IMMEDIATE ROLLOUT HALT | CTO + DBA + Security Team |
| Downtime >30 seconds | Rejected for production | Rewrite migration strategy |
| Missing HIPAA/GDPR comments | PR blocked | Compliance review required |
| Deploying without staging validation | Automatic rollback + incident post-mortem | DevOps + Backend lead + CTO |

---

## ✅ Definition of Done (DoD)

A migration is considered **COMPLETE** when:

```
Local Development ✅
  ✓ Migration file created with upgrade() + downgrade()
  ✓ Local up/down/up cycle succeeds
  ✓ Schema validated with database descriptor
  ✓ Performance baseline established

Code Review ✅
  ✓ GitHub issue [MIGRATION] approved by DBA
  ✓ PR submitted with complete description
  ✓ Code review: 1 approval (backend lead)
  ✓ All CI checks pass (lint, type, security)

Staging Validation ✅
  ✓ Merged to develop branch
  ✓ Deployed to staging successfully
  ✓ Smoke test passes
  ✓ Query validation matches production schema
  ✓ Monitoring alerts configured

Production Ready ✅
  ✓ 2 approvals (backend lead + DBA)
  ✓ Scheduled for maintenance window
  ✓ Rollback plan documented and tested
  ✓ Post-deployment verification completed
```

---

## 📚 Reference Links

- **Alembic Documentation:** https://alembic.sqlalchemy.org/
- **Django Migrations:** https://docs.djangoproject.com/en/5.0/topics/migrations/
- **HIPAA Compliance:** memory-bank/techContext.md (Section: Compliance & Regulations)
- **File Protection Matrix:** AGENTS.md (Section: File Protection)
- **Commit Workflow:** AGENTS.md (Section: Obligatory Commit Workflow)

---

## 📞 Questions or Exceptions?

Contact: **Backend Team Lead** (@backend-lead on GitHub)  
Escalation: **CTO James Osei** (@jamesoosei)

**Last Updated:** November 17, 2026  
**Review Cycle:** Every 3 months or after major release
