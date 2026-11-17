# Agent Skill: Appointment Reminder Processor

**Version:** 1.0  
**Author:** Copilot  
**Last Updated:** 2025  
**Status:** Active  
**Aligns with:** Project Brief Section 3 (No-Show Rate Reduction)

---

## 📌 Executive Summary

**Single Objective:** Process upcoming patient appointments and generate multimodal reminders (email, SMS, phone) to reduce the 22% no-show rate that costs HealthCore ~$1.8M annually.

**Key Metric:** Successfully process ≥95% of scheduled appointments within 48 hours before appointment time, with <2% reminder delivery failures.

---

## 🎯 Objective & Scope

### Primary Goal
Automate daily appointment reminder workflow to:
- Retrieve next-day and next-3-days appointments from both USA (EHR System A) and UK (EHR System B) clinics
- Validate patient contact preferences and consent status
- Generate context-aware reminder messages
- Trigger multimodal delivery (email, SMS, voice call)
- Log outcomes for audit trail and analytics

### Scope Boundaries

**IN SCOPE:**
- Appointments scheduled ≥48 hours in advance
- Patients who have opted into reminders (consent flag = TRUE)
- All 12 clinics (9 USA + 3 UK)
- Both EHR systems (System A/B unified via API)
- Reminder formats: Email, SMS, Phone Call
- Failure handling and retry logic

**OUT OF SCOPE:**
- Appointments within 24 hours (handled by clinical staff manually)
- Patients with opt-out flag = TRUE
- Rescheduling or cancellation requests (escalate to clinical staff)
- Payment collection reminders (separate Finance workflow)
- Post-appointment follow-ups (separate Follow-up Processor skill)

### Business Context
- **Problem:** 22% no-show rate (vs 8-10% industry average) = $1.8M/year lost revenue
- **Target Reduction:** 22% → 15% within 12 months (cost savings: ~$500K/year)
- **Success Metric:** No-show rate decrease + reminder delivery success rate >97%
- **Compliance:** HIPAA (USA) + UK GDPR + HealthCore data residency requirements

---

## 📥 Inputs (Documented & Typed)

### 1. **Appointment Data Source**
```
INPUT TYPE: SQL Query Result / JSON API Response
SOURCE: Unified Clinical API Endpoint
ENDPOINT: /api/v2/appointments/upcoming
QUERY_PARAMS:
  - date_from: YYYY-MM-DD (today)
  - date_to: YYYY-MM-DD (today + 3 days)
  - status: "SCHEDULED" or "CONFIRMED"
  - region: ["USA", "UK"] or specific clinic_id

SCHEMA:
{
  "appointment_id": "string (UUID)",
  "patient_id": "string (UUID)",
  "clinic_id": "string (clinic code, e.g., 'AUS-001')",
  "region": "string ('USA' or 'UK')",
  "scheduled_datetime": "ISO 8601 datetime",
  "appointment_type": "string (e.g., 'PRIMARY_CARE', 'SPECIALIST', 'CHRONIC_DISEASE')",
  "clinician_name": "string",
  "clinician_specialty": "string",
  "room_location": "string",
  "status": "string ('SCHEDULED', 'CONFIRMED', 'RESCHEDULED')",
  
  "patient_contact": {
    "email": "string or null",
    "phone_primary": "string (E.164 format) or null",
    "phone_secondary": "string (E.164 format) or null",
    "language_preference": "string ('en', 'es')",
    "timezone": "string (e.g., 'America/Chicago', 'Europe/London')"
  },
  
  "patient_preferences": {
    "reminder_opt_in": "boolean",
    "reminder_channels": "array of ['EMAIL', 'SMS', 'PHONE_CALL']",
    "reminder_time_before": "integer (hours, e.g., 24, 48)",
    "do_not_contact_flag": "boolean"
  },
  
  "clinical_notes": {
    "requires_preparation": "boolean",
    "preparation_instructions": "string or null",
    "insurance_verification_needed": "boolean",
    "last_visit_date": "ISO 8601 date or null"
  }
}

VALIDATION RULES:
✓ appointment_id must be non-null UUID
✓ scheduled_datetime must be ≥48 hours from now
✓ patient_id must exist and be active (not deleted/archived)
✓ clinic_id must map to valid clinic in system
✓ at least one contact channel (email or phone) must be non-null
✓ language_preference must be supported language ('en', 'es')
```

