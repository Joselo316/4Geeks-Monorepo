# Memory Bank — HealthCore Digital Project Knowledge Base

**Purpose:** Central repository of authoritative business and technical context that all agents read at session startup

**Version:** 1.0  
**Last Updated:** November 17, 2026  
**Maintained By:** Copilot & Development Team

---

## 📚 Structure: Business Context + Technical Context

The Memory Bank contains **TWO complementary knowledge bases**:

### 1️⃣ BUSINESS CONTEXT

**File:** [`projectbrief.md`](projectbrief.md)  
**Purpose:** Define the business problem, organization, and strategic goals  
**Length:** 276 lines  
**Audience:** All team members (business stakeholders, engineers, agents)

**Contains:**
- 📋 Business Description (12 clinics, 2 regions, ~200 employees, $28M revenue)
- 🎯 Project Objectives (6 strategic goals: modernize tech, improve efficiency, optimize patient experience, maximize revenue, ensure compliance, enable scaling)
- ⚠️ Problems Being Solved (4 critical problems with business impact):
  1. Fragmented Clinical Systems (USA vs UK disconnected)
  2. Clinical Staff Administrative Burden (35 min/day wasted on documentation)
  3. **Critical No-Show Rate (22% = $1.8M/year lost revenue)**
  4. Insurance Claim Rejection Rate (revenue leakage)
- 🏥 Organizational Structure (3 key departments: Clinical, Operations, Finance + growing tech team)
- 📊 Key Success Metrics (data-driven targets)
- 👥 Stakeholders (Dr. Sandra Okonkwo CEO, James Osei CTO, etc.)

**Why Read This:**
- Understand **WHY** we're building (business problems)
- Understand **WHO** we serve (12 clinics, 2 regions)
- Understand **WHAT SUCCESS LOOKS LIKE** (measurable metrics)
- Align your work with business objectives

