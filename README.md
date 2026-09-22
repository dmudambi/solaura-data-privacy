# Solaura Therapy — Data Privacy & Transparency

**Open audit documentation for Solaura Therapy data handling practices.**

This repository provides transparent, honest documentation of how [Solaura Therapy](https://solaura.app) handles user data, including our current encryption implementation, known limitations, and roadmap for improvement.

> **Note:** The Solaura companion app repository (AURA_APP) is private. This public transparency repo exists to allow external security review of our data handling claims.

## What is Solaura Therapy?

Solaura Therapy is a clinician support platform developed by **Solaura Technologies, Inc.**, a Delaware C-corporation. The app helps therapists and clinicians by generating AI-powered session debriefs, emotional insights, and personalized recommendations — all subject to clinician review before sharing with clients.

## Current State: Honest Summary

### Phase 1 (Current)

- **At-rest encryption**: Sensitive fields use server-side envelope encryption with keys stored outside the database (Vercel environment / KMS)
- **Residual risk**: Operators with both KMS access AND database access can decrypt stored data
- **Ephemeral plaintext**: During AI debrief generation, transcripts are sent to LLM providers (OpenAI, etc.) in plaintext over TLS
- **No HIPAA claims**: We do not claim HIPAA compliance

### Phase 2 (Roadmap)

- Client-held encryption keys (WebCrypto) so backend cannot decrypt stored blobs
- Sealed/client-side inference to eliminate ephemeral plaintext exposure to LLM providers
- Full end-to-end encryption for sensitive session data

## Documentation

| Document | Description |
|----------|-------------|
| [Training Transparency](docs/training-transparency.md) | How our AI insight model was developed (no fine-tuning on private data) |
| [Threat Model](docs/threat-model.md) | Attack surfaces, threat actors, and mitigations |
| [Data Inventory](docs/data-inventory.md) | Complete field-by-field data sensitivity mapping |
| [Encryption Design](docs/encryption-design.md) | Key hierarchy, envelope encryption, and roadmap |
| [Ephemeral Plaintext](docs/ephemeral-plaintext.md) | Honest disclosure of plaintext during AI inference |
| [Audit Checklist](docs/audit-checklist.md) | Verification steps for external reviewers |
| [Legal Notes](docs/legal-notes.md) | Entity information and regulatory posture |

## Key Principles

1. **Honesty over marketing**: We document actual implementation, not aspirational claims
2. **Residual risks acknowledged**: We explicitly state what we cannot yet protect against
3. **Phased improvement**: Clear roadmap from current state to full E2E encryption
4. **External verifiability**: Documentation designed for independent security review

## Contributing

Security researchers and privacy advocates are welcome to:
- Open issues for questions or concerns
- Submit PRs to improve documentation accuracy
- Request clarification on any claims made

## License

This documentation is released under the [MIT License](LICENSE).

---

*Maintained by Solaura Technologies, Inc.*