### 2. **Patient Consent & Compliance Data**
```
INPUT TYPE: Lookup Query
SOURCE: Compliance & Consent Database
ENDPOINT: /api/v2/patient/{patient_id}/consent

SCHEMA:
{
  "patient_id": "string (UUID)",
  "region": "string ('USA' or 'UK')",
  "gdpr_consent": "boolean (UK requirement)",
  "hipaa_signed": "boolean (USA requirement)",
  "marketing_emails_opt_in": "boolean",
  "appointment_reminders_opt_in": "boolean",
  "last_consent_update": "ISO 8601 datetime",
  "data_processing_agreement": "string (reference or timestamp)"
}

VALIDATION RULES:
✓ For USA patients: hipaa_signed must be true
✓ For UK patients: gdpr_consent must be true
✓ appointment_reminders_opt_in must be true
✓ last_consent_update must be within 24 months
✗ If any consent check fails → SKIP reminder for this patient
```

### 3. **Reminder Template Configuration**
```
INPUT TYPE: Configuration Document
SOURCE: Template Library
PATH: /agents/config/reminder-templates.json

SCHEMA (per language/channel):
{
  "template_id": "string (e.g., 'APPT_EMAIL_EN_v2')",
  "language": "string ('en' or 'es')",
  "channel": "string ('EMAIL', 'SMS', 'PHONE_CALL')",
  "subject": "string (email only)",
  "body_template": "string with placeholders {patient_name}, {clinician_name}, {date}, {time}, {clinic_name}, {prep_instructions}",
  "include_map_link": "boolean",
  "include_cancel_link": "boolean",
  "include_insurance_note": "boolean",
  "tone": "string ('friendly', 'clinical', 'formal')",
  "character_limit": "integer (SMS: max 160, Email: unlimited, Phone: for TTS length)"
}

EXAMPLE (Email EN):
{
  "template_id": "APPT_EMAIL_EN_v2",
  "language": "en",
  "channel": "EMAIL",
  "subject": "Appointment Reminder: {{clinician_name}} on {{date}}",
  "body_template": "Hi {{patient_name}},\n\nThis is a friendly reminder that you have an appointment with {{clinician_name}} ({{specialty}}) tomorrow at {{time}} at {{clinic_name}}.\n\nLocation: {{room_location}}\n\n{{prep_instructions}}\n\n[Cancel or Reschedule]({{cancel_link}})\n\nQuestions? Call us at {{clinic_phone}}"
}
```

---

## 🔄 Processing Workflow

### Step 1: Retrieve Appointment Data
1. Query Unified Clinical API for appointments scheduled 48-72 hours ahead
2. Validate response integrity (non-empty, schema compliance)
3. Filter by clinic region (USA vs UK)
4. **Fail-Safe:** Log any API errors to alert channel; skip batch if >10% of clinics unreachable

### Step 2: Validate Compliance & Consent
For each appointment:
1. Fetch patient consent record (HIPAA for USA, GDPR for UK)
2. Check `appointment_reminders_opt_in` = TRUE
3. Check `do_not_contact_flag` = FALSE
4. If GDPR-regulated patient: verify consent within 24-month window
5. **Action:** Skip patients failing compliance checks; log reason

### Step 3: Select Reminder Channels
For each compliant patient:
1. Read `patient_preferences.reminder_channels` array
2. For each preferred channel (EMAIL, SMS, PHONE_CALL):
   - Verify contact data is complete (email format, phone E.164)
   - Select appropriate template (match language + channel)
   - Check template exists in config; fallback to default if missing

### Step 4: Personalize Reminder Message
1. Load template by language + channel
2. Replace placeholders:
   - `{{patient_name}}` → first name from patient record
   - `{{clinician_name}}` → clinician full name
   - `{{date}}` → format per locale (e.g., "Tuesday, Jan 15" for EN, "Martes, 15 de enero" for ES)
   - `{{time}}` → format per timezone + locale
   - `{{clinic_name}}` → clinic display name
   - `{{room_location}}` → specific location or "Main Clinic"
   - `{{prep_instructions}}` → from clinical_notes (if present)
   - `{{cancel_link}}` → unique secure link to reschedule
   - `{{clinic_phone}}` → clinic direct phone (not toll-free, if available)
3. For SMS: enforce character limit (max 160 chars; truncate gracefully)
4. For PHONE_CALL: format for text-to-speech (clear dates, spell names)

### Step 5: Deliver Reminder
1. **Email:** Queue to email service (SendGrid, AWS SES) with retry policy (3x, exponential backoff)
2. **SMS:** Queue to SMS provider (Twilio) with delivery confirmation
3. **PHONE_CALL:** Queue to IVR/voice service (optional; depends on regional preferences)
4. Record delivery timestamp and status in audit log
5. **Failure Handling:**
   - Soft failure (delivery in progress): retry after 2 hours
   - Hard failure (invalid contact): flag for manual follow-up; notify clinic manager
   - API error: log error; re-queue batch job within 4 hours

### Step 6: Log & Analytics
1. For each reminder successfully sent:
   - Log: `{appointment_id, patient_id, channel, template_id, timestamp, delivery_status, region, clinic_id}`
