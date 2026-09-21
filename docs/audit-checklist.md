# Audit Checklist

Verification steps for external security reviewers to validate Solaura's data handling claims.

## Purpose

This checklist enables independent verification of claims made in this transparency documentation. Auditors should be able to confirm implementation matches documentation.

---

## 1. Encryption at Rest

### 1.1 Schema Verification

- [ ] **Encrypted columns exist**: Verify sensitive columns store encrypted blobs, not plaintext
  ```sql
  -- Check that transcript_json contains encrypted format, not raw JSON
  SELECT id, 
         LEFT(transcript_json::text, 100) as sample,
         encryption_version
  FROM sessions 
  LIMIT 5;
  ```
  
- [ ] **Encryption version markers present**: Confirm `encryption_version` column exists and is populated
  ```sql
  SELECT encryption_version, COUNT(*) 
  FROM sessions 
  GROUP BY encryption_version;
  ```

- [ ] **No plaintext in encrypted fields**: Encrypted content should not be valid JSON/readable text
  ```sql
  -- This should return 0 rows if properly encrypted
  SELECT id FROM sessions 
  WHERE transcript_json::text LIKE '{%' 
    AND encryption_version > 0;
  ```

### 1.2 Tables to Verify

| Table | Encrypted Columns | Version Column |
|-------|------------------|----------------|
| sessions | transcript_json, emotions_json, highlights_json, goal_framework, pre_session_notes | encryption_version |
| session_debriefs | insight, focus_next, insight_edited, focus_next_edited | encryption_version |
| feedback_events | event_data | encryption_version |
| shared_items | content | encryption_version |
| bond_soul | encrypted_content | encryption_version |

### 1.3 Key Separation

- [ ] **KEK not in database**: Confirm no tables contain encryption keys
  ```sql
  -- Search for potential key storage (should return nothing)
  SELECT table_name, column_name 
  FROM information_schema.columns 
  WHERE column_name ILIKE '%key%' 
    OR column_name ILIKE '%kek%'
    OR column_name ILIKE '%secret%';
  ```

- [ ] **KEK in Vercel environment**: Verify key exists in Vercel env vars (requires Vercel access)
  - Environment variable name should be documented
  - Should be marked as "Sensitive" in Vercel dashboard

---

## 2. Row Level Security (RLS)

### 2.1 RLS Enabled

- [ ] **RLS active on sensitive tables**:
  ```sql
  SELECT tablename, rowsecurity 
  FROM pg_tables 
  WHERE schemaname = 'public' 
    AND tablename IN ('sessions', 'session_debriefs', 'feedback_events', 'shared_items', 'bond_soul');
  ```
  All should show `rowsecurity = true`

### 2.2 Policy Verification

- [ ] **User isolation policies exist**:
  ```sql
  SELECT tablename, policyname, permissive, roles, cmd, qual 
  FROM pg_policies 
  WHERE schemaname = 'public';
  ```
  
- [ ] **Policies reference auth.uid()**: Confirm policies restrict access to `auth.uid() = user_id`

### 2.3 Cross-User Access Test

- [ ] **Cannot read other users' data**: As authenticated user A, attempt to read user B's sessions
  ```sql
  -- Should return 0 rows for sessions belonging to other users
  SELECT * FROM sessions WHERE user_id != auth.uid();
  ```

---

## 3. Logging Verification

### 3.1 No Plaintext in Logs

- [ ] **Vercel function logs clean**: Review recent function invocation logs for:
  - No transcript content
  - No decrypted session data
  - No full request bodies containing user content

- [ ] **Search for sensitive patterns**:
  ```bash
  # In Vercel logs, search for patterns that shouldn't appear
  # These patterns indicate accidental plaintext logging
  grep -i "transcript" logs/
  grep -i "insight" logs/
  grep -i "emotion" logs/
  ```

### 3.2 Structured Logging Check

- [ ] **Log statements reviewed**: In codebase, verify `console.log`, `logger.*` calls don't include:
  - Decrypted content variables
  - Full request/response bodies for session endpoints
  - Unredacted error messages containing user data

---

## 4. API Security

### 4.1 Authentication Required

