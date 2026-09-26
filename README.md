# Solaura Therapy — Data Privacy & Transparency

**Public documentation of how Solaura Therapy handles session data, including what we do, what we do not do, and the limits of both.**

This repository describes how [Solaura Therapy](https://solaura.app) handles user data. As of **2026-09-26** it is organised around Solaura's de-identified research corpus. Encryption of live session data is still in place and is described further down.

> **Note:** The Solaura app repository is private. This public repository exists so that clients, therapists, and outside reviewers can read and question our data-handling claims.

> **Status labels:** In this repository, **Live** means running in production today. **Rolling out** means the design and policy are set and the implementation is being built and released now. Rolling-out items are not yet guaranteed for every user or session. **Planned** means an intended future use that is not active.

## What is Solaura Therapy?

Solaura Therapy is a clinician support platform developed by **Solaura Technologies, Inc.**, a Delaware C-corporation. It helps therapists by generating AI-assisted session debriefs, emotional insights, and recommendations. Clinicians review all of this before anything is shared with clients.

## Summary

| Area | What happens | Status |
|------|--------------|--------|
| Live sessions | Stored transcripts, audio, and related session content are encrypted at rest | Live |
| Research corpus | Optional, de-identified conversation text used to improve Solaura's internal models for understanding and supporting emotional states | Rolling out |
| Consent | Separate opt-in for client **and** therapist. Off by default, revocable, and a conversation is included only when both have opted in | Rolling out |
| External research licensing | Possible future licensing of de-identified data to vetted research organisations, under a second, separate opt-in. **No data has been shared or sold. No partners exist.** | Planned, not active |
| Regulatory posture | India users under the DPDP Act (specific consent). No HIPAA claims | Policy |

## De-identified research corpus (rolling out)

Solaura is building a **de-identified research corpus** to improve its internal models for understanding and supporting emotional states. It is separate from the live app and has its own safeguards.

### 1. Consent from both sides

- The client and the therapist are each asked separately and explicitly. This is not bundled into the Terms of Service.
- This opt-in covers **internal use only**. External research sharing is a second, separate opt-in (see below).
- **Off by default.** Nothing is contributed unless both have opted in.
- A session's conversation is eligible **only when both the client and the therapist in that bond have opted in**.
- Either person can withdraw at any time. Withdrawal stops contributions from **future sessions**. Already-saved contributions cannot be pulled back (see *Limits* below), and we say so at the time consent is asked.

### 2. Removing identifiers before anything is stored

Before a conversation is written to the corpus:

1. **Direct identifiers are removed automatically:** names, phone numbers, email addresses, postal addresses, ID numbers, URLs, and exact dates.
2. **Quasi-identifiers are generalised:**
   - age → decade (for example, "34" becomes "30s")
   - location → broad region, or removed
   - job → category (for example, "cardiac nurse at a named hospital" becomes "healthcare worker")
   - rare or distinctive life events → paraphrased so the specific event is not recognisable

### 3. Stored separately, with no link back

- The corpus is kept in a **separate store** from live app data.
- Corpus records contain **no bond ID, user ID, or session ID** and **no exact timestamp** (at most, the week the conversation happened).
- There is **no mapping table** from corpus records back to any person, account, or session.
- **Which client/therapist bond produced a conversation is never recorded.**

### 4. Checks after scrubbing

- **Residual-PII rescan:** every scrubbed record is scanned again for leftover identifiers. Records that fail are **quarantined** and not added to the corpus.
- **Human review:** reviewers read random samples of scrubbed records to catch what automated tools miss.

### 5. Limits

- The corpus is **de-identified, not anonymous.** We do not claim that re-identification is impossible.
- Automated scrubbing can miss **context clues**, meaning combinations of details that could point to a person even with names removed. Generalisation, the rescan, and human review lower this risk. They do not remove it.
- **Once a contribution is saved, it cannot be deleted individually.** The link back to the person is never stored, so we cannot find a specific person's contributions later. This is stated when consent is requested.

### 6. Planned future use: licensing to vetted research organisations

Solaura plans to eventually license de-identified corpus data to **vetted outside research organisations**. This is a **planned future use and is not active.**

- **No data has been shared, licensed, or sold today. No research partners exist yet.**
- **A second, separate opt-in** for both client and therapist. Off by default and revocable. People can choose internal use without choosing external sharing.
- Before any sharing, Solaura would require all of the following:
  - a data use agreement that bans re-identification, limits use to research, bans onward transfer, and gives Solaura audit rights
  - access in a controlled environment where feasible
  - stricter de-identification and human review for each outbound batch
  - legal review under the DPDP Act
  - a published list of partners in this repository

Full design: [docs/deidentified-research-corpus.md](docs/deidentified-research-corpus.md)

## Live session encryption

Live session data (stored transcripts and audio, debriefs, and related fields) **stays encrypted at rest** under the Phase 1 design documented on 2026-09-21. This protection is still in place and is unchanged by the corpus work.

- **At-rest encryption (Live):** server-side envelope encryption (AES-256-GCM), with keys held outside the database (Vercel environment / KMS).
- **Remaining risk:** an operator with both key access and database access could decrypt stored live-session data.
- **Plaintext during processing:** to generate AI debriefs, transcripts are decrypted on the server and sent to the LLM provider over TLS. See [docs/ephemeral-plaintext.md](docs/ephemeral-plaintext.md).
- **Earlier roadmap:** client-held keys and sealed or on-device inference are still described in [docs/encryption-design.md](docs/encryption-design.md) as roadmap items. This update does not change their status.

## Jurisdiction and regulatory posture

- **Entity:** Solaura Technologies, Inc., a Delaware C-corporation.
- **India:** users in India are covered by the Digital Personal Data Protection Act, 2023 (DPDP Act). Contribution to the research corpus relies on **specific, separate consent** for that purpose. Any future external sharing would need its own separate consent and legal review under the DPDP Act first.
- **No HIPAA claims.** We do not claim HIPAA compliance.
- **Mental-health data is treated as highly sensitive**, including in the de-identified corpus.
- We do not claim any certifications, third-party audits, or accuracy metrics for the de-identification pipeline.

Details: [docs/legal-notes.md](docs/legal-notes.md)

## Documentation

| Document | Description |
|----------|-------------|
| [De-identified Research Corpus](docs/deidentified-research-corpus.md) | Consent, de-identification pipeline, storage, review, limits (rolling out), and planned external licensing (not active) |
| [Training Transparency](docs/training-transparency.md) | How Solaura's AI works today and how the corpus relates to model improvement |
| [Data Inventory](docs/data-inventory.md) | Field-by-field data mapping, including the corpus store |
| [Threat Model](docs/threat-model.md) | Threat actors, attack surfaces, and mitigations, including re-identification |
| [Legal Notes](docs/legal-notes.md) | Entity information and regulatory posture (DPDP, no HIPAA claims) |
| [Encryption Design](docs/encryption-design.md) | Live session encryption: key hierarchy and envelope encryption |
| [Ephemeral Plaintext](docs/ephemeral-plaintext.md) | Plaintext during AI inference and de-identification processing |
| [Audit Checklist](docs/audit-checklist.md) | Verification steps for outside reviewers |
| [Archive note](docs/archive/README.md) | What changed from the original encryption-first framing |
| [CHANGELOG](CHANGELOG.md) | Dated record of changes to this repository |

## Principles

1. **Say what we actually do.** Rolling-out items are labelled as such.
2. **State the limits.** De-identified is not anonymous, and we say so.
3. **Consent from both sides.** Nothing is contributed without explicit opt-in from both client and therapist, and external sharing needs a second, separate opt-in.
4. **No link back.** The corpus is designed so that no one, including Solaura staff, can trace a record to a person.
5. **No invented assurances.** No made-up metrics, certifications, or audits.

## Contributing

Security researchers, clinicians, and privacy advocates are welcome to:
- Open issues with questions or concerns
- Submit PRs that improve the accuracy of this documentation
- Ask us to clarify any claim made here

## License

This documentation is released under the [MIT License](LICENSE).

---

*Maintained by Solaura Technologies, Inc.*
