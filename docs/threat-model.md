# Threat Model

This document sets out threat actors, attack surfaces, and mitigations for Solaura's data handling.

*Updated 2026-09-26: added threats specific to the de-identified research corpus (sections 7–9). Threats 1–6 cover live session data, which stays encrypted at rest.*

## Threat Actors

### 1. Malicious Operators / Insiders

**Description**: Employees or contractors with legitimate access to infrastructure components.

**Access vectors**:
- Supabase dashboard access (service role key)
- Direct database queries via SQL editor
- Vercel deployment logs and environment variables
- KMS/secrets management console

**Phase 1 mitigation**:
- Envelope encryption separates data (DB) from keys (KMS/Vercel env)
- Requires compromise of BOTH systems to decrypt
- Audit logging on Supabase and Vercel dashboards
- Principle of least privilege for team access

**Residual risk**: An operator with access to both KMS and database can decrypt all sensitive fields.

### 2. Database Breach / Backup Exfiltration

**Description**: Unauthorized access to database contents through SQL injection, backup theft, or Supabase platform compromise.

**Attack vectors**:
- SQL injection (if present in application code)
- Stolen database backup files
- Supabase platform-level breach
- Compromised service role key

**Phase 1 mitigation**:
- Sensitive fields encrypted at rest with AES-256-GCM
- Encryption keys NOT stored in database
- Row Level Security (RLS) policies enforce user isolation
- Parameterized queries prevent SQL injection

**Residual risk**: Encrypted blobs are recoverable but not decryptable without KMS access. Metadata (timestamps, session IDs, duration) remains plaintext.

### 3. Vercel Logs / Environment Variable Exposure

**Description**: Compromise of deployment platform revealing secrets or logged data.

**Attack vectors**:
- Vercel account compromise
- Log aggregation service breach
- Environment variable exposure in build logs
- Function invocation logs capturing request/response data

**Phase 1 mitigation**:
- Sensitive data not logged in application code
- Environment variables marked as sensitive in Vercel
- Build logs do not print secret values
- Runtime logs redact user content

**Residual risk**: Misconfigured logging could expose plaintext. Vercel platform-level access exposes KEK.

### 4. LLM Provider Exposure (OpenAI, etc.)

**Description**: Transcript plaintext sent to third-party LLM providers during debrief generation.

**Attack vectors**:
- LLM provider data retention policies
- Provider platform breach
- Man-in-the-middle on TLS (unlikely but non-zero)
- Provider employee access to inference logs

**Phase 1 mitigation**:
- TLS encryption in transit
- Use of provider APIs with data processing agreements
- No persistent storage requested on provider side (where configurable)

**Residual risk**: **Plaintext transcripts exist ephemerally at LLM provider during inference.** This is a known limitation until Phase 2 sealed inference.

### 5. Compromised Client Session

**Description**: Attacker gains access to authenticated user session.

**Attack vectors**:
- Session token theft (XSS, network sniffing)
- Device compromise (malware)
- Shoulder surfing / physical access
- Social engineering of authentication flow

**Phase 1 mitigation**:
- Secure session token handling (HttpOnly, Secure cookies)
- Short session expiry with refresh tokens
- RLS ensures users can only access their own data
- No sensitive data in localStorage

**Residual risk**: Compromised session has full access to that user's decrypted data via API. Multi-device session revocation not yet implemented.

### 6. Supply Chain / Dependency Attacks

**Description**: Malicious code introduced through npm packages or other dependencies.

**Attack vectors**:
- Compromised npm package
- Typosquatting attacks
- Build-time code injection

**Phase 1 mitigation**:
- Lockfile pinning (package-lock.json)
- Dependabot/Renovate for security updates
- Limited production dependencies

**Residual risk**: Novel supply chain attacks may not be detected immediately.

### 7. Re-identification of the Research Corpus (rolling out)

**Description**: Someone uses the content of a de-identified record to work out who it is about, for example by combining a distinctive situation with outside knowledge.

