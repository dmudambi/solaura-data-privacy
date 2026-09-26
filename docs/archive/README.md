# Archive Note: Encryption-First Framing (2026-09-21 to 2026-09-25)

From 2026-09-21 until 2026-09-26, this repository was organised around **Phase 1 at-rest encryption** of live session data. Its main message was that operators should not have plaintext access to stored session data without both database and key access. It also presented a Phase 2 roadmap toward client-held keys and sealed inference.

## What changed on 2026-09-26

The repository is now organised around Solaura's **de-identified research corpus**. See [../deidentified-research-corpus.md](../deidentified-research-corpus.md).

## What still applies

- **Phase 1 at-rest encryption of live session data is still in place.** It is documented in [../encryption-design.md](../encryption-design.md) and summarised in the README's "Live session encryption" section.
- The ephemeral plaintext disclosure for AI debriefs ([../ephemeral-plaintext.md](../ephemeral-plaintext.md)) still applies.
- The Phase 1.5 and Phase 2 encryption roadmap items are kept as written. This update does not change their status.

## What was superseded

| Earlier statement (2026-09-21 / 2026-09-22) | Current position (2026-09-26) |
|---------------------------------------------|-------------------------------|
| README headline: at-rest encryption and the Phase 2 end-to-end roadmap | README headline: opt-in de-identified research corpus. Encryption described as retained protection for live sessions |
| "No therapy transcripts from users are used to train models" / "User sessions are not collected as training data" (training-transparency.md) | With opt-in from both client and therapist, de-identified conversation text is used to improve Solaura's internal models (rolling out). Sessions without both opt-ins are not used |

The full earlier text is kept in this repository's git history (commit `ad794be`, 2026-09-22).
