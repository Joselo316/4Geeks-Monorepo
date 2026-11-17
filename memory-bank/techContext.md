# HealthCore Digital — Technical Context

## 🏗️ Architecture Overview

HealthCore Digital follows a **monorepo architecture** organized by responsibility, with clear separation of concerns:

```
4Geeks-Monorepo/
├── uis/                 # Frontend applications (React/TypeScript)
│   ├── website/         # ONE public website
│   └── backoffice/      # ONE internal admin dashboard
├── services/            # FastAPI backend (centralized)
│   └── api/             # ONE centralized FastAPI service
├── packages/shared/     # Shared TypeScript types (single source of truth)
├── agents/              # AI agents and automation (Python)
├── .agents/             # Agent development rules (RULE_NNN.md files)
├── workflows/           # n8n orchestration flows
├── mcps/                # MCP tool servers
├── data/                # Datasets and pipelines
├── infra/               # Docker, deployment configs
└── docs/                # Architecture and decisions
```

### Design Principles

1. **Single Responsibility per Layer**
   - `uis/website/` = ONE public-facing website (React/TypeScript)
   - `uis/backoffice/` = ONE internal admin dashboard (React/TypeScript)
   - `services/api/` = ONE centralized FastAPI backend with routers per domain
   - `packages/shared/` = Shared types (single source of truth for frontend + backend)
   - `agents/` = AI-driven automation and decision-making
   - `workflows/` = Cross-system orchestration
   - `.agents/` = Development rules governing agent behavior

2. **Avoid Premature Fragmentation**
   - ⚠️ **NO parallel apps:** Create `uis/admin/`, `uis/portal/`, `uis/app-v2/` ← FORBIDDEN
   - ⚠️ **NO separate microservices early:** Create `services/appointments-api/`, `services/patient-service/` ← FORBIDDEN
   - ✅ Keep one main FastAPI app with routers per domain (appointments.py, patients.py, etc.)
   - ✅ Extract microservices ONLY when necessary (high load, independent scaling)
   - ⚠️ **NO duplicated types:** Types in `uis/` AND `services/` ← FORBIDDEN; use `packages/shared/`

3. **Data-Driven**
   - Single source of truth for domain models in `packages/shared/`
   - Shared types prevent schema drift between frontend/backend
   - One type definition used by both uis/website, uis/backoffice, and services/api

4. **Compliance-First**
   - All patient data handling must follow HIPAA (USA) and UK GDPR
   - Audit logging and encryption mandatory in every layer
   - Region-specific data residency: USA data stays in USA, UK data stays in UK

---

### Where Code Belongs (Authoritative Reference)

**See RULE_002:** [`.agents/RULE_002_MONOREPO_FOLDER_STRUCTURE.md`](../agents/RULE_002_MONOREPO_FOLDER_STRUCTURE.md) for complete folder structure rules.

| Artifact Type | Folder | Count | Allowed? | Example |
|---------------|--------|-------|----------|---------|
| **Public Website** | `uis/website/` | 1 | ❌ NO duplicates | Patient portal, clinic info pages |
| **Admin Dashboard** | `uis/backoffice/` | 1 | ❌ NO duplicates | Clinic management, patient records, reporting |
| **Backend API** | `services/api/` | 1 | ❌ NO duplicates (unless microservice) | FastAPI service with routers: appointments, patients, clinics |
| **Shared Types** | `packages/shared/` | 1 | ❌ NO duplicates | Appointment, Patient, Clinic TypeScript interfaces |
| **Agent Skills** | `agents/skills/` | Unlimited | ✅ YES (each unique) | APPOINTMENT_REMINDER_PROCESSOR.md, [future skills] |
| **Development Rules** | `.agents/RULE_NNN.md` | Unlimited | ✅ YES (each unique) | RULE_001, RULE_002, [future rules] |

---

## 💻 Technology Stack

### Frontend

| Technology | Purpose | Version |
|---|---|---|
| **React** | UI framework | 18+ |
| **TypeScript** | Type safety in frontend code | 5.x |
| **Vite** | Build tool and dev server | 5.x |
| **Tailwind CSS** | Utility-first styling | 3.x |
| **React Query / TanStack Query** | Server state management | 5.x |
| **React Router** | Client-side routing | 6.x |
| **Zod** | Runtime schema validation | Latest |
| **Storybook** | Component documentation | 7.x |

