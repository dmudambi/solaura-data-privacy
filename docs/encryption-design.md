# Encryption Design

Technical specification for Solaura's encryption architecture, including current implementation and roadmap.

## Design Principles

1. **Defense in depth**: Multiple layers of protection
2. **Key separation**: Encryption keys never stored alongside encrypted data
3. **Forward compatibility**: Design supports migration to stronger models
4. **Minimal plaintext surface**: Encrypt anything that doesn't need to be queryable

## Key Hierarchy

### Phase 1: Server-Side Envelope Encryption

```
┌─────────────────────────────────────────────────────────┐
│                    Key Hierarchy                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌─────────────┐                                       │
│   │  Root KEK   │  ← Stored in Vercel env / KMS        │
│   │  (Master)   │     NOT in database                   │
│   └──────┬──────┘                                       │
│          │                                              │
│          │ derives                                      │
│          ▼                                              │
│   ┌─────────────┐                                       │
│   │  App KEK    │  ← Application-level key             │
│   │             │     Rotatable without re-encrypting   │
│   └──────┬──────┘     all data                         │
│          │                                              │
│          │ wraps                                        │
│          ▼                                              │
│   ┌─────────────┐                                       │
│   │    DEKs     │  ← Per-record data encryption keys   │
│   │  (wrapped)  │     Stored encrypted alongside data   │
│   └──────┬──────┘                                       │
│          │                                              │
│          │ encrypts                                     │
│          ▼                                              │
│   ┌─────────────┐                                       │
│   │  User Data  │  ← Transcripts, insights, etc.       │
│   │ (encrypted) │     Stored as ciphertext in DB       │
│   └─────────────┘                                       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Encryption Algorithm

- **Algorithm**: AES-256-GCM
- **DEK size**: 256 bits (generated per-record or per-field)
- **IV/Nonce**: 96 bits, randomly generated per encryption operation
- **Authentication tag**: 128 bits (GCM provides authenticated encryption)

### Envelope Encryption Process

**Encryption (Write):**
1. Generate random DEK (256-bit)
2. Encrypt plaintext with DEK using AES-256-GCM
3. Encrypt DEK with KEK (key wrapping)
4. Store: `{ wrapped_dek, iv, ciphertext, auth_tag, encryption_version }`

**Decryption (Read):**
1. Retrieve encrypted blob from database
2. Unwrap DEK using KEK from Vercel env
3. Decrypt ciphertext using DEK and IV
4. Verify authentication tag
5. Return plaintext

### Key Storage

| Key | Storage Location | Access Control |
|-----|-----------------|----------------|
| Root KEK | Vercel Environment Variables (encrypted at rest) | Vercel dashboard access |
| App KEK | Derived from Root KEK | Application runtime only |
| DEKs | Stored wrapped in database | Useless without KEK |

## Phase 1 Security Properties

**Protected against:**
- Database dump without KEK
- Backup exfiltration without KEK
- SQL injection reading ciphertext only
- Supabase platform breach (without Vercel access)

**NOT protected against:**
- Operator with both Vercel env access AND database access
- Compromise of application runtime (keys in memory)
- LLM provider during inference (see [ephemeral-plaintext.md](ephemeral-plaintext.md))

## Phase 1.5: Enhanced Key Management (Planned)

Improvements without requiring client-side changes:

1. **AWS KMS / GCP Cloud KMS integration**
   - Hardware-backed key storage
   - Audit logging of all key operations
   - Automatic key rotation

2. **Per-user KEKs**
   - Derived from master KEK + user ID
   - Limits blast radius of key compromise
   - Enables user-specific key rotation

3. **Key versioning**
   - Support multiple KEK versions simultaneously
   - Graceful rotation without downtime
   - `encryption_version` field already in schema

## Phase 2: Client-Held Keys (Roadmap)

**Goal**: Backend cannot decrypt stored data.

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│              Phase 2: Client-Side Keys                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌─────────────┐                                       │
│   │ User Secret │  ← Password / biometric / passkey    │
│   └──────┬──────┘                                       │
│          │                                              │
│          │ derives (PBKDF2/Argon2)                      │
│          ▼                                              │
│   ┌─────────────┐                                       │
│   │  User KEK   │  ← Never leaves client device        │
│   │  (client)   │     Derived deterministically        │
│   └──────┬──────┘                                       │
│          │                                              │
│          │ wraps                                        │
│          ▼                                              │
│   ┌─────────────┐                                       │
│   │    DEKs     │  ← Generated client-side             │
│   │  (wrapped)  │     Wrapped DEK synced to server     │
│   └──────┬──────┘                                       │
│          │                                              │
│          │ encrypts (client-side)                       │
│          ▼                                              │
│   ┌─────────────┐                                       │
│   │  User Data  │  ← Encrypted before leaving device   │
│   │ (E2E blob)  │     Server stores opaque blob        │
│   └─────────────┘                                       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### WebCrypto Implementation

```javascript
// Pseudocode for client-side encryption
const userKey = await deriveKey(userPassword, salt);
const dek = await crypto.subtle.generateKey(
  { name: 'AES-GCM', length: 256 },
  true,
  ['encrypt', 'decrypt']
);
const ciphertext = await crypto.subtle.encrypt(
  { name: 'AES-GCM', iv: randomIV },
  dek,
  plaintextBuffer
);
const wrappedDek = await crypto.subtle.wrapKey('raw', dek, userKey, 'AES-KW');
// Send { wrappedDek, iv, ciphertext } to server
// Server stores but CANNOT decrypt
```

### Key Recovery (Phase 2)

Options under consideration:
1. **Recovery key**: User-generated, stored offline
2. **Social recovery**: Split key among trusted contacts
3. **Escrow (opt-in)**: Server-held recovery key, user choice

### Migration Path

1. New sessions encrypted client-side
2. User-initiated re-encryption of historical data
3. Server-side encrypted data marked with `encryption_version`
4. Gradual migration as users access old sessions

## What Stays Server-Side

Even in Phase 2, some data must remain server-accessible:

| Data | Reason |
|------|--------|
| User ID | Authentication, RLS enforcement |
| Email | Account recovery, notifications |
| Session IDs | Database relationships |
| Timestamps | Sorting, retention, billing |
| Encryption metadata | Version selection, key IDs |
| Aggregate analytics | Service improvement (no content) |

## Encryption Version Markers

Each encrypted field includes version metadata:

| Version | Meaning |
|---------|---------|
| `0` or null | Legacy plaintext (migration needed) |
| `1` | Phase 1 server-side envelope encryption |
| `2` | Phase 2 client-side E2E encryption |

This allows:
- Gradual migration without downtime
- Correct decryption path selection
- Audit of encryption coverage

## Key Rotation

### Phase 1 Rotation Process

1. Generate new KEK version
2. Add to Vercel env alongside old KEK
3. New writes use new KEK
4. Background job re-encrypts old records
5. Remove old KEK after full migration

### Rotation Triggers

- Scheduled (quarterly recommended)
- Personnel changes (team member departure)
- Suspected compromise
- Compliance requirements

## Security Considerations

### Timing Attacks

- Constant-time comparison for authentication tags
- No early-exit on decryption failure

### Memory Handling

- Keys cleared from memory after use where possible
- No logging of key material or plaintext

### Randomness

- All IVs/nonces from cryptographically secure RNG
- No IV reuse (random generation per operation)