2. Aggregate daily metrics:
   - Total appointments processed
   - Reminders sent by channel (email %, SMS %, phone %)
   - Delivery success rate
   - Compliance skip reasons (top 5)
   - Regional comparison (USA vs UK)
3. Flag anomalies:
   - >5% delivery failures for any clinic
   - >10% compliance skips for any clinic
   - Any repeat failures for same patient/channel

---

## ✅ Acceptance Criteria (Explicit & Verifiable)

### Functional Criteria

| ID | Criterion | Verification Method | Pass Condition |
|---|-----------|---------------------|-----------------|
| **F1** | Retrieve appointments ≥48 hours ahead | Query Unified API; count results | ≥95% of clinics responding with valid data |
| **F2** | Validate HIPAA/GDPR consent | Audit log review | 100% of USA patients have HIPAA signed; 100% of UK patients have GDPR consent |
| **F3** | Support bilingual reminders (EN/ES) | Template coverage check | Templates exist for both languages for all 3 channels (Email, SMS, Phone) |
| **F4** | Personalize messages accurately | Sample message audit | No placeholder `{{variable}}` remains unreplaced; names/dates formatted per locale |
| **F5** | Deliver via multiple channels | Delivery log inspection | For patients with multi-channel preference, reminders sent to ALL preferred channels |
| **F6** | SMS character limit enforcement | SMS content audit | 100% of SMS messages ≤160 characters; no truncation of critical info (date, time) |
| **F7** | Respect timezone formatting | Delivery log inspection | Appointment times displayed in patient's timezone; timezone offset shown where applicable |
| **F8** | Handle opt-outs correctly | Compliance audit | 0% of reminders sent to `reminder_opt_in = FALSE` patients |
| **F9** | Retry failed deliveries | Delivery log | Failed deliveries retried ≤3x; exponential backoff (2h, 4h, 8h intervals) |
| **F10** | Audit trail completeness | Log inspection | Every reminder logged with: ID, patient, channel, template, timestamp, status, error (if failed) |

### Performance Criteria

| ID | Metric | Target | Method |
|---|--------|--------|--------|
| **P1** | Processing latency | <5 min for 1,000 appointments | Measure end-to-end time from API query to last delivery queue |
| **P2** | Delivery success rate | ≥97% | (Delivered + Deferred OK) / Total attempted |
| **P3** | Compliance check speed | <100ms per patient | Measure consent lookup latency |
| **P4** | Daily throughput | 2,000-3,000 reminders/day | Baseline: 12 clinics × avg 200 appts/clinic/week = ~350 appts/day × 3-day window |
| **P5** | No-show rate impact | Reduce from 22% → 15% | Track over 12 months; measure via clinic appointments-vs-shows ratio |

### Security & Compliance Criteria

| ID | Criterion | Verification | Compliance |
|---|-----------|---------------|------------|
| **S1** | HIPAA data residency (USA) | Audit log review | All USA patient data processed in USA region only; no cross-border transfers |
| **S2** | UK GDPR compliance | Consent audit + data logs | UK patient data not transferred to USA; consent active & timestamped |
| **S3** | Encryption in transit | Network log inspection | All API calls to reminder services use TLS 1.2+; email/SMS encrypted end-to-end |
| **S4** | PII minimization | Message content audit | Reminders contain only: first name, appointment date/time, clinic name, clinician specialty (NOT SSN, MRN, diagnosis) |
| **S5** | Access control | Agent capability audit | Only this skill (Appointment Reminder Processor) can query patient contact data; no other agent can |
| **S6** | Error logging sanitization | Error messages do NOT contain PII; failed delivery logs reference patient_id only (not name/email) | Log inspection |

### Business Criteria

| ID | Metric | Target | Method |
|---|--------|--------|--------|
| **B1** | Appointment reminder coverage | ≥90% of eligible appointments | (Reminders sent) / (appointments ≥48h away AND reminder_opt_in=TRUE) |
| **B2** | Channel preference compliance | 100% | If patient prefers [EMAIL, SMS], send to both; never default to only EMAIL |
| **B3** | Regional parity | No >5% variance | Email delivery success rate USA vs UK ≤5 percentage points difference |
| **B4** | No-show attribution | Reduce to 15% within 12mo | Track appointment show/no-show status; correlate with reminder delivery |
| **B5** | Clinician feedback score | ≥8/10 | Manual survey post-implementation: "reminders are helpful & professional" |

---

## 📊 Input/Output Example

