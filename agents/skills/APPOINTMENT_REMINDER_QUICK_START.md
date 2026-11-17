# Quick Start: Appointment Reminder Processor Skill

**TL;DR:** Automated daily appointment reminders to reduce no-show rate from 22% → 15%  
**Status:** ✅ Ready for deployment  
**Last Updated:** 2025

---

## 🎯 What This Skill Does

Processes upcoming patient appointments across all 12 HealthCore clinics (USA + UK) and automatically sends multimodal reminders (Email, SMS, Phone Call) to patients who have opted in.

**Key Numbers:**
- **Daily Volume:** 2,000-3,000 reminders
- **Coverage:** 95% of eligible appointments (≥48 hours ahead)
- **Delivery Success:** ≥97% across all channels
- **Business Impact:** Save ~$500K/year by reducing no-show rate from 22% → 15%

---

## 📥 Inputs Required

1. **Appointment Data** → Unified Clinical API
   - Scheduled appointments 48-72 hours in advance
   - Patient contact info (email, phone, language preference, timezone)
   - Appointment details (date, time, clinician, location, prep instructions)

2. **Patient Consent** → Compliance Database
   - HIPAA signed (USA patients)
   - GDPR consent active (UK patients)
   - Appointment reminders opt-in = TRUE
   - No "do not contact" flag

3. **Reminder Templates** → Configuration File
   - Bilingual (English + Spanish)
   - Multi-channel (Email, SMS, Phone)
   - Placeholders for personalization

---

## ⚙️ How It Works (6 Steps)

| Step | Action | Compliance |
|------|--------|-----------|
| 1️⃣ | Retrieve appointments ≥48 hours ahead | Validate API response integrity |
| 2️⃣ | Check patient HIPAA/GDPR consent | 100% USA = HIPAA signed; 100% UK = GDPR active |
| 3️⃣ | Validate patient preferences & contact | Email/Phone complete; language = en/es |
| 4️⃣ | Load template & personalize message | Replace placeholders (name, date, time, clinic) |
| 5️⃣ | Deliver via preferred channels | Email + SMS (3x retry with exponential backoff) |
| 6️⃣ | Log results & flag anomalies | Audit trail: ID, patient, channel, status, timestamp |

---

## ✅ Success Criteria (At-a-Glance)

| Category | Criterion | ✓ Pass Condition |
|----------|-----------|-----------------|
| **Functional** | Appointment retrieval | ≥95% of clinics responding with valid data |
| | Consent validation | 100% HIPAA USA, 100% GDPR UK |
| | Bilingual support | EN/ES templates for all 3 channels |
| | Message personalization | No unreplaced placeholders; locale-formatted dates/times |
| | Multi-channel delivery | Send to ALL preferred channels (never default to 1) |
| | SMS enforcement | ≤160 characters; critical info never truncated |
| | Timezone handling | Appointment times in patient's timezone |
| | Opt-out respect | 0% reminders to `opt_in=FALSE` patients |
| | Retry logic | 3x retries with exponential backoff (2h, 4h, 8h) |
| | Audit completeness | Every reminder logged with ID, channel, timestamp, status |
| **Performance** | Processing speed | <5 minutes for 1,000 appointments |
| | Delivery success | ≥97% (Delivered + OK Deferred) / Total |
| | Compliance check | <100ms per patient consent lookup |
| | Daily throughput | 2,000-3,000 reminders/day |
| | No-show reduction | 22% → 15% within 12 months |
| **Security** | Data residency | USA data in USA only; UK data in UK only |
| | Encryption | TLS 1.2+ for all APIs; encrypted delivery |
| | PII minimization | Reminders: first name + date/time + clinic only (NO SSN, MRN, diagnosis) |
| | Access control | Only this skill reads patient contact data |
| | Error logging | Error messages DO NOT contain PII |

---

## 📊 Input Example

```json
{
  "appointment_id": "appt-2025-01-15-00847",
  "patient_id": "pat-98765432",
  "clinic_id": "AUS-001",
  "region": "USA",
  "scheduled_datetime": "2025-01-16T14:30:00-06:00",
  "clinician_name": "Dr. Maria Gonzalez",
  "patient_contact": {
    "email": "john.smith@email.com",
    "phone_primary": "+14155552671",
    "language_preference": "en",
    "timezone": "America/Chicago"
  },
  "patient_preferences": {
    "reminder_opt_in": true,
    "reminder_channels": ["EMAIL", "SMS"],
    "do_not_contact_flag": false
  }
}
```

---

## 📤 Output Example

```json
{
  "job_id": "job-reminder-2025-01-15-batch1",
  "timestamp": "2025-01-15T09:00:00Z",
  "total_appointments_processed": 847,
  "summary": {
    "reminders_sent": 723,
    "skipped_no_consent": 92,
    "delivery_success_rate": 0.978
  },
  "compliance_audit": {
    "hipaa_violations": 0,
    "gdpr_violations": 0
  }
}
```

---

## 🚀 How to Use This Skill

### For Agent Developers:

1. **Read full spec:** [`agents/skills/APPOINTMENT_REMINDER_PROCESSOR.md`](../agents/skills/APPOINTMENT_REMINDER_PROCESSOR.md) (20 min)
2. **Review acceptance criteria** (10 min)
3. **Study input/output examples** (5 min)
4. **Plan implementation:**
   - API integration (Unified Clinical API + Consent DB)
   - Message templating & personalization
   - Delivery queuing (Email, SMS)
   - Audit logging & compliance checks
5. **Test before deployment:**
   - Dry-run mode (no actual sends)
   - HIPAA/GDPR consent validation
   - Bilingual template rendering
   - Timezone formatting
6. **Monitor first 7 days:**
   - Delivery success rate trend
   - Compliance violations (should be 0)
   - Regional parity (USA vs UK performance)

### For Infrastructure/DevOps:

1. **Setup reminder delivery services:**
   - Email: SendGrid or AWS SES (TLS 1.2+)
   - SMS: Twilio (E.164 phone format validation)
   - Phone: Optional IVR service (depends on clinic preferences)

2. **Configure scheduling:**
   - Daily cron: 7:00 AM UTC (covers USA early morning + UK afternoon)
   - Retry queue: Handle 3x retries with exponential backoff

3. **Enable monitoring:**
   - Alert if delivery success rate < 95%
   - Alert if any compliance violations detected
   - Alert if >10% compliance skips for any clinic

4. **Setup audit logging:**
   - Every reminder: ID, appointment_id, patient_id (no names), channel, status, timestamp
   - Daily: Summary by region, clinic, channel

---

## 🛑 Critical Compliance Rules

**Non-Negotiable (0% violation tolerance):**

✅ **USA Patients:**
- Verify HIPAA signed before sending ANY reminder
- Process USA data only in USA region
- No cross-border transfer to UK region

✅ **UK Patients:**
- Verify GDPR consent active before sending ANY reminder
- Consent must be within 24-month window
- Process UK data only in UK region
- No cross-border transfer to USA region

✅ **Reminder Content:**
- Include ONLY: patient first name, appointment date/time, clinic name, clinician specialty
- DO NOT include: SSN, MRN, diagnosis, medical history, insurance details
- PII in error logs: Reference patient_id only (not name, email, phone)

✅ **Patient Preferences:**
- Respect reminder_opt_in = FALSE (0% reminders to opted-out patients)
- Respect reminder_channels preference (never default to only EMAIL)
- Support language preference (en/es only; fallback to en if unsupported)

---

## 📞 Support & Escalation

**Question:** Where is the full skill specification?
→ See [`agents/skills/APPOINTMENT_REMINDER_PROCESSOR.md`](../agents/skills/APPOINTMENT_REMINDER_PROCESSOR.md)

**Question:** How do I invoke this skill?
→ See "Activation & Usage" section in full spec; use APScheduler or cron job for daily 7 AM UTC

**Question:** What if delivery fails for a patient?
→ Retry 3x with exponential backoff; flag for manual follow-up after 3rd failure

**Question:** How do I know if compliance rules are being followed?
→ Monitor compliance_audit section of daily output report; alert on any violations

**Bug or Issue?**
→ File GitHub issue with tag `skill-appointment-reminders` or `skill-bug`

**New Requirement?**
→ File GitHub issue with tag `skill-proposal` or `skill-enhancement`

---

## 🎓 Learning Path

**Time Investment:** ~90 minutes

1. **Overview** (10 min) — This quick-start guide
2. **Full Spec** (30 min) — Read [`APPOINTMENT_REMINDER_PROCESSOR.md`](../agents/skills/APPOINTMENT_REMINDER_PROCESSOR.md)
3. **Business Context** (15 min) — Review `projectbrief.md` sections on no-show problem
4. **Compliance** (15 min) — Review `techContext.md` sections on HIPAA & GDPR
5. **Implementation Planning** (20 min) — Outline your approach
6. **Code Review** (varies) — Share implementation with Tech Lead for feedback

---

## 📋 Pre-Deployment Checklist

Before going live with this skill:

- [ ] API integration complete (Unified Clinical API accessible)
- [ ] Consent DB integration complete (HIPAA/GDPR checks working)
- [ ] Email service configured (SendGrid/AWS SES with TLS 1.2+)
- [ ] SMS service configured (Twilio with E.164 validation)
- [ ] Template files loaded (bilingual EN/ES, 3 channels)
- [ ] Dry-run testing passed (100 test appointments, no actual sends)
- [ ] HIPAA/GDPR compliance tests passed (sample test patients)
- [ ] Bilingual rendering validated (EN/ES dates, times, clinic names)
- [ ] Timezone formatting validated (5+ timezones tested)
- [ ] Audit logging working (every reminder logged, PII-safe)
- [ ] Monitoring configured (success rate, compliance violations, regional parity)
- [ ] Approval from Tech Lead (CTO) obtained
- [ ] Team notified of daily execution (7 AM UTC)
- [ ] Runbook documented for incident response (delivery failures, compliance alerts)

---

**Skill Version:** 1.0  
**Status:** ✅ Ready for Implementation  
**Last Updated:** 2025  
**Next Review:** 2025 Q2 or upon business requirement change