**Decision Rationale:**
- React chosen for ecosystem maturity and AI component library support
- TypeScript enforces data contracts across frontend/backend
- Vite for fast HMR during development
- Tailwind for rapid, consistent UI
- React Query handles healthcare real-time data (appointments, notes)

### Backend

| Technology | Purpose | Version |
|---|---|---|
| **Python** | Backend language | 3.11+ |
| **FastAPI** | Web framework | 0.100+ |
| **Pydantic** | Data validation and serialization | 2.x |
| **SQLAlchemy** | ORM for database abstraction | 2.x |
| **Alembic** | Database migrations | 1.x |
| **Python-jose** | JWT authentication | Latest |
| **Cryptography** | Encryption for PHI | Latest |
| **APScheduler** | Background scheduling (reminders, CME alerts) | 3.x |
| **Celery** | Async task queue (optional, for high-volume workflows) | 5.x |
| **Redis** | Cache and message broker | 7.x |
| **pytest** | Unit and integration testing | 7.x |
| **black** | Code formatting | Latest |
| **ruff** | Linting | Latest |

**Decision Rationale:**
- FastAPI chosen for async support (critical for high-concurrency healthcare APIs)
- Pydantic ensures data validation at API boundaries
- SQLAlchemy enables easy switching between databases (PostgreSQL, MySQL)
- Python ecosystem mature for AI/ML tasks (NumPy, Pandas, scikit-learn)
- Cryptography library for HIPAA-compliant encryption

### Data Layer

| Technology | Purpose | Version |
|---|---|---|
| **PostgreSQL** | Primary database (HIPAA/GDPR capable) | 14+ |
| **Redis** | Caching and message broker | 7.x |
| **Apache Kafka** (optional) | Event streaming for high-volume data | 3.x |
| **Alembic** | Schema version control | 1.x |

**Decision Rationale:**
- PostgreSQL chosen for:
  - Native encryption at rest (pgcrypto)
  - Row-level security (HIPAA audit trails)
  - JSONB for flexible clinic data structures
  - Mature ecosystem
- Redis for session management and appointment availability caching
- Kafka optional for future event-driven architecture (e.g., billing events)

### AI & Agents

| Technology | Purpose | Version |
|---|---|---|
| **LangChain** | LLM orchestration | 0.1+ |
| **OpenAI API** | LLM provider (documentation, coding assistance) | Latest |
| **Claude API** (Anthropic) | LLM provider (compliance analysis, clinical reasoning) | Latest |
| **AutoGen** | Multi-agent orchestration (optional) | Latest |
| **scikit-learn** | ML models (no-show prediction, claim risk scoring) | 1.3+ |
| **Pandas** | Data manipulation for analysis | 2.x |
| **Polars** | High-performance data processing | Latest |
| **MLflow** | Model tracking and deployment | 2.x |

**Decision Rationale:**
- LangChain abstracts LLM provider switching
- OpenAI for fast prototyping and documentation tasks
- Claude for compliance-sensitive operations (UK GDPR, HIPAA analysis)
- scikit-learn for interpretable ML models (medical professionals need explainability)
- MLflow tracks model performance and versioning

### Orchestration & Workflows

| Technology | Purpose | Version |
|---|---|---|
| **n8n** | Low-code workflow automation | 1.x |
| **Apache Airflow** (optional) | DAG-based data pipelines | 2.x |

**Decision Rationale:**
- n8n for non-technical staff (clinic operations) to automate workflows
- Airflow for complex multi-step data pipelines (optional future)
- Both integrate with FastAPI via webhooks/APIs

### Infrastructure

| Technology | Purpose | Version |
|---|---|---|
| **Docker** | Containerization | 24+ |
| **Docker Compose** | Local dev orchestration | 2.x |
| **Kubernetes** (optional) | Production orchestration | 1.28+ |
| **GitHub Actions** | CI/CD pipelines | Latest |
| **AWS / Azure** (optional) | Cloud hosting | — |

**Decision Rationale:**
- Docker Compose for local dev (simple, no cloud setup needed)
- Kubernetes optional for future multi-region deployment (US + UK data residency)
- GitHub Actions for automated testing and linting

### Shared Types

| Technology | Purpose | Version |
|---|---|---|
| **TypeScript** | Shared type definitions | 5.x |
| **Zod** | Runtime validation schemas | Latest |
| **OpenAPI / Swagger** | API documentation | 3.0 |

