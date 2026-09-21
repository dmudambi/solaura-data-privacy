# Data Inventory

Complete inventory of sensitive data fields, their storage locations, current protection status, and encryption roadmap.

## Sensitivity Levels

| Level | Description | Examples |
|-------|-------------|----------|
| **Critical** | Highly personal, therapeutic content | Transcripts, emotional insights |
| **Sensitive** | Personal but less revealing | Goals, session notes |
| **Internal** | Operational data, not user-generated content | Session IDs, timestamps |
| **Public** | Non-sensitive identifiers | User ID (opaque UUID) |

## Data Field Inventory

### Session Data (`sessions` table)

| Field | Store | Sensitivity | Plaintext Today? | Encrypted Target | Who Can Decrypt | Retention |
|-------|-------|-------------|------------------|------------------|-----------------|-----------|
| `id` | sessions.id | Internal | Yes | Stays plaintext | N/A (identifier) | Indefinite |
| `user_id` | sessions.user_id | Internal | Yes | Stays plaintext | N/A (FK for RLS) | Indefinite |
| `bond_id` | sessions.bond_id | Internal | Yes | Stays plaintext | N/A (FK) | Indefinite |
| `transcript_json` | sessions.transcript_json | **Critical** | **Phase 1: Encrypted** | Client-held key (Phase 2) | Phase 1: KEK holders; Phase 2: User only | User-controlled |
| `emotions_json` | sessions.emotions_json | **Critical** | **Phase 1: Encrypted** | Client-held key (Phase 2) | Phase 1: KEK holders; Phase 2: User only | User-controlled |
| `highlights_json` | sessions.highlights_json | **Critical** | **Phase 1: Encrypted** | Client-held key (Phase 2) | Phase 1: KEK holders; Phase 2: User only | User-controlled |
| `goal_framework` | sessions.goal_framework | Sensitive | **Phase 1: Encrypted** | Client-held key (Phase 2) | Phase 1: KEK holders; Phase 2: User only | User-controlled |
| `pre_session_notes` | sessions.pre_session_notes | Sensitive | **Phase 1: Encrypted** | Client-held key (Phase 2) | Phase 1: KEK holders; Phase 2: User only | User-controlled |
| `duration_seconds` | sessions.duration_seconds | Internal | Yes | Stays plaintext | N/A (metadata) | Indefinite |
| `created_at` | sessions.created_at | Internal | Yes | Stays plaintext | N/A (metadata) | Indefinite |
| `updated_at` | sessions.updated_at | Internal | Yes | Stays plaintext | N/A (metadata) | Indefinite |
| `too_short_for_summary` | sessions.too_short_for_summary | Internal | Yes | Stays plaintext | N/A (flag) | Indefinite |
| `encryption_version` | sessions.encryption_version | Internal | Yes | Stays plaintext | N/A (metadata) | Indefinite |

### Session Debriefs (`session_debriefs` table)

| Field | Store | Sensitivity | Plaintext Today? | Encrypted Target | Who Can Decrypt | Retention |
|-------|-------|-------------|------------------|------------------|-----------------|-----------|
| `id` | session_debriefs.id | Internal | Yes | Stays plaintext | N/A | Indefinite |
| `session_id` | session_debriefs.session_id | Internal | Yes | Stays plaintext | N/A (FK) | Indefinite |
| `insight` | session_debriefs.insight | **Critical** | **Phase 1: Encrypted** | Client-held key (Phase 2) | Phase 1: KEK holders; Phase 2: User only | User-controlled |
| `focus_next` | session_debriefs.focus_next | Sensitive | **Phase 1: Encrypted** | Client-held key (Phase 2) | Phase 1: KEK holders; Phase 2: User only | User-controlled |
| `insight_edited` | session_debriefs.insight_edited | **Critical** | **Phase 1: Encrypted** | Client-held key (Phase 2) | Phase 1: KEK holders; Phase 2: User only | User-controlled |
| `focus_next_edited` | session_debriefs.focus_next_edited | Sensitive | **Phase 1: Encrypted** | Client-held key (Phase 2) | Phase 1: KEK holders; Phase 2: User only | User-controlled |
| `created_at` | session_debriefs.created_at | Internal | Yes | Stays plaintext | N/A | Indefinite |

### Feedback Events (`feedback_events` table)

