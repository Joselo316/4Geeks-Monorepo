# 📋 Agent Development Rules Index

**Central registry of all development rules with explicit scope of application**

**Last Updated:** November 17, 2026  
**Owner:** CTO James Osei  
**Version:** 1.0

---

## 📌 Overview

The `.agents/` folder contains development rules that ALL agents must follow when working on HealthCore Digital. Each rule has an **explicit scope of application** defining:

- **WHERE** it applies (code locations, files, folders)
- **WHO** it applies to (team members, agent types)
- **WHEN** it applies (branches, environments, phases)
- **WHAT** it covers (technologies, data types)

---

## 🗂️ Rules Registry

### **RULE 001: Database Migrations Protocol**
- **File:** [`RULE_001_DATABASE_MIGRATIONS.md`](./RULE_001_DATABASE_MIGRATIONS.md)
- **Status:** ✅ ACTIVE - Mandatory
- **Scope of Application:**
  - **WHERE:** `services/api/migrations/`, `services/api/models/`, `*.sql` files
  - **WHO:** Backend developers, DBAs, all agents modifying schemas
  - **WHEN:** ALL branches (main, develop, feature/*, hotfix/*)
  - **WHAT:** Database schema changes, PHI/patient data, audit logs

| Applies | Does NOT Apply |
|---------|----------------|
| ✅ Schema changes | ❌ Read-only SELECT queries |
| ✅ Migrations | ❌ Index creation on existing columns |
| ✅ Patient data modifications | ❌ Config changes |
| ✅ Compliance auditing | ❌ Documentation-only updates |

- **Key Requirements:** 6 sequential steps (planning → testing → staging → production)
- **Enforcement:** GitHub issue approval required, PR blocked without DBA review
- **Escalation:** Backend lead → CTO for violations

---

## 📊 Scope Matrix

| Rule | Code Locations | Team | Branches | Environments | Technologies | Compliance |
|------|---|---|---|---|---|---|
| **001: DB Migrations** | `services/api/migrations/` | Backend, DBA | ALL | LOCAL→STAGING→PROD | Alembic, Django ORM, SQL | HIPAA, GDPR |
| **002: Monorepo Structure** | `uis/website/`, `uis/backoffice/`, `services/api/`, `packages/shared/` | ALL developers, agents | ALL branches | n/a | Frontend, Backend, Shared | Monorepo organization |

---

## 🔐 Mandatory Rules for All Agents

Before creating or modifying code, verify:

1. ✅ **Scope Check:** Which rule(s) apply to this task?
2. ✅ **Location Check:** Is the code in a protected folder?
3. ✅ **Approval Check:** Do I need external approval before proceeding?
4. ✅ **Compliance Check:** Are HIPAA/GDPR requirements met?
5. ✅ **Testing Check:** Have I tested locally before pushing?

---

## 📖 How to Use These Rules

### For Agents
1. Read the applicable rule file at session startup
2. Follow the **Scope of Application** section first
3. Execute all **Requirements** steps in order
4. Submit PR only after **Definition of Done** is 100% complete
5. If blocked → escalate to contact listed in rule

### For Developers
1. When assigning work to agents, reference the specific rule
2. Example: "Implement the database migration following RULE_001"
3. Use the **Violations & Enforcement** section to review completed work
4. Document exceptions in GitHub issue with @CTO approval

### For Code Review
1. Use the **Definition of Done** checklist
2. Cross-reference PR changes with applicable rules
3. Block merges that violate non-negotiable requirements
4. Document violations in PR review comments

---

## ⚠️ Common Violations by Rule

### RULE 001: Database Migrations
- ❌ Committing migration without downgrade function
- ❌ Skipping GitHub issue [MIGRATION] approval
- ❌ Pushing to production without staging validation
- ❌ Schema changes affecting >3 tables in one migration
- ❌ Missing HIPAA/GDPR compliance comments

**Consequence:** PR rejection + rewrite required

### RULE 002: Monorepo Folder Structure
- ❌ Creating `uis/admin/`, `uis/dashboard/`, `uis/app-v2/` (use uis/backoffice/)
- ❌ Creating `services/appointments-api/`, `services/patient-service/` (use services/api/ with routers)
- ❌ Duplicating shared types: types in `uis/` AND `services/` (centralize in packages/shared/)
- ❌ Creating development rules outside `.agents/RULE_NNN.md` format
- ❌ Maintaining parallel/legacy apps without refactoring plan (>3 months)

**Consequence:** PR REJECTED; rewrite in correct folder or escalate to CTO

---

## 🔗 Integration Points

| System | Integration |
|--------|------------|
| **AGENTS.md** | Master file protection matrix + commit workflow |
| **agents/DEVELOPMENT_RULES.md** | Detailed rules (legacy location, being migrated to .agents/) |
| **docs/COMMIT_WORKFLOW.md** | 5-step workflow all commits must follow |
| **docs/FILE_PROTECTION_MATRIX.md** | Protected files escalation paths |
| **memory-bank/techContext.md** | Technical constraints & compliance requirements |
| **memory-bank/projectbrief.md** | Business context for decision-making |

---

## 📌 Scope Definition Template

Every rule file MUST include this section at the top:

```yaml
Scope of Application:
  
  APPLIES TO:
    Code Locations: [folders, file types]
    Team Members: [roles, agent types]
    Technologies: [languages, frameworks]
    Branches: [main, develop, feature/*, hotfix/*, etc.]
    Repositories: [which repos in monorepo]
    Data Types: [what kind of data]
    Environments: [DEV, STAGING, PROD]

  DOES NOT APPLY TO:
    - [exception 1]
    - [exception 2]
    - [exception 3]
```

---

## 🚀 Adding New Rules

To create a new rule:

1. Create file: `.agents/RULE_NNN_DESCRIPTIVE_NAME.md`
2. Use format from RULE_001 as template
3. Include all 6 sections:
   - Executive Summary
   - Scope of Application
   - Requirements (with steps)
   - Violations & Enforcement
   - Definition of Done
   - Reference Links
4. Add entry to this INDEX file
5. Link in AGENTS.md "File Protection" section
6. Commit with message: `docs: Add RULE_NNN to .agents/`

---

## 📞 Contact & Escalation

| Issue | Contact | Escalation |
|-------|---------|-----------|
| Rule interpretation question | Rule owner (listed in file) | CTO James Osei |
| Rule exception needed | Rule owner | CTO + Backend lead |
| Rule violation in PR | Reviewer → PR author | Rule owner → CTO |
| New rule proposal | Backend team lead | CTO for approval |

---

## ✅ Verification Checklist

- [x] .agents/ folder created
- [x] RULE_001_DATABASE_MIGRATIONS.md documented
- [x] Explicit scope of application defined
- [x] Integration with AGENTS.md confirmed
- [x] INDEX file created with registry
- [x] All 6 sections completed per template

**Status:** ✅ COMPLETE - Ready for agent use

---

**Last Updated:** November 17, 2026, 04:58 UTC  
**Next Review:** February 17, 2027