**Decision Rationale:**
- Shared TypeScript types prevent frontend/backend schema drift
- Zod validates data at runtime (defensive programming for healthcare)
- OpenAPI auto-generates docs from FastAPI

### Testing & Quality

| Technology | Purpose | Version |
|---|---|---|
| **pytest** | Python unit/integration tests | 7.x |
| **Jest** | JavaScript/TypeScript tests | 29.x |
| **Cypress** | E2E frontend testing | 13.x |
| **Postman / Thunder Client** | API testing | — |
| **SonarQube** (optional) | Code quality scanning | Latest |
| **OWASP ZAP** (optional) | Security scanning | Latest |

**Decision Rationale:**
- pytest + Jest for comprehensive unit test coverage
- Cypress for critical patient flows (booking, medical records)
- Security scanning mandatory for HIPAA/GDPR compliance

---

## 🏛️ Architectural Decisions

### 1. **Centralized FastAPI Backend (Not Microservices)**

**Decision:** Keep one main FastAPI application with routers per domain, not separate microservices per domain.

**Reasoning:**
- HealthCore has 12 clinics with ~200 staff — traffic is moderate, not Netflix-scale
- Microservices add operational complexity (deployment, monitoring, debugging)
- One FastAPI app can scale horizontally if needed
- Simplifies cross-domain queries (appointments → patient → claims)

**When to Extract:** Only extract a microservice when:
- A specific domain requires independent scaling (e.g., billing service under peak load)
- Two domains have fundamentally different data models
- A team owns and operates the service independently

**Directory Structure:**
```
services/
└── api/
    ├── routers/
    │   ├── appointments.py        # Patient scheduling
    │   ├── clinical_notes.py      # Clinical documentation
    │   ├── billing_claims.py      # Insurance claims
    │   ├── patients.py            # Patient records
    │   ├── users.py               # Authentication
    │   └── audit.py               # Compliance logging
    ├── models/                    # SQLAlchemy ORM models
    ├── schemas/                   # Pydantic request/response schemas
    ├── dependencies/              # DI (database sessions, auth, permissions)
    ├── middleware/                # CORS, logging, error handling
    ├── main.py                    # FastAPI app initialization
    └── requirements.txt
```

---

### 2. **Unified Patient Identity (Master Patient Index)**

**Decision:** Create a centralized Master Patient Index (MPI) that maps patients across both EHR systems (USA + UK).

**Reasoning:**
- Two EHR systems don't communicate — patients can have different IDs in each system
- MPI is single source of truth for patient identity
- Enables safe cross-region queries without data integrity issues

**Implementation:**
```
database schema:
- master_patients (id, national_id, email, phone, clinic_region)
- patient_records (master_patient_id, ehr_id, ehr_system_name, record_data)
```

**Deduplication Strategy:**
- Primary: National ID (SSN in USA, NHS Number in UK)
- Secondary: Deterministic fuzzy matching (name + DOB + phone) for unmatched records

---

### 3. **Encryption at Rest for All PHI**

**Decision:** Encrypt all Protected Health Information (PHI) at the database layer using PostgreSQL pgcrypto + application-level encryption for sensitive fields.

**Reasoning:**
- HIPAA requires encryption of PHI both at rest and in transit
- UK GDPR requires pseudonymization and encryption
- Transparent to applications (use SQLAlchemy hooks)

**Implementation:**
```python
# In models.py
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy_utils import EncryptedType

class PatientRecord(Base):
    __tablename__ = "patient_records"
    
    id = Column(Integer, primary_key=True)
    ssn = Column(EncryptedType())          # Encrypted at application level
    health_data = Column(Text())           # Encrypted at DB level
    created_at = Column(DateTime)
```

---

### 4. **Audit Logging for Compliance**

**Decision:** Every access to patient data is logged with actor, action, timestamp, and context.

**Reasoning:**
- HIPAA & UK GDPR require audit trails for data access
- Enables quick investigation of data breaches
- Supports compliance monitoring dashboard

**Implementation:**
```
audit_logs table:
- id (primary key)
- actor_id (user who performed action)
- action (READ, WRITE, DELETE)
- resource_type (patient, appointment, claim)
- resource_id (what was accessed)
- timestamp (when)
- context (IP, session, reason)
```

**Automatic Logging:**
- Use SQLAlchemy events to log all database changes
- FastAPI middleware to log API access
- Sensitive operations flagged for review

---

### 5. **Role-Based Access Control (RBAC)**

