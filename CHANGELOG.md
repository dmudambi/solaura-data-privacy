# Changelog

All notable changes to this repository are recorded here. Dates are in YYYY-MM-DD format.

## 2026-09-26

### Changed
- **The repository is now organised around the de-identified research corpus.** The README was rewritten so that the corpus approach replaces the earlier encryption-first framing. Live session encryption is still in place and now has its own "Live session encryption" section.
- `docs/training-transparency.md`: the earlier statement that user sessions are not collected as training data no longer applies. The document now describes the opt-in, de-identified corpus used to improve Solaura's internal models (rolling out).
- `docs/data-inventory.md`: added the separate corpus store (no bond, user, or session ID; week bucket at most; no mapping table), the consent flags, and corpus retention.
- `docs/threat-model.md`: added corpus re-identification, insider linkage, and future external recipient misuse.
- `docs/legal-notes.md`: added DPDP Act specific consent for the corpus and for planned external sharing, a statement that mental-health data is treated as highly sensitive, and non-claims for anonymity, partners, and certifications. HIPAA non-claim kept.
- `docs/encryption-design.md`, `docs/ephemeral-plaintext.md`: added scope notes saying these cover live session data. Added a section on plaintext handling during de-identification.
- `docs/audit-checklist.md`: added Section 10 with reviewer checks for the corpus.

### Added
- `docs/deidentified-research-corpus.md`: consent from both client and therapist (default off, revocable for future sessions), the de-identification pipeline, separate storage with no link back, residual-PII rescan and quarantine, human review of random samples, and limits (de-identified, not anonymous; saved contributions cannot be individually deleted).
- Planned future use: licensing de-identified corpus data to vetted research organisations, under a second, separate opt-in and with required safeguards. **Not active. No data has been shared, licensed, or sold, and no partners exist.**
- `docs/archive/README.md`: note on what the encryption-first framing said and what replaced it.
- `CHANGELOG.md` (this file).

### Status
- Corpus components are marked **Rolling out**. External licensing is marked **Planned, not active**. Phase 1 at-rest encryption of live session data is **Live** and unchanged.

## 2026-09-22

### Added
- `docs/training-transparency.md`: prompt-engineered foundation models, evaluation domains, clinician review workflow.
- README updated to Solaura Therapy branding.

## 2026-09-21

### Added
- Initial transparency documentation: README (Phase 1 vs Phase 2 encryption), threat model, data inventory, encryption design, ephemeral plaintext disclosure, audit checklist, legal notes, MIT License.