**Attack vectors**:
- Context clues that automated scrubbing missed (unusual events, rare combinations of details)
- Linking corpus records to outside data sources
- Matching the time bucket with known session activity

**Mitigations (design)**:
- Automated removal of names, phone numbers, emails, addresses, IDs, URLs, and exact dates
- Generalisation of quasi-identifiers (age to decade, location to region or removed, job to category, rare life events paraphrased)
- Residual-PII rescan. Failing records are quarantined.
- Human review of random samples
- No exact timestamps (week bucket at most)

**Residual risk**: **The corpus is de-identified, not anonymous.** Automated scrubbing can miss context clues, and a determined person with outside knowledge could still recognise someone. The mitigations lower this risk but cannot remove it.

### 8. Insider Attempts to Link Corpus Records to People (rolling out)

**Description**: An operator with access to both the live database and the corpus tries to match corpus records to specific users, sessions, or bonds.

**Mitigations (design)**:
- Separate corpus store
- No bond ID, user ID, or session ID in corpus records
- No mapping table. Which bond produced a conversation is never recorded.
- Week-level time bucket at most

**Residual risk**: An insider who can decrypt live session data could try to match text between the live store and the corpus. The generalisation and paraphrasing steps make this harder but do not rule it out. Access to live data stays protected by the Phase 1 encryption described in threats 1–2.

### 9. Misuse by a Future External Research Recipient (planned, not active)

**Description**: If Solaura licenses de-identified data to outside research organisations in future, a recipient could try to re-identify people, use the data for other purposes, or pass it on.

**Status**: **No data has been shared, licensed, or sold. No research partners exist.**

**Safeguards required before any sharing**:
- Separate opt-in (Tier 2) from both client and therapist, off by default and revocable
- Data use agreement that bans re-identification, limits use to research, bans onward transfer, and gives Solaura audit rights
- Access in a controlled environment where feasible
- Stricter de-identification and human review for each outbound batch
- Legal review under the DPDP Act
- Published list of partners

**Residual risk**: Contracts and audits limit misuse but cannot guarantee a recipient's behaviour. Data shared outside Solaura is harder to control.

## Attack Surface Summary

| Surface | Sensitivity | Phase 1 Protection | Residual Risk |
|---------|-------------|-------------------|---------------|
| Database at rest | High | Envelope encryption | KEK+DB holder can decrypt |
| Database backups | High | Encrypted blobs | Same as above |
| Vercel env/logs | Medium | Secrets isolation | Platform compromise |
| LLM inference | High | TLS only | **Ephemeral plaintext exposure** |
| Client session | High | RLS, secure tokens | Session hijacking |
| Network transit | Medium | TLS 1.3 | Minimal |
| Research corpus (rolling out) | High | De-identification, rescan and quarantine, human review, no identifiers or mapping | **Context clues may allow re-identification** |
| External sharing (planned, not active) | High | DUA, controlled access, per-batch review, DPDP legal review | Recipient misuse despite contract |

## Phase 2 Mitigations (Roadmap)

| Threat | Phase 2 Solution |
|--------|-----------------|
| Operator/DB access | Client-held keys (WebCrypto) - backend cannot decrypt |
| LLM provider exposure | Sealed inference / client-side AI processing |
| Backup exfiltration | E2E encrypted blobs meaningless without client keys |
| Session compromise | Per-device key derivation, remote wipe capability |

## Assumptions

1. TLS is correctly implemented and certificate validation is enforced
2. Supabase RLS policies are correctly configured and tested
3. Vercel platform security is maintained by Vercel
4. Users protect their own device and authentication credentials
5. LLM providers honor their stated data handling policies
6. The de-identification pipeline runs before any corpus write, and quarantined records are never added to the corpus

## Out of Scope

- Physical security of user devices
- Social engineering attacks on individual users
- Nation-state level adversaries with TLS interception capabilities
- Attacks requiring Supabase or Vercel platform-level compromise (we trust these platforms)