- [ ] **Session endpoints require auth**: Attempt unauthenticated access
  ```bash
  curl -X GET https://[app-url]/api/sessions
  # Should return 401 Unauthorized
  ```

- [ ] **No session data in public endpoints**: Review API routes for unprotected data exposure

### 4.2 Input Validation

- [ ] **SQL injection protected**: Parameterized queries used throughout
- [ ] **No direct SQL string interpolation**: Code review for string concatenation in queries

---

## 5. LLM Integration

### 5.1 Transit Security

- [ ] **TLS enforced**: OpenAI API calls use HTTPS
  ```javascript
  // Verify in code: baseURL should be https://
  ```

- [ ] **No plaintext logging of API requests**: LLM request/response bodies not logged

### 5.2 Data Minimization

- [ ] **Only necessary data sent**: Review `persistSessionDebrief` function
  - User ID not sent to LLM
  - Email not sent to LLM
  - Session IDs not sent to LLM

### 5.3 Provider Configuration

- [ ] **API-tier access confirmed**: Using OpenAI API (not ChatGPT consumer product)
- [ ] **DPA in place**: Data Processing Agreement with OpenAI exists

---

## 6. Key Management

### 6.1 Key Rotation Capability

- [ ] **Multiple key versions supported**: Code handles `encryption_version` field
- [ ] **Rotation procedure documented**: Internal runbook exists

### 6.2 Key Access Audit

- [ ] **Limited personnel access**: Document who has access to:
  - Vercel environment variables
  - Supabase dashboard
  - Both (can decrypt data)

---

## 7. Data Retention

### 7.1 Deletion Functionality

- [ ] **User can delete sessions**: Test session deletion flow
- [ ] **Cascade deletion works**: Deleting session removes debriefs

### 7.2 Retention Enforcement

- [ ] **Feedback events expire**: Verify 90-day cleanup job exists or is planned
- [ ] **Account deletion removes all data**: Test full account deletion

---

## 8. Code Review Items

### 8.1 Encryption Implementation

- [ ] **AES-256-GCM used**: Verify algorithm in encryption module
- [ ] **Random IV per operation**: No IV reuse
- [ ] **Auth tag verified**: GCM authentication checked on decrypt

### 8.2 Secret Handling

- [ ] **No hardcoded secrets**: Grep codebase for potential secrets
  ```bash
  grep -r "sk-" --include="*.ts" --include="*.js"
  grep -r "password" --include="*.ts" --include="*.js"
  ```

- [ ] **Secrets from environment only**: All secrets loaded from `process.env`

---

## 9. Compliance Items

### 9.1 Privacy Policy

- [ ] **Matches implementation**: Privacy policy accurately describes data handling
- [ ] **AI processing disclosed**: Users informed about LLM usage

### 9.2 Data Export

- [ ] **Export functionality exists**: Users can export their data
- [ ] **Export includes all user data**: Verify completeness

---

## Audit Report Template

```markdown
# Solaura Security Audit Report

**Auditor**: [Name/Organization]
**Date**: [Date]
**Scope**: [Areas reviewed]

## Summary

[Overall findings summary]

## Checklist Results

| Category | Items Passed | Items Failed | Notes |
|----------|--------------|--------------|-------|
| Encryption at Rest | X/Y | | |
| Row Level Security | X/Y | | |
| Logging | X/Y | | |
| API Security | X/Y | | |
| LLM Integration | X/Y | | |
| Key Management | X/Y | | |
| Data Retention | X/Y | | |
| Code Review | X/Y | | |
| Compliance | X/Y | | |

## Findings

### Critical
[List any critical findings]

### High
[List any high-severity findings]

### Medium
[List any medium-severity findings]

### Low
[List any low-severity findings]

## Recommendations

[Prioritized recommendations]

## Verification Evidence

[Screenshots, logs, or other evidence]
```

---

## Notes for Auditors

1. **Access requirements**: Full audit requires Supabase dashboard access and Vercel access
2. **Limited audit**: Schema and RLS can be verified with database read access only
3. **Code review**: Application repository access needed for implementation verification
4. **Questions**: Open issues on this repository for clarification