**Key Statistics:**
- 22% no-show rate = $1.8M/year impact (Problem #3 - **directly addressed by Appointment Reminder Processor skill**)
- 35 min/day clinician burden = automation opportunity (Problem #2)
- 2 regions = must handle USA (HIPAA) + UK (GDPR)
- 12 clinics = scale across network

---

### 2️⃣ TECHNICAL CONTEXT

**File:** [`techContext.md`](techContext.md)  
**Purpose:** Define the technical architecture, decisions, and constraints  
**Length:** 723 lines  
**Audience:** Engineers, architects, agents

**Contains:**
- 🏗️ Architecture Overview (monorepo with clear responsibility separation)
- 📦 Technology Stack (by component):
  - **Backend:** FastAPI + Python + PostgreSQL + Redis
  - **Frontend:** React + TypeScript + Tailwind CSS
  - **Data Layer:** Unified Clinical API, Event-driven architecture
  - **AI & Agents:** Python agents, Pydantic for validation
  - **Infrastructure:** Docker + AWS/GCP + Data residency enforcement
- 🏛️ Architectural Decisions (8 documented decisions):
  1. Monorepo for unified development
  2. Event-driven patient data updates
  3. Centralized authentication via API Gateway
  4. Separation: Backend services + Frontend UIs
  5. Shared TypeScript types across frontend
  6. Python agents for automation
  7. **Data Residency by Region (USA data in USA, UK data in UK)**
  8. AI-assisted workflows (documentation, predictions)
- 🔒 Security & Compliance Constraints:
  - **HIPAA** (USA patients): Data residency, audit logging, PHI handling
  - **UK GDPR** (UK patients): Consent management, data portability, privacy by design
  - Encryption at rest + in transit (TLS 1.2+)
  - PII minimization (never log full SSN, MRN, diagnosis)
  - Zero-trust access control
- ⚙️ Technical Constraints & Trade-offs:
  - Latency targets (API <200ms, frontend <1s)
  - Throughput (handle 2,000-3,000 patients/day)
  - Cost optimization (shared infra, batch processing)
  - Scalability (support clinic expansion to 20+ locations)
- 🚀 Deployment Strategy (staging → production with approval gates)
- 📊 Performance Targets (desktop/mobile response times)

**Why Read This:**
- Understand **HOW** we build (architecture decisions)
- Understand **WHAT CONSTRAINTS** we work within (data residency, compliance)
- Understand **TECHNICAL CHOICES** and reasoning
- Align your implementation with established patterns

**Key Constraints:**
- Data must stay in region (USA data ≠ UK data, never transfer)
- All APIs must be auditable (HIPAA requirement)
- Encryption non-negotiable (at rest + in transit)
- PII in logs = security violation (never store full SSN, MRN, diagnosis in logs)
- FastAPI backend must validate ALL inputs (Pydantic)

---

### 3️⃣ PROGRESS & ROADMAP

**File:** [`progress.md`](progress.md)  
**Purpose:** Track development status, sprints, blockers, and roadmap  
**Length:** 539 lines  
**Audience:** Project managers, tech lead, engineers, stakeholders

**Contains:**
- 📊 Executive Summary (current phase, what's done, what's next)
- 🏗️ Current State by Component:
  - Backend (FastAPI skeleton ready)
  - Frontend (React skeleton ready)
  - Data Layer (Unified API architecture defined)
  - Agents (framework ready)
  - Infrastructure (Docker setup ready)
- 🎯 Phase Breakdown:
  - **Phase 1 (Current):** Foundation APIs + Patient Data Layer
  - **Phase 2:** Automation Workflows (AI agents, claim processing)
  - **Phase 3:** Analytics & Reporting
  - **Phase 4:** Scaling & Optimization
- 📈 Tracking & Metrics (velocity, burndown, success criteria)
- 🚨 Blockers (if any)
- 🛣️ Roadmap (3-6 months ahead)

**Why Read This:**
- Understand **WHERE WE ARE** (current progress)
- Understand **WHAT'S NEXT** (upcoming sprints)
- Understand **WHAT'S BLOCKING** (dependencies)
- Plan your work within the phase/sprint context

---

## 🎯 How Agents Use This Memory Bank

### At Session Startup

Every agent reads the memory bank sections **relevant to its role**:

```
Session Start
  ↓
Agent loads memory bank:
  ├─ projectbrief.md (always - business context)
  ├─ techContext.md (always - technical constraints)
  └─ progress.md (optional - roadmap context)
  ↓
Agent understands:
  • Business problems being solved
  • Technical constraints & decisions
  • Compliance requirements (HIPAA/GDPR)
  • Data residency rules
  • Current development phase
  ↓
Agent can now work aligned with business + technical strategy
```

### Role-Specific Reading Paths

**🔧 Backend Agent (FastAPI):**
1. `projectbrief.md` → Business problems (focus on #1, #3, #4)
2. `techContext.md` → Backend stack, compliance, data layer
3. `progress.md` → Phase 1 Sprint status

**🎨 Frontend Agent (React):**
1. `projectbrief.md` → Patient experience goals
2. `techContext.md` → Frontend stack, compliance, performance targets
3. `progress.md` → Frontend component status

**🤖 ML/Agent Developer:**
1. `projectbrief.md` → Business problems (#3 no-show, #2 documentation burden)
2. `techContext.md` → AI decisions, agent architecture
3. `progress.md` → Phase 2 agent roadmap

**🛡️ Compliance/Security Officer:**
1. `projectbrief.md` → Compliance objectives
2. `techContext.md` → HIPAA/GDPR constraints, encryption, audit logging
3. → Review all code against constraints

---

## 📋 Content Summary Matrix

| Aspect | Business Context | Technical Context | Progress |
|--------|------------------|-------------------|----------|
| **What problem does it solve?** | ✅ 4 problems defined | ✅ Architectural solutions | ✅ Phase breakdown |
| **Why are we building this?** | ✅ Business objectives | ✅ Technical decisions | ✅ Strategic roadmap |
| **Who is involved?** | ✅ Stakeholders listed | ✅ Tech stack listed | ✅ Team & sprints |
| **What are constraints?** | ✅ Budget, timeline | ✅ HIPAA, GDPR, data residency | ✅ Dependencies |
| **What's the success metric?** | ✅ KPIs defined | ✅ Performance targets | ✅ Tracking metrics |
| **What's next?** | ✅ Roadmap | ✅ Phase plan | ✅ Sprint backlog |

---

## 🔗 Integration with Other Documents

### Memory Bank is Referenced By:

- **AGENTS.md** (line 15) → "Memory Bank Structure" section
  - Specifies which memory files each agent type must read
  - Links to all 3 memory bank files

- **agents/DEVELOPMENT_RULES.md** (Rule #2)
  - Security & Compliance First
  - References HIPAA/GDPR constraints from techContext.md

- **agents/skills/APPOINTMENT_REMINDER_PROCESSOR.md**
  - Problem: From projectbrief.md Problem #3 ($1.8M no-show cost)
  - Architecture: From techContext.md Unified API design
  - Compliance: From techContext.md HIPAA/GDPR constraints

- **docs/** folder
  - FILE_PROTECTION_MATRIX.md references AGENTS.md
  - WORKFLOW_SUMMARY.md links to memory bank

---

## ✅ Completeness Checklist

### Business Context (projectbrief.md)
- [x] Organization description (HealthCore network, 12 clinics, 2 regions)
- [x] Business objectives (6 strategic goals)
- [x] Problems being solved (4 critical problems with business impact)
- [x] Success metrics (measurable, data-driven)
- [x] Stakeholders (key people & roles)
- [x] Scale & constraints (200 employees, $28M revenue)

### Technical Context (techContext.md)
- [x] Architecture overview (monorepo, components, responsibility)
- [x] Technology stack (all layers: backend, frontend, data, AI, infra)
- [x] Architectural decisions (8 documented decisions with reasoning)
- [x] Security & compliance constraints (HIPAA, GDPR, encryption, audit logging)
- [x] Technical constraints (latency, throughput, scalability, cost)
- [x] Data residency enforcement (USA data in USA, UK data in UK)
- [x] Deployment strategy (stages, approval gates)
- [x] Performance targets (SLAs, response times)

### Progress & Roadmap (progress.md)
- [x] Current phase (Foundation → Ready for Phase 1)
- [x] Component status (all skeletons ready)
- [x] Sprint breakdown (Phase 1 Sprints 1-4)
- [x] Blockers & dependencies (tracked)
- [x] Success criteria (measurable by sprint)
- [x] 3-6 month roadmap (Phase 1-4 planning)

---

## 🚀 Usage Example: Creating the Appointment Reminder Processor Skill

**How the Memory Bank Guided This Work:**

1. **Business Context Guidance:**
   - `projectbrief.md` Problem #3: "22% no-show rate = $1.8M/year lost revenue"
   - Skill objective: Reduce 22% → 15% (target reduction)
   - Business impact: ~$500K/year savings
   - Scope: All 12 clinics (9 USA + 3 UK)

2. **Technical Context Guidance:**
   - `techContext.md` Architectural Decision #2: Event-driven data updates
   - Skill uses: Unified Clinical API for appointment data
   - `techContext.md` Constraint: Data residency enforcement
   - Skill implementation: USA reminders stay in USA, UK in UK
   - `techContext.md` Compliance: HIPAA + GDPR
   - Skill validation: 100% consent check before sending reminders

3. **Progress Guidance:**
   - `progress.md` Phase 1 Sprint 4: "Implement no-show reduction workflow"
   - Skill fits in: Foundation phase before patient-facing features
   - Dependencies: Unified API (Phase 1 Sprint 1) must complete first

**Result:** Skill is 100% aligned with business + technical + roadmap context

---

## 📖 Reading Time Estimates

| Document | Length | Reading Time | Best For |
|----------|--------|--------------|----------|
| `projectbrief.md` | 276 lines | 15-20 min | Understanding business context |
| `techContext.md` | 723 lines | 30-40 min | Understanding technical decisions |
| `progress.md` | 539 lines | 20-25 min | Understanding current status/roadmap |
| **All three** | 1,538 lines | **60-90 min** | Complete onboarding |

**Recommended:** New team members read all three during first week. Agents read relevant sections at session start.

---

## 🔄 Maintenance Cycle

- **Updated:** Every sprint end (2 weeks)
- **Review:** Every month by tech lead + product manager
- **Audit:** Every quarter for compliance consistency
- **Version Control:** Committed to main branch with approval gates

---

## 📞 Questions & Support

**Q: Where is the business context?**  
A: [`projectbrief.md`](projectbrief.md)

**Q: Where are the technical constraints?**  
A: [`techContext.md`](techContext.md) — Sections on HIPAA, GDPR, data residency, encryption

**Q: What's the current project status?**  
A: [`progress.md`](progress.md) — Executive Summary section

**Q: How do I know what to build next?**  
A: [`progress.md`](progress.md) → Roadmap + Phase breakdown

**Q: Where are the compliance requirements?**  
A: [`techContext.md`](techContext.md) → Security & Compliance Constraints section

---

**Memory Bank Status:** ✅ **COMPLETE** (Both business + technical context)  
**Last Verified:** November 17, 2026  
**Next Review:** 2026 Q4 or upon major architecture change