| Field | Store | Sensitivity | Plaintext Today? | Encrypted Target | Who Can Decrypt | Retention |
|-------|-------|-------------|------------------|------------------|-----------------|-----------|
| `id` | feedback_events.id | Internal | Yes | Stays plaintext | N/A | Indefinite |
| `user_id` | feedback_events.user_id | Internal | Yes | Stays plaintext | N/A (FK) | Indefinite |
| `event_type` | feedback_events.event_type | Internal | Yes | Stays plaintext | N/A | Indefinite |
| `event_data` | feedback_events.event_data | Sensitive | **Phase 1: Encrypted** | Client-held key (Phase 2) | Phase 1: KEK holders; Phase 2: User only | 90 days |
| `created_at` | feedback_events.created_at | Internal | Yes | Stays plaintext | N/A | 90 days |

### Shared Items (`shared_items` table)

| Field | Store | Sensitivity | Plaintext Today? | Encrypted Target | Who Can Decrypt | Retention |
|-------|-------|-------------|------------------|------------------|-----------------|-----------|
| `id` | shared_items.id | Internal | Yes | Stays plaintext | N/A | Until deleted |
| `user_id` | shared_items.user_id | Internal | Yes | Stays plaintext | N/A (FK) | Until deleted |
| `item_type` | shared_items.item_type | Internal | Yes | Stays plaintext | N/A | Until deleted |
| `content` | shared_items.content | **Critical** | **Phase 1: Encrypted** | Client-held key (Phase 2) | Phase 1: KEK holders; Phase 2: User only | Until deleted |
| `share_token` | shared_items.share_token | Internal | Yes | Stays plaintext | N/A (access control) | Until deleted |

### Bond Soul (`bond_soul` table)

| Field | Store | Sensitivity | Plaintext Today? | Encrypted Target | Who Can Decrypt | Retention |
|-------|-------|-------------|------------------|------------------|-----------------|-----------|
| `id` | bond_soul.id | Internal | Yes | Stays plaintext | N/A | Indefinite |
| `bond_id` | bond_soul.bond_id | Internal | Yes | Stays plaintext | N/A (FK) | Indefinite |
| `encrypted_content` | bond_soul.encrypted_content | **Critical** | **Phase 1: Encrypted** | Client-held key (Phase 2) | Phase 1: KEK holders; Phase 2: User only | Indefinite |
| `encryption_version` | bond_soul.encryption_version | Internal | Yes | Stays plaintext | N/A | Indefinite |

### Authentication & Identity

| Field | Store | Sensitivity | Plaintext Today? | Encrypted Target | Who Can Decrypt | Retention |
|-------|-------|-------------|------------------|------------------|-----------------|-----------|
| `user_id` | auth.users.id | Public | Yes | Stays plaintext | N/A (opaque UUID) | Account lifetime |
| `email` | auth.users.email | Sensitive | Yes | Stays plaintext | N/A (required for auth) | Account lifetime |
| `phone` | auth.users.phone | Sensitive | Yes (if provided) | Stays plaintext | N/A (required for auth) | Account lifetime |
| `created_at` | auth.users.created_at | Internal | Yes | Stays plaintext | N/A | Account lifetime |

> **Note on auth fields**: Email and phone are required in plaintext for Supabase Auth to function (login, password reset, etc.). These cannot be encrypted without breaking authentication.

## Encryption Status Legend

| Status | Meaning |
|--------|---------|
| **Phase 1: Encrypted** | Server-side envelope encryption with KEK in Vercel env/KMS |
| **Stays plaintext** | Required for functionality (auth, RLS, indexing) or non-sensitive metadata |
| **Client-held key (Phase 2)** | Future state: encrypted with user-controlled keys |

## Data Flow Summary

```
User Input → App Client → Vercel Functions → Supabase
                              ↓
                    [Phase 1: Server encrypts before DB write]
                    [Phase 2: Client encrypts before send]
                              ↓
                    Encrypted blob stored in Supabase
```

## Retention Policies

| Data Type | Retention | User Control |
|-----------|-----------|--------------|
| Session content | Indefinite | User can delete individual sessions |
| Debriefs | Indefinite | Deleted with parent session |
| Feedback events | 90 days | Auto-deleted |
| Shared items | Until deleted | User can revoke/delete |
| Account data | Account lifetime | User can request deletion |

## What Stays Plaintext (By Design)

The following remain unencrypted for operational necessity:

1. **User ID / Email** — Required for authentication and account recovery
2. **Session/Bond IDs** — Required for database relationships and RLS
3. **Timestamps** — Required for sorting, querying, retention enforcement
4. **Duration** — Aggregated analytics (no content)
5. **Encryption version markers** — Required to select correct decryption path
6. **Boolean flags** — Operational (e.g., `too_short_for_summary`)

These fields do not contain user-generated therapeutic content.
