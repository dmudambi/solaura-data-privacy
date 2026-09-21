# Legal Notes

Entity information, regulatory posture, and compliance status for Solaura.

## Corporate Entity

| Field | Value |
|-------|-------|
| **Legal Name** | Solaura Technologies, Inc. |
| **Jurisdiction** | Delaware, USA |
| **Entity Type** | C-Corporation |
| **Primary Operations** | United States, with users globally |

## Data Classification

### What Solaura Collects

| Data Type | Classification | Notes |
|-----------|---------------|-------|
| Session transcripts | Sensitive personal data | User-generated therapeutic/reflective content |
| Emotional insights | Sensitive personal data | Inferred emotional states |
| Session debriefs | Sensitive personal data | AI-generated analysis of sessions |
| Goals and reflections | Sensitive personal data | Personal development content |
| Email address | Personal data | Required for authentication |
| Usage metadata | Operational data | Timestamps, session duration |

### Sensitivity Acknowledgment

**Session audio, transcripts, and AI-generated insights are sensitive personal data.** This content may contain:
- Personal reflections and feelings
- Discussion of relationships
- Mental health related content
- Life events and circumstances

We treat this data with the highest level of protection available in our current architecture.

## Regulatory Posture

### HIPAA (United States)

| Status | Details |
|--------|---------|
| **Claim** | ❌ We do NOT claim HIPAA compliance |
| **Reason** | Solaura is not a healthcare provider, health plan, or healthcare clearinghouse |
| **Implication** | Do not use Solaura for protected health information (PHI) in clinical contexts |

**Important**: Solaura is a personal reflection and growth tool, not a healthcare or therapy service. Users should not consider Solaura a substitute for professional mental health care.

### GDPR (European Union)

| Aspect | Status |
|--------|--------|
| Lawful basis | Consent (user agreement to Terms of Service) |
| Data subject rights | Supported (access, deletion, portability) |
| Data transfers | US-based processing; Standard Contractual Clauses where required |
| DPO | Not appointed (threshold not met) |

### CCPA/CPRA (California)

| Aspect | Status |
|--------|--------|
| Consumer rights | Supported (know, delete, opt-out) |
| Sale of data | We do not sell personal information |
| Sensitive PI | Recognized and protected |

### India DPDP Act (Digital Personal Data Protection)

| Aspect | Status |
|--------|--------|
| Awareness | ✅ DPDP-aware design |
| Data fiduciary | Solaura Technologies, Inc. |
| Significant data fiduciary | To be determined based on user volume |
| Consent | Obtained through Terms of Service |
| Purpose limitation | Data used only for stated service purposes |
| Data localization | Monitoring requirements; currently US-processed |

**Note**: We are monitoring DPDP implementation and will adapt our practices as regulations are finalized. Users in India are subject to the same data protections as all users.

### Other Jurisdictions

For users in jurisdictions with specific data protection requirements not listed above, we apply our baseline protections which meet or exceed common requirements:
- Consent-based processing
- Purpose limitation
- Data minimization
- Security measures
- User access and deletion rights

## What We Explicitly Do NOT Claim

To maintain honesty and avoid overclaiming:

| Claim | Our Position |
|-------|-------------|
| "HIPAA Compliant" | ❌ Not claimed |
| "SOC 2 Certified" | ❌ Not claimed (infrastructure providers are) |
| "End-to-End Encrypted" | ❌ Not true during Phase 1 for AI features |
| "We never see your data" | ❌ Not true — server processes data for LLM calls |
| "Zero-knowledge architecture" | ❌ Not yet — roadmap for Phase 2 |
| "Therapy or healthcare" | ❌ Solaura is NOT a healthcare service |

## Data Processor Relationships

### Infrastructure Providers

| Provider | Role | Data Access | Compliance |
|----------|------|-------------|------------|
| Supabase | Database hosting | Encrypted blobs at rest | SOC 2 Type II |
| Vercel | Application hosting | Environment variables, function execution | SOC 2 Type II |
| OpenAI | LLM inference | Plaintext during inference | SOC 2 Type II |

### Data Processing Agreements

- [ ] Supabase DPA: In place
- [ ] Vercel DPA: In place
- [ ] OpenAI DPA: In place

## User Rights

All users have the following rights:

| Right | How to Exercise |
|-------|-----------------|
| **Access** | Export your data through app settings |
| **Deletion** | Delete individual sessions or full account |
| **Portability** | Export in standard format |
| **Rectification** | Edit your sessions and profile |
| **Withdraw consent** | Delete account |

### Exercising Rights

Users can exercise their rights by:
1. Using in-app controls (preferred)
2. Contacting support at [support email]
3. For formal requests: [privacy email]

Response timeframe: 30 days (as required by GDPR)

## Data Breach Notification

In the event of a data breach involving user content:

| Jurisdiction | Notification Timeframe |
|--------------|----------------------|
| GDPR (EU) | 72 hours to supervisory authority |
| California | "Most expedient time possible" |
| Other | Per applicable law |

Users will be notified directly if their data is compromised, with:
- Description of the breach
- Data types affected
- Remediation steps taken
- Recommended user actions

## Terms of Service

Use of Solaura is governed by our Terms of Service, which include:
- Description of data processing
- User consent to AI-powered features
- Limitation of liability
- Dispute resolution

## Contact

For privacy-related inquiries:
- General: [Contact through app]
- Formal requests: [Privacy email]
- Security concerns: [Security email]

## Document History

| Date | Change |
|------|--------|
| [Initial] | Initial version |

---

*This document is for transparency purposes. It does not constitute legal advice. Consult a qualified attorney for legal guidance.*
