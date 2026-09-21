# Threat Model

This document identifies threat actors, attack surfaces, and mitigations for Solaura's data handling.

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

## Attack Surface Summary

| Surface | Sensitivity | Phase 1 Protection | Residual Risk |
|---------|-------------|-------------------|---------------|
| Database at rest | High | Envelope encryption | KEK+DB holder can decrypt |
| Database backups | High | Encrypted blobs | Same as above |
| Vercel env/logs | Medium | Secrets isolation | Platform compromise |
| LLM inference | High | TLS only | **Ephemeral plaintext exposure** |
| Client session | High | RLS, secure tokens | Session hijacking |
| Network transit | Medium | TLS 1.3 | Minimal |

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

## Out of Scope

- Physical security of user devices
- Social engineering attacks on individual users
- Nation-state level adversaries with TLS interception capabilities
- Attacks requiring Supabase or Vercel platform-level compromise (we trust these platforms)
