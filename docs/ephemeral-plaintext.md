# Ephemeral Plaintext During AI Inference

**Honest disclosure of plaintext exposure during LLM-powered features.**

## Overview

Solaura uses AI/LLM services (currently OpenAI) to generate personalized session debriefs, insights, and focus recommendations. This document honestly describes the plaintext exposure that occurs during this process.

## Current Implementation (Phase 1)

### The Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                    Debrief Generation Flow                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   1. Session completes                                           │
│                                                                  │
│   2. Client calls POST /api/session/complete                     │
│                                                                  │
│   3. Server decrypts transcript from database                    │
│      (using server-held KEK)                                     │
│                                                                  │
│   4. Server calls persistSessionDebrief()                        │
│         │                                                        │
│         ▼                                                        │
│   ┌─────────────────────────────────────────────────────────┐    │
│   │  PLAINTEXT TRANSCRIPT SENT TO LLM PROVIDER              │    │
│   │                                                         │    │
│   │  • Transmitted over TLS (encrypted in transit)          │    │
│   │  • Processed in LLM provider's infrastructure           │    │
│   │  • Exists in memory during inference                    │    │
│   │  • Subject to provider's data handling policies         │    │
│   └─────────────────────────────────────────────────────────┘    │
│         │                                                        │
│         ▼                                                        │
│   5. LLM returns generated debrief                               │
│                                                                  │
│   6. Server encrypts debrief, stores in database                 │
│                                                                  │
│   7. Transcript plaintext no longer in server memory             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### What Gets Sent to LLM Providers

| Data | Sent to LLM? | Purpose |
|------|--------------|---------|
| Session transcript | **Yes** | Generate insights and summary |
| Emotions data | **Yes** | Contextualize emotional journey |
| Session highlights | **Yes** | Identify key moments |
| Goal framework | **Yes** | Personalize recommendations |
| User ID | **No** | Not needed for inference |
| Email | **No** | Not needed for inference |
| Session metadata | **Partial** | Duration for context only |

### Duration of Exposure

| Phase | Duration | Location |
|-------|----------|----------|
| In transit | Milliseconds | TLS-encrypted network |
| Processing | Seconds | LLM provider infrastructure |
| Post-inference | None | Not retained by request |

## LLM Provider Policies

### OpenAI (Current Provider)

As of our integration:
- API data is **not used to train models** by default for API customers
- Data retention configurable (we request minimal retention)
- SOC 2 Type II certified
- Data Processing Agreement in place

> **Caveat**: We rely on OpenAI's stated policies. We cannot independently verify their internal data handling.

### What We Cannot Guarantee

1. **LLM provider internal practices**: We trust but cannot verify
2. **Provider infrastructure security**: Beyond our control
3. **Provider employee access**: Subject to their access controls
4. **Regulatory requests to provider**: May be disclosed under legal compulsion
5. **Provider breach**: Our data could be exposed

## Why This Exposure Exists

### Technical Necessity

Current LLM APIs require plaintext input:
- No homomorphic encryption support
- No secure enclave inference available
- No client-side models of sufficient quality

### User Value Trade-off

The debrief feature provides significant user value:
- Personalized insights from session content
- Actionable focus recommendations
- Pattern recognition across sessions

Users implicitly accept this trade-off when using AI-powered features.

## Mitigations in Place

### In Transit
- TLS 1.3 for all API calls
- Certificate pinning where supported
- No plaintext logging of requests

### At Provider
- Data Processing Agreement (DPA)
- API-tier data handling (no training use)
- Request-level processing (no persistent storage requested)

### Post-Processing
- Plaintext cleared from server memory
- Generated insights encrypted before storage
- No caching of decrypted transcripts

## Phase 2 Solutions (Roadmap)

### Option A: Sealed Inference

```
┌────────────────────────────────────────────────────────┐
│                  Sealed Inference                      │
├────────────────────────────────────────────────────────┤
│                                                        │
│   Trusted Execution Environment (TEE)                  │
│   ┌──────────────────────────────────────────────┐     │
│   │                                              │     │
│   │   Client encrypts transcript                 │     │
│   │            ↓                                 │     │
│   │   Sealed enclave decrypts                    │     │
│   │            ↓                                 │     │
│   │   LLM inference inside enclave               │     │
│   │            ↓                                 │     │
│   │   Output encrypted to client key             │     │
│   │                                              │     │
│   └──────────────────────────────────────────────┘     │
│                                                        │
│   • Host cannot see plaintext                          │
│   • Requires TEE-capable providers                     │
│   • Performance overhead                               │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**Status**: Monitoring Azure Confidential Computing, AWS Nitro Enclaves for LLM support.

### Option B: On-Device Inference

```
┌────────────────────────────────────────────────────────┐
│               Client-Side Inference                    │
├────────────────────────────────────────────────────────┤
│                                                        │
│   User Device                                          │
│   ┌──────────────────────────────────────────────┐     │
│   │                                              │     │
│   │   Local LLM (quantized, optimized)           │     │
│   │            ↓                                 │     │
│   │   Transcript never leaves device             │     │
│   │            ↓                                 │     │
│   │   Debrief generated locally                  │     │
│   │            ↓                                 │     │
│   │   Only encrypted debrief synced              │     │
│   │                                              │     │
│   └──────────────────────────────────────────────┘     │
│                                                        │
│   • Zero server exposure                               │
│   • Requires capable device                            │
│   • Quality trade-offs vs. cloud models                │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**Status**: Evaluating WebLLM, llama.cpp WASM builds for browser-based inference.

### Option C: Federated / Split Inference

Hybrid approach where sensitive processing happens client-side with cloud assistance for non-sensitive operations.

**Status**: Research phase.

## User Transparency

### What Users Should Know

1. AI-powered features require sending content to LLM providers
2. Content is transmitted securely (TLS) but processed in plaintext
3. We use business-tier APIs with data protection agreements
4. Zero-knowledge AI inference is not yet technically feasible at quality

### Opt-Out Options

Currently, users can:
- Skip debrief generation (manual mode)
- Delete sessions immediately after viewing
- Request account deletion (removes all data)

Future options under consideration:
- Fully local debrief generation (quality trade-off)
- Encrypted storage without AI features

## Audit Evidence

To verify our claims, external reviewers can:

1. Inspect `/api/session/complete` route handler
2. Verify no plaintext logging in Vercel functions
3. Review OpenAI API integration code
4. Check that encrypted storage occurs post-inference
5. Confirm DPA is in place with OpenAI

## Summary

| Claim | Honest Status |
|-------|---------------|
| "We never see your data" | ❌ **False** — Server decrypts for LLM calls |
| "End-to-end encrypted" | ❌ **False during Phase 1** — Broken for AI features |
| "Encrypted in transit" | ✅ True — TLS for all transmission |
| "Encrypted at rest" | ✅ True — Stored encrypted in database |
| "LLM provider sees plaintext" | ✅ **True** — Ephemeral during inference |
| "Working toward zero-knowledge AI" | ✅ True — Phase 2 roadmap |