**Decision:** Implement RBAC with clinic-scoped permissions (a clinician in Austin clinic cannot see Manchester patients).

**Reasoning:**
- Different roles have different data access rights
- Regional data residency requirements (UK data must stay in UK systems)
- Prevents accidental cross-clinic data leaks

**Roles:**
- `clinic_admin` — Manage appointments, staff, clinic settings
- `clinician` — View/edit patient records within own clinic
- `billing_analyst` — Process claims and revenue reports
- `compliance_officer` — Audit trails and regulatory reporting
- `system_admin` — Full access (rare, audit-logged)

**Permission Scoping:**
```python
# Clinician can only see patients from their clinic
query = db.query(Patient).filter(
    Patient.clinic_id == current_user.clinic_id
)
```

---

### 6. **Async-First Patient Engagement**

**Decision:** Use FastAPI's async capabilities + background job queue for time-sensitive patient notifications (appointment reminders, billing alerts).

**Reasoning:**
- Appointment reminders must go out at specific times (not on-demand)
- Clinician should not wait for SMS/email sending during API call
- High concurrency: 1000+ patients receiving reminders simultaneously

**Implementation:**
```python
# In routers/appointments.py
@router.post("/appointments/{appointment_id}/send_reminder")
async def send_appointment_reminder(appointment_id: int):
    # Queue job, return immediately
    task.delay(send_sms_reminder, appointment_id)
    return {"status": "queued"}

# Background worker processes jobs
def send_sms_reminder(appointment_id):
    appointment = db.query(Appointment).get(appointment_id)
    sms_provider.send(appointment.patient.phone, reminder_text)
```

Using:
- **APScheduler** for recurring jobs (daily reminder batches)
- **Celery + Redis** for high-volume one-off jobs (scaling later)

---

### 7. **Data Residency Compliance (USA vs UK)**

**Decision:** Separate database instances for USA and UK data, with strict data residency rules.

**Reasoning:**
- UK GDPR mandates UK personal data stays in UK jurisdiction
- USA HIPAA doesn't restrict, but operational separation simplifies compliance
- Different encryption/security standards can apply per region

**Implementation:**
```
services/api/db/
├── connection_us.py      # PostgreSQL in AWS US
├── connection_uk.py      # PostgreSQL in AWS UK (or on-premise)
├── routing.py            # Routes queries to correct database

# Middleware detects clinic region from request
@app.middleware("http")
async def region_routing_middleware(request, call_next):
    clinic_region = extract_clinic_region(request)
    request.state.db = get_db_for_region(clinic_region)
    return await call_next(request)
```

**Shared Data (Allowed Across Regions):**
- Organizational metadata (clinic names, staff directory)
- System logs (non-PHI)

**Isolated Data (Region-locked):**
- Patient records
- Clinical notes
- Billing information

---

### 8. **AI-Assisted Workflows (Not Fully Automated)**

**Decision:** AI suggestions require human review/approval before execution (especially for clinical and billing decisions).

**Reasoning:**
- Healthcare decisions have legal/safety implications
- Builds clinician trust in AI tooling
- Provides audit trail for compliance

**Workflow Example (Claim Review):**
```
1. Billing analyst submits claim
2. AI model: "This claim has 72% risk of rejection (missing modifier)"
3. Human reviews AI suggestion
4. Human approves or overrides
5. Claim submitted with approval logged
6. If denied → workflow triggers with AI-suggested fix
```

---

## 🔐 Security & Compliance Constraints

### HIPAA Compliance (USA)

| Requirement | Implementation |
|---|---|
| **Encryption at rest** | PostgreSQL pgcrypto + application-level for sensitive fields |
| **Encryption in transit** | TLS 1.2+ for all connections |
| **Access controls** | RBAC, MFA for clinical staff |
| **Audit logging** | All access to PHI logged with actor, action, timestamp |
| **Data minimization** | Only collect data needed for care/operations |
| **Breach notification** | Automated incident response workflow |
| **Business Associate Agreements** | Required for all third-party services |

### UK GDPR Compliance (UK)

| Requirement | Implementation |
|---|---|
| **Data Processing Addendum (DPA)** | Required for all processors (cloud providers, vendors) |
| **Right to be forgotten** | API endpoint to delete patient records + related data |
| **Data portability** | Export patient data in machine-readable format (JSON/CSV) |
| **Consent management** | Explicit opt-in for communications, tracking, analytics |
| **Data Protection Impact Assessment (DPIA)** | Required before new high-risk processing |
| **Data residency** | UK personal data must stay in UK jurisdiction |
| **Legitimate interest test** | Documented justification for each data use |