### Example Input Appointment
```json
{
  "appointment_id": "appt-2025-01-15-00847",
  "patient_id": "pat-98765432",
  "clinic_id": "AUS-001",
  "region": "USA",
  "scheduled_datetime": "2025-01-16T14:30:00-06:00",
  "appointment_type": "PRIMARY_CARE",
  "clinician_name": "Dr. Maria Gonzalez",
  "clinician_specialty": "Primary Care",
  "room_location": "Building B, Room 204",
  
  "patient_contact": {
    "email": "john.smith@email.com",
    "phone_primary": "+14155552671",
    "language_preference": "en",
    "timezone": "America/Chicago"
  },
  
  "patient_preferences": {
    "reminder_opt_in": true,
    "reminder_channels": ["EMAIL", "SMS"],
    "reminder_time_before": 24,
    "do_not_contact_flag": false
  },
  
  "clinical_notes": {
    "requires_preparation": true,
    "preparation_instructions": "Please bring recent lab results and insurance card. Fasting recommended if labs needed.",
    "insurance_verification_needed": true,
    "last_visit_date": "2024-10-12"
  }
}
```

### Example Output - Reminders Sent
```json
{
  "job_id": "job-reminder-2025-01-15-batch1",
  "timestamp": "2025-01-15T09:00:00Z",
  "region": "USA",
  "total_appointments_processed": 847,
  "summary": {
    "reminders_sent": 723,
    "skipped_no_consent": 92,
    "skipped_no_contact": 24,
    "skipped_opted_out": 8,
    "delivery_success_rate": 0.978,
    "avg_processing_time_ms": 2847
  },
  "reminders": [
    {
      "reminder_id": "rem-2025-01-15-00847-email",
      "appointment_id": "appt-2025-01-15-00847",
      "patient_id": "pat-98765432",
      "channel": "EMAIL",
      "template_id": "APPT_EMAIL_EN_v2",
      "sent_datetime": "2025-01-15T09:00:34Z",
      "status": "DELIVERED",
      "delivery_provider": "SendGrid",
      "message_id": "sg-msg-9999888877776",
      "clinic_id": "AUS-001"
    },
    {
      "reminder_id": "rem-2025-01-15-00847-sms",
      "appointment_id": "appt-2025-01-15-00847",
      "patient_id": "pat-98765432",
      "channel": "SMS",
      "template_id": "APPT_SMS_EN_v2",
      "sent_datetime": "2025-01-15T09:00:45Z",
      "status": "DELIVERED",
      "delivery_provider": "Twilio",
      "message_id": "SM1234567890abcdef",
      "clinic_id": "AUS-001"
    }
  ],
  "failed_reminders": [
    {
      "reminder_id": "rem-2025-01-15-01203-email",
      "patient_id": "pat-11111111",
      "appointment_id": "appt-2025-01-15-01203",
      "channel": "EMAIL",
      "status": "FAILED",
      "error_reason": "INVALID_EMAIL_FORMAT",
      "retry_schedule": "2025-01-15T11:00:00Z",
      "escalation_flag": true
    }
  ],
  "compliance_audit": {
    "hipaa_violations": 0,
    "gdpr_violations": 0,
    "consent_expiry_warnings": 3
  }
}
```

---

## 🚀 Activation & Usage

### For Agents Using This Skill

**When to invoke:**
- Daily schedule: 7:00 AM UTC (covers both USA waking hours and UK afternoon)
- Triggered by: APScheduler (Python) or equivalent cron job
- Invocation command:
  ```
  invoke_agent_skill(
    skill_name="APPOINTMENT_REMINDER_PROCESSOR",
    parameters={
      "date_from": "today",
      "date_to": "today+3",
      "regions": ["USA", "UK"],
      "dry_run": false
    }
  )
  ```

**Expected return value:**
- JSON object with summary (reminders_sent, delivery_success_rate, errors)
- Detailed delivery log for audit
- Flagged anomalies requiring human review

### For Developers Integrating This Skill

1. **Install dependency:** Ensure agent has access to Unified Clinical API credentials
2. **Configure templates:** Load reminder-templates.json into agent working directory
3. **Set permissions:** Agent must have READ access to patient consent DB, WRITE access to delivery queue
4. **Test coverage:**
   - Dry-run mode: Process sample appointments without sending reminders
   - Compliance test: Verify HIPAA/GDPR checks with known test patients
   - Bilingual test: Validate EN/ES templates render correctly
5. **Monitor:** Set up alerts for >3% delivery failures or any compliance violations

---

## 📋 Summary

| Aspect | Detail |
|--------|--------|
| **Objective** | Process 2,000-3,000 appointment reminders daily to reduce 22% no-show rate |
| **Input** | Scheduled appointments, patient consent, reminder templates, contact preferences |
| **Output** | Delivery status report, audit trail, compliance validation, analytics |
| **Success Rate Target** | ≥97% delivery success; ≥90% coverage; 0% compliance violations |
| **Compliance** | HIPAA (USA) + UK GDPR; region-specific data residency; PII minimization |
| **Business Impact** | Projected 22% → 15% no-show reduction = $500K/year cost savings |
| **Recurrence** | Daily, automated 7:00 AM UTC |

---

**Last Review:** 2025 | **Next Review:** Every 3 months or upon business requirement change
