# 🗂️ RULE 002: Monorepo Folder Structure & Naming Convention

**Version:** 1.0  
**Date:** November 17, 2026  
**Owner:** CTO James Osei  
**Status:** ACTIVE - Mandatory Compliance Required

---

## 📌 Executive Summary

HealthCore Digital uses a **single monorepo with strict folder organization** to prevent duplicate codebases, reduce complexity, and ensure single source of truth. This rule defines WHERE each type of code belongs and explicitly PROHIBITS parallel/duplicate applications.

**Core Principle:** One app per category (one website, one backoffice, one API service) — no parallel versions.

---

## 🎯 Scope of Application

### **APPLIES TO:**

| Category | Details |
|----------|---------|
| **Code Locations** | ALL folders in monorepo root; ALL new features/services/UIs |
| **Team Members** | All developers, all agents proposing new code/features |
| **Technologies** | React (frontend), FastAPI (backend), Python (agents), TypeScript (shared) |
| **Branches** | ALL branches (main, develop, feature/*, hotfix/*) |
| **Repositories** | HealthCore Digital monorepo only |
| **Decision Points** | BEFORE creating ANY new folder/app; BEFORE cloning existing code |
| **Environments** | LOCAL DEV → STAGING → PRODUCTION |

### **DOES NOT APPLY TO:**

- ❌ `/data/` folder (datasets, not apps)
- ❌ `/docs/` folder (documentation)
- ❌ `/infra/` folder (deployment configs)
- ❌ `/scripts/` folder (automation scripts)
- ❌ `/internal/` folder (internal utilities)
- ❌ Third-party dependencies (packages/node_modules, venv, etc.)

---

## 📋 Monorepo Structure (Authoritative)

```
4Geeks-Monorepo/                    # Single monorepo for entire company
│
├── 🎨 uis/                         # FRONTEND APPLICATIONS ONLY
│   ├── website/                    # 🌐 ONE public website (React/TypeScript)
│   │   ├── src/
│   │   ├── public/
│   │   ├── package.json
│   │   └── README.md
│   │
│   └── backoffice/                 # 🛠️ ONE internal admin app (React/TypeScript)
│       ├── src/
│       ├── public/
│       ├── package.json
│       └── README.md
│       
│       ⚠️ NO: uis/admin/, uis/dashboard/, uis/app-v2/, uis/legacy-ui/ (FORBIDDEN)
│
├── 🔧 services/                    # BACKEND SERVICES ONLY
│   ├── api/                        # 🚀 ONE centralized FastAPI application
│   │   ├── routers/                # Domain routers (appointments, patients, etc.)
│   │   ├── models/                 # SQLAlchemy ORM models
│   │   ├── schemas/                # Pydantic request/response schemas
│   │   ├── db/
│   │   ├── security/
│   │   ├── integrations/
│   │   ├── main.py                 # FastAPI app entry point
│   │   ├── requirements.txt
│   │   └── README.md
│   │
│   ⚠️ NO: services/appointments-api/, services/patient-api/, services/api-v2/ (FORBIDDEN)
│
├── 🤖 agents/                      # AI AGENTS & AUTOMATION
│   ├── skills/                     # Reusable agent capabilities
│   │   ├── APPOINTMENT_REMINDER_PROCESSOR.md
│   │   ├── [NEW_SKILLS_HERE]
│   │   └── README.md
│   │
│   ├── tools/                      # Agent tools (MCP servers)
│   ├── DEVELOPMENT_RULES.md        # Legacy: being migrated to .agents/
│   └── README.md
│
├── 📦 packages/                    # SHARED CODE (TypeScript/Python)
│   ├── shared/                     # Shared TypeScript types (single source of truth)
│   │   ├── src/types/
│   │   ├── src/schemas/
│   │   └── package.json
│   │
│   ⚠️ NO: packages/frontend-utils/, packages/backend-utils/ (use shared/)
│
├── 🔗 .agents/                     # AGENT DEVELOPMENT RULES
│   ├── README.md                   # Rules registry
│   ├── RULE_001_DATABASE_MIGRATIONS.md
│   ├── RULE_002_MONOREPO_FOLDER_STRUCTURE.md
│   └── [NEW_RULES_HERE]
│
├── 📋 memory-bank/                 # AGENT MEMORY & CONTEXT
│   ├── projectbrief.md             # Business context (all agents read)
│   ├── techContext.md              # Technical context (all agents read)
│   ├── progress.md                 # Progress tracking
│   └── README.md
│
├── 📂 workflows/                   # N8N ORCHESTRATION
│   ├── appointment-reminders.json
│   ├── data-sync-workflow.json
│   └── README.md
│
├── 🗄️ data/                        # DATASETS & PIPELINES (not apps)
│   ├── migrations/
│   ├── seeds/
│   ├── exports/
│   └── README.md
│
├── 🐳 infra/                       # INFRASTRUCTURE & DEPLOYMENT
│   ├── docker/
│   ├── kubernetes/
│   ├── terraform/
│   └── README.md
│
├── 📚 docs/                        # DOCUMENTATION
│   ├── ARCHITECTURE.md
│   ├── COMMIT_WORKFLOW.md
│   ├── FILE_PROTECTION_MATRIX.md
│   └── README.md
│
├── 🛠️ internal/                    # INTERNAL UTILITIES
│   ├── scripts/
│   ├── dev-tools/
│   └── README.md
│
├── 🎬 mcps/                        # MCP TOOL SERVERS
│   ├── github-tools/
│   ├── calendar-integration/
│   └── README.md
│
└── 📄 [Root Files]
    ├── AGENTS.md                   # Central agent initialization
    ├── Briefing.md                 # Business briefing
    ├── CONTRIBUTING.md
    ├── package.json                # Monorepo root (npm workspaces)
    └── README.md
```

---

## ✅ WHERE EACH CODE BELONGS

### **FRONTEND: React/TypeScript Applications → `/uis/`**

**Rule:** Exactly 2 frontend applications, no more, no duplicates.

| Application | Folder | Purpose | Tech Stack | Audience |
|-------------|--------|---------|-----------|----------|
| **Website** | `uis/website/` | Public-facing company website, patient portal | React 18+, TypeScript, Tailwind, Vite | Patients, clinic staff |
| **Backoffice** | `uis/backoffice/` | Internal admin dashboard, clinic operations, reporting | React 18+, TypeScript, Tailwind, Vite | Admin staff, clinicians, managers |

**Examples of CORRECT organization:**
- ✅ `uis/website/pages/appointment-booking.tsx` (feature in website)
- ✅ `uis/backoffice/pages/patient-management.tsx` (feature in backoffice)

**Examples of FORBIDDEN structure (VIOLATES RULE):**
- ❌ `uis/admin/` (create in backoffice, not separate app)
- ❌ `uis/dashboard/` (feature of backoffice, not separate app)
- ❌ `uis/patient-portal/` (feature of website, not separate app)
- ❌ `uis/app-v2/` (evolve website/backoffice, don't duplicate)
- ❌ `uis/legacy-ui/` (refactor, don't parallel-maintain)

---

### **BACKEND: FastAPI Service → `/services/api/`**

**Rule:** Exactly 1 centralized FastAPI application with routers per domain.

**Decision Rationale:**
- Microservices add operational complexity (deployment, monitoring, debugging)
- One app with routers keeps team velocity high during early growth
- Extract microservices only when SPECIFIC need emerges (high load, independent scaling)
- See decision in `memory-bank/techContext.md` Section: "Centralized FastAPI Backend"

| Component | Folder | Purpose |
|-----------|--------|---------|
| **API Service** | `services/api/` | FastAPI application with ALL endpoints |
| **Routers** | `services/api/routers/` | Domain-specific routers (appointments, patients, clinics, etc.) |
| **Database Models** | `services/api/models/` | SQLAlchemy ORM models |
| **Schemas** | `services/api/schemas/` | Pydantic request/response schemas |
| **Database** | `services/api/db/` | Alembic migrations, session management |
| **Integrations** | `services/api/integrations/` | Third-party API clients (EHR System A, System B, SMS, Email) |

**Examples of CORRECT organization:**
- ✅ `services/api/routers/appointments.py` (appointments router)
- ✅ `services/api/routers/patients.py` (patients router)
- ✅ `services/api/integrations/ehr_system_a.py` (EHR integration)
- ✅ `services/api/models/appointment.py` (Appointment ORM model)

**Examples of FORBIDDEN structure (VIOLATES RULE):**
- ❌ `services/appointments-api/` (routers go in services/api/routers/)
- ❌ `services/patient-service/` (routers go in services/api/routers/)
- ❌ `services/api-v2/` (evolve services/api/, don't duplicate)
- ❌ `services/legacy-api/` (refactor, don't parallel-maintain)
- ❌ `services/workers/` for Celery tasks (can live in services/api/workers/ if needed)

**Microservices Extraction Rule:**
- Keep everything in `services/api/` by default
- Extract to `services/[specific-service]/` ONLY when:
  1. Performance: >100 requests/sec on specific endpoint
  2. Scaling: Service needs independent horizontal scaling
  3. Deployment: Independent release cycle required
  4. Example: `services/reminder-worker/` (async reminder processing) ← only if needed
- ALWAYS document decision in `memory-bank/techContext.md` Architectural Decisions section

---

### **AGENTS & AUTOMATION → `/agents/`**

**Rule:** All AI automation centralized; skills registered in `/agents/skills/`

| Component | Folder | Purpose |
|-----------|--------|---------|
| **Skills** | `agents/skills/` | Reusable agent capabilities (documented) |
| **Tools** | `agents/tools/` | Agent tools & MCP servers |
| **Development Rules** | `.agents/` | RULE files with agent constraints |

**Examples of CORRECT organization:**
- ✅ `agents/skills/APPOINTMENT_REMINDER_PROCESSOR.md` (documented skill)
- ✅ `agents/skills/[NEW_SKILL_NAME].md` (new skills here)

---

### **SHARED CODE → `/packages/shared/`**

**Rule:** TypeScript types and schemas shared between frontend and backend go HERE, not duplicated.

| Component | Folder | Purpose |
|-----------|--------|---------|
| **Shared Types** | `packages/shared/src/types/` | Appointment, Patient, Clinic types (single source of truth) |
| **Shared Schemas** | `packages/shared/src/schemas/` | Zod/Pydantic schemas for validation |

**Examples of CORRECT organization:**
- ✅ `packages/shared/src/types/appointment.ts` (Appointment type, shared by frontend + backend)
- ✅ `packages/shared/src/schemas/appointment-schema.ts` (validation logic)

**Examples of FORBIDDEN structure:**
- ❌ `uis/website/types/appointment.ts` + `services/api/models/appointment.py` (duplicate definitions)
- ❌ `packages/frontend-utils/` (use packages/shared/)
- ❌ `packages/backend-utils/` (use packages/shared/)

---

## 🚫 Non-Negotiable Rules

| Rule | Violation | Consequence |
|------|-----------|------------|
| **One Website** | Creating `uis/admin/`, `uis/portal/`, `uis/app-v2/` | PR REJECTED; rewrite as feature in existing app |
| **One Backoffice** | Creating `uis/dashboard/`, `uis/legacy-ui/` | PR REJECTED; refactor into backoffice |
| **One API Service** | Creating `services/appointments-api/`, `services/api-v2/` | PR REJECTED; use routers in services/api/ |
| **No Microservices (early)** | Extracting microservices without CTO approval | PR BLOCKED; must document rationale in GitHub issue |
| **Shared Types** | Duplicating types in uis/ and services/ separately | PR REJECTED; centralize in packages/shared/ |
| **Agents in .agents/** | Creating agents/ folder outside structure | File/folder protection rule; needs CTO approval |
| **Rules in .agents/** | Creating development rules outside `.agents/` folder | File protection; use `.agents/RULE_NNN.md` only |

---

## 📊 Folder Organization Matrix

| Folder | App Type | Allowed Count | Duplication Allowed? | Purpose |
|--------|----------|---------------|----------------------|---------|
| **uis/website/** | Public website | 1 | ❌ NO | Patient portal + clinic info |
| **uis/backoffice/** | Admin dashboard | 1 | ❌ NO | Internal operations, reporting |
| **services/api/** | FastAPI backend | 1 | ❌ NO (unless microservice) | Centralized API service |
| **packages/shared/** | Shared types/schemas | 1 | ❌ NO | Single source of truth |
| **agents/skills/** | Reusable skills | Unlimited | ✅ Encouraged | Each skill unique |
| **.agents/RULE_* ** | Development rules | Unlimited | ✅ Encouraged | Each rule unique |

---

## ✅ Definition of Done (DoD)

A folder structure change is considered **COMPLETE** when:

```
Before Creating New Folder/App:
  ✓ Checked .agents/README.md folder registry
  ✓ Confirmed this code doesn't already exist in allowed folder
  ✓ Reviewed RULE_002 to verify WHERE it belongs
  ✓ Verified count limits (max 1 website, max 1 backoffice, etc.)
  ✓ For microservice: documented rationale in GitHub issue
  ✓ For shared code: moved to packages/shared/

Code Review Phase:
  ✓ PR reviewer checks folder location against RULE_002
  ✓ PR reviewer confirms no duplicate/parallel apps created
  ✓ No violations of monorepo structure rules
  ✓ All new code in correct folder per rule

Merge Phase:
  ✓ Folder structure verified against authoritative diagram
  ✓ No parallel applications introduced
  ✓ Monorepo remains clean and organized
  ✓ Single source of truth maintained for shared code
```

---

## 🔐 Violations & Enforcement

| Violation | Evidence | Consequence | Escalation |
|-----------|----------|------------|-----------|
| Creating `uis/admin/` or `uis/dashboard/` | Folder exists in PR | PR REJECTED | Rewrite to use uis/backoffice/ |
| Creating `services/appointments-api/` or microservice without approval | Folder exists in PR | PR BLOCKED | Document rationale in GitHub issue, get CTO approval |
| Duplicating shared types in multiple locations | Types in uis/ AND services/ separately | PR REJECTED | Move to packages/shared/ |
| Creating development rule outside `.agents/RULE_NNN.md` format | File in agents/ or docs/ | PR REJECTED | Move to `.agents/RULE_NNN.md` |
| Parallel-maintaining legacy app (e.g., `uis/legacy-ui/`) | Old app + new app coexist >3 months | ESCALATION | CTO + Team lead: plan refactoring sprint |
| Ignoring microservice decision (extracting without rationale) | Microservice folder without documented decision | PR BLOCKED | Merge into services/api/, or document architectural decision |

---

## 📞 Contact & Escalation

| Situation | Contact | Decision Authority |
|-----------|---------|-------------------|
| "Where should I put this feature?" | Architecture channel (Slack) | CTO James Osei |
| "Can I create a microservice?" | GitHub issue [ARCHITECTURE] | CTO (requires documented rationale) |
| "Should I create a new UI app?" | Backend team lead | CTO (answer: NO, use existing) |
| "Can I keep the old app alongside new one?" | CTO James Osei | CTO (answer: NO, refactor) |
| "PR rejected for folder violation" | PR reviewer + CTO | CTO (final authority) |

---

## 🗂️ Related Documentation

- **Tech Stack:** `memory-bank/techContext.md` (Section: Technology Stack)
- **Architecture:** `memory-bank/techContext.md` (Section: Architecture Overview)
- **Architectural Decisions:** `memory-bank/techContext.md` (Section: 8 Key Decisions)
- **File Protection:** `AGENTS.md` (Section: Protected Files & Directories)
- **Monorepo Guide:** `uis/README.md`, `services/README.md`

---

## 📋 Quick Reference Checklist

Before creating ANY new folder or app:

- [ ] Read RULE_002 (this file)
- [ ] Check `.agents/README.md` folder registry
- [ ] Verify folder not already in monorepo
- [ ] Confirm this rule applies (is it frontend/backend/agent code?)
- [ ] Respect count limits (1 website, 1 backoffice, 1 API service)
- [ ] For microservice: GitHub issue [ARCHITECTURE] + CTO approval
- [ ] For shared code: use packages/shared/ (not duplicated)
- [ ] Submit PR with link to memory-bank/techContext.md rationale
- [ ] Get approval before merging

**Shorthand Rule:** 
> "If it's a full application (website, dashboard, API), it belongs in the monorepo **exactly once**. If it's supporting code (types, utils), it goes in `packages/shared/`."

---

**Last Updated:** November 17, 2026  
**Review Cycle:** Annually or upon architectural decision change  
**Owner:** CTO James Osei  
**Version:** 1.0