### Both Frameworks

- **No cross-border data transfer** without explicit safeguards
- **Incident response plan** tested quarterly
- **Staff training** mandatory (HIPAA, GDPR, security awareness)
- **Vendor audits** annual security reviews

---

## 🔧 Technical Constraints & Trade-offs

### Constraint #1: Legacy EHR Integration

**Problem:** USA and UK EHRs have different APIs and data models. No shared schema.

**Constraint:** Cannot modify EHRs (vendor lock-in), must build adapters.

**Solution:**
```
services/api/integrations/
├── ehr_us/
│   ├── api_client.py     # Vendor API calls
│   ├── schema_mapper.py  # Map vendor schema → unified schema
│   └── sync_worker.py    # Periodic sync to our database
├── ehr_uk/
│   ├── api_client.py
│   ├── schema_mapper.py
│   └── sync_worker.py
└── unified_schema.py     # Single patient record format
```

**Trade-off:** Eventual consistency (EHR data synced every 5-15 minutes, not real-time)

---

### Constraint #2: Real-Time Appointment Availability

**Problem:** Clinic staff need real-time view of available slots; EHR queries are slow.

**Constraint:** Cannot afford 2-second response time (staff will abandon UI).

**Solution:** Cache available slots in Redis, invalidate on booking.

```python
# Check cache first (< 100ms)
slots = redis.get(f"slots:{clinic_id}:{date}")
if not slots:
    # Fall back to EHR API if needed
    slots = fetch_from_ehr(clinic_id, date)
    redis.setex(f"slots:{clinic_id}:{date}", 3600, slots)
return slots
```

**Trade-off:** Slight staleness (up to 1 minute) acceptable for UX

---

### Constraint #3: AI Model Latency (No-Show Prediction)

**Problem:** Predict no-show probability at booking time (< 500ms required).

**Constraint:** Models take seconds to run; cannot block user.

**Solution:** Pre-compute during off-hours, serve cached predictions.

```python
# Off-peak batch job (11 PM)
def batch_predict_no_shows():
    upcoming_appointments = get_appointments_next_7_days()
    predictions = model.predict(upcoming_appointments)
    cache_predictions(predictions)

# Booking endpoint (real-time)
@router.post("/appointments")
async def create_appointment(request):
    pred = get_cached_no_show_prediction(request.patient_id, request.time)
    if pred.risk_score > 0.7:
        # Show reminder nudge to patient
        response["recommended_reminder"] = True
    return response
```

**Trade-off:** Predictions use data from previous day (acceptable for 24hr+ bookings)

---

### Constraint #4: Compliance Audit Performance

**Problem:** Compliance queries (e.g., "who accessed patient X in past 30 days?") must return instantly.

**Constraint:** Audit log table will have 100M+ rows (billions of access records).

**Solution:** Partition audit logs by date, create indexed views for common queries.

```python
# Partitioned table (auto-managed)
class AuditLog(Base):
    __tablename__ = "audit_logs"
    __table_args__ = (
        Index("idx_patient_timestamp", "patient_id", "timestamp"),
        {"postgresql_partition_by": "RANGE (timestamp)"}
    )

# Common query optimized
select * from audit_logs
where patient_id = X
  and timestamp > now() - interval '30 days'
order by timestamp desc
```

**Trade-off:** Slightly longer retention periods (90 days) instead of forever (cost/performance)

---

### Constraint #5: Multi-Tenancy (12 Clinics)

**Problem:** Each clinic has different workflows, staff, and patient populations.

**Constraint:** Cannot rebuild system for each clinic; must support multi-tenancy.

**Solution:** Single codebase, database row-level security per clinic.

```python
# Every table has clinic_id
class Patient(Base):
    __tablename__ = "patients"
    id = Column(Integer, primary_key=True)
    clinic_id = Column(Integer, ForeignKey("clinics.id"))  # Always present
    ssn = Column(String)
    # ...

# Middleware adds clinic_id filter to all queries
@app.middleware("http")
async def clinic_scoping(request, call_next):
    clinic_id = extract_clinic_id(request)
    request.state.clinic_id = clinic_id
    return await call_next(request)

# In SQLAlchemy base query
query = db.query(Patient).filter(Patient.clinic_id == request.state.clinic_id)
```

**Trade-off:** Slight performance overhead (extra filter on every query), but negligible at HealthCore's scale

---

## 📋 Development Workflow

### Local Development

```bash
# Start full stack locally
docker-compose up

# This starts:
# - PostgreSQL (port 5432)
# - Redis (port 6379)
# - FastAPI (port 8000)
# - React dev server (port 3000)
# - n8n (port 5678, optional)
```

### Code Organization

```
packages/shared/
└── types/
    ├── patient.ts
    ├── appointment.ts
    ├── claim.ts
    ├── clinic.ts
    └── user.ts

services/api/
├── schemas/
│   ├── patient.py          # Pydantic (from packages/shared types)
│   ├── appointment.py
│   └── claim.py
└── models/
    ├── patient.py          # SQLAlchemy
    ├── appointment.py
    └── claim.py

uis/backoffice/
└── src/
    ├── types.ts            # Import from @repo/shared-types
    ├── api/
    │   └── patient.ts
    └── components/
        └── PatientForm.tsx
```

### Git Workflow

- **Main branch:** Deployable, all tests pass, code reviewed
- **Feature branches:** `feature/patient-portal`, `fix/claim-rejection`
- **Pull requests:** Require 1 approval, all CI checks pass
- **Commit messages:** `type(scope): description` (commitizen style)

---

## 🚀 Deployment Strategy

### Environments

| Environment | Purpose | Region | Updates |
|---|---|---|---|
| **local** | Developer machines | localhost | On-demand |
| **dev** | Testing new features | AWS USA | Every merge to `develop` |
| **staging** | Pre-production testing | AWS USA + UK | Weekly |
| **production** | Live patient data | AWS USA + UK | Approved releases |

### Data Residency

- **USA Data:** AWS USA East (Virginia), encrypted at rest, encrypted in transit
- **UK Data:** AWS UK (London), separate PostgreSQL instance, GDPR-compliant
- **No cross-region queries** except for aggregated, non-PHI reports

### Automated Testing in CI

```
Pre-commit:
  - Lint (black, ruff)
  - Type check (mypy, TypeScript)
  - Unit tests (pytest, jest)
  
On PR:
  - Integration tests (full stack)
  - Security scan (OWASP ZAP)
  - Performance baseline (appointment queries < 500ms)
  
Before Production Deploy:
  - Full E2E test suite (Cypress)
  - Database migration dry-run
  - Compliance check (audit log coverage, encryption status)
```

---

## 📊 Performance Targets

| Operation | Target | Priority |
|---|---|---|
| Appointment booking | < 2 seconds | Critical |
| Patient record lookup | < 1 second | Critical |
| Claim submission | < 5 seconds | High |
| No-show prediction | < 500ms | High |
| Compliance query (30-day audit) | < 10 seconds | High |
| Appointment reminder batch | < 1 hour for 5000 patients | Medium |
| Daily reports | < 15 minutes | Medium |

---

## 🛠️ Future Considerations

### Scaling (Phase 2+)

- Extract **billing microservice** if claims volume > 50k/day
- Implement **Kafka** for real-time event streaming (EHR sync, billing events)
- Add **Kubernetes** for multi-region deployment
- Build **data lake** in Snowflake for analytics

### AI Enhancements (Phase 2+)

- Fine-tune LLM on HealthCore clinical notes (private LLM)
- Implement **RAG (Retrieval-Augmented Generation)** for clinical guidelines
- Build **interpretability dashboard** for AI suggestions
- Experiment with **federated learning** for privacy-preserving ML

### Compliance Automation (Phase 2+)

- Automated HIPAA/GDPR impact assessments
- Continuous compliance monitoring (real-time risk alerts)
- Automated breach response playbooks

---

## 📚 Reference & Standards

- **HIPAA Technical Safeguards:** https://www.hhs.gov/hipaa/for-professionals/security/security-technical-safeguards/index.html
- **UK GDPR Guide:** https://ico.org.uk/for-organisations/guide-to-the-general-data-protection-regulation-gdpr/
- **OpenAPI 3.0:** https://spec.openapis.org/oas/v3.0.0
- **SQLAlchemy ORM:** https://docs.sqlalchemy.org/
- **FastAPI:** https://fastapi.tiangolo.com/
- **PostgreSQL Security:** https://www.postgresql.org/docs/current/sql-security.html

