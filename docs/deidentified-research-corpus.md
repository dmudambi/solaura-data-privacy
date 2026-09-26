# De-identified Research Corpus

**Status: Rolling out.** This document sets out the design and policy for Solaura's de-identified research corpus. The implementation is being built and released now. Until a step is marked Live, read it as the design Solaura is committing to, not as a description of production behaviour today.

*First published: 2026-09-26*

## Purpose

Solaura Technologies, Inc. is building a de-identified corpus of therapy-session conversation text. It will be used to **improve Solaura's internal models for understanding and supporting emotional states**. Licensing to vetted outside research organisations is a **planned future use** that is not active ([Section 5](#5-planned-future-use-licensing-to-vetted-research-organisations)).

Nothing enters the corpus without explicit opt-in from both people in the session. It is kept apart from live app data, and it is designed so that no record can be traced back to a person, an account, or a client/therapist bond.

## Scope

| In scope | Out of scope |
|----------|--------------|
| Conversation text from sessions where **both** client and therapist opted in, after de-identification | Sessions where either person has not opted in |
| Generalised context (age decade, broad region, job category) | Names, contact details, IDs, URLs, exact dates, exact ages, precise locations |
| A coarse time bucket (week, at most) | Exact timestamps; bond, user, or session identifiers |

Live session storage (encrypted at rest) is covered in [encryption-design.md](encryption-design.md). This document covers only the corpus.

## 1. Consent

Consent has **two separate tiers**, each asked separately of both the client and the therapist:

| Tier | Covers | Default | Status |
|------|--------|---------|--------|
| **Tier 1: Internal use** | Contributing de-identified conversations to improve Solaura's internal models | Off | Rolling out |
| **Tier 2: External research sharing** | Sharing or licensing de-identified corpus data to vetted outside research organisations | Off | **Planned future use. Not active.** |

Tier 1 can be chosen without Tier 2. Tier 2 is a separate choice and is never implied by Tier 1. Details of Tier 2 are in [Section 5](#5-planned-future-use-licensing-to-vetted-research-organisations).

### 1.1 Tier 1: two separate opt-ins

| Principle | Design |
|-----------|--------|
| Separate | Corpus consent is its own explicit choice. It is **not** part of general Terms of Service acceptance. |
| Both parties | The **client** and the **therapist** each opt in separately. |
| Default | **Off.** No one is opted in by default. |
| Eligibility | A session is eligible only if **both** the client and the therapist in that bond have opted in at the time of the session. |
| Revocable | Either person can withdraw at any time. Withdrawal applies to **future sessions only.** |
| Informed | The consent screen says plainly that saved contributions are de-identified (not anonymous), that scrubbing can miss context clues, and that saved contributions cannot be individually deleted. |
| Separate from external sharing | Tier 1 covers internal use only. It does not permit sharing or licensing outside Solaura. |

### 1.2 Withdrawal

Withdrawing stops any further sessions from being contributed. It cannot remove contributions already saved. Those records carry no link to the person who withdrew, so they cannot be found (see [Section 6](#6-limits)).

## 2. De-identification pipeline

The pipeline runs **before** anything is written to the corpus store. A conversation that does not pass every step is not stored in the corpus.

```
Eligible session (client AND therapist opted in)
        │
        ▼
Step 1  Automated removal of direct identifiers
        │
        ▼
Step 2  Generalisation of quasi-identifiers
        │
        ▼
Step 3  Residual-PII rescan ──── fail ───► Quarantine (not added to corpus)
        │ pass
        ▼
Step 4  Write to separate corpus store
        (no bond / user / session ID, week bucket at most)
        │
        ▼
Step 5  Human review of random samples
```

### Step 1: Remove direct identifiers

Automated removal of:

- Names (people, and places or organisations that would identify someone)
- Phone numbers
- Email addresses
- Postal and street addresses
- Identification numbers (government IDs, account numbers, and similar)
- URLs and handles
- Exact dates

Removed items may be replaced with neutral placeholders (for example `[NAME]`, `[DATE]`) so the conversation still reads coherently.

### Step 2: Generalise quasi-identifiers

Some details are not identifying alone but can be in combination. These are generalised:

| Detail | Treatment | Example |
|--------|-----------|---------|
| Age | Reduced to decade | "I'm 34" → "30s" |
| Location | Reduced to broad region, or removed | A neighbourhood or city → region, or removed |
| Occupation | Reduced to category | A specific role at a named employer → job category |
| Rare life events | Paraphrased so the specific event is not recognisable | A widely reported or unusual event → a general description |

### Step 3: Residual-PII rescan

Each scrubbed record is scanned again for leftover identifiers. **Records that fail are quarantined.** They are held back and not added to the corpus.

### Step 4: Separate storage

See [Section 3](#3-storage).

### Step 5: Human review

Reviewers read **random samples** of scrubbed records to find identifiers or context clues the automated steps missed. Reviewers work from the scrubbed records, and what they find is used to improve the automated steps.

## 3. Storage

| Property | Design |
|----------|--------|
| Location | A **separate store**, apart from live app data |
| Bond ID | **Not stored** |
| User ID (client or therapist) | **Not stored** |
| Session ID | **Not stored** |
| Timestamp | **No exact timestamp.** A week bucket at most. |
| Mapping / lookup table | **None.** No table, log, or key links corpus records to people, accounts, sessions, or bonds. |
| Source bond | **Never recorded** |

Because no link is kept, **nobody, including Solaura staff, can use the corpus to find which person or bond a record came from**, and Solaura cannot find a specific person's records on request.

## 4. Use

- The corpus is used to improve Solaura's **internal** models for understanding and supporting emotional states.
- Mental-health content is treated as **highly sensitive** even after de-identification.
- **Today, corpus data is for internal use only.** No corpus data has been shared, licensed, or sold to anyone.

## 5. Planned future use: licensing to vetted research organisations

**Status: Planned. Not active.** Solaura plans to eventually license parts of the de-identified corpus to vetted outside research organisations. The design is below.

- **No data has been shared, licensed, or sold today.**
- **No research partners exist yet.**

### 5.1 Consent (Tier 2)

| Principle | Design |
|-----------|--------|
| Separate | A **second, separate opt-in**, distinct from Tier 1 internal use |
| Both parties | Both the **client** and the **therapist** must each opt in |
| Default | **Off** |
| Revocable | Either person can withdraw at any time. Withdrawal applies to **future** sessions and future outbound batches. |
| Independent | Choosing internal use (Tier 1) does **not** mean agreeing to external sharing. People can choose Tier 1 without Tier 2. |
| Eligibility | Only conversations from sessions where **both** people had opted in to **both** tiers would be eligible for external sharing |

The same limit applies as for Tier 1. Because the corpus keeps no link to people, records already contributed cannot be individually located or recalled. Solaura will stop including future contributions after withdrawal.

### 5.2 Safeguards any future sharing would require

Solaura will not share or license any corpus data externally unless all of the following are in place:

1. **A data use agreement** with each recipient that:
   - **bans re-identification** and any attempt to link records to people
   - limits use to **research only**
   - bans **onward transfer** to any other party
   - gives Solaura **audit rights**
2. **Access in a controlled environment where feasible**, so recipients work with the data without taking copies away.
3. **Stricter de-identification and human review for each outbound batch**, on top of the standard pipeline, before anything leaves Solaura.
4. **Legal review under the DPDP Act** before any sharing begins.
5. **A published list of partners** in this repository, so anyone can see which organisations have access.

### 5.3 What we are not claiming

- No partners, agreements, revenue, or ethics boards exist today, and none are implied here.
- This section will be updated, and a CHANGELOG entry added, before any external sharing begins.

## 6. Limits

- **De-identified, not anonymous.** The pipeline lowers the risk of re-identification but cannot guarantee it is impossible.
- **Automated scrubbing can miss context.** A combination of details (a distinctive situation, a sequence of events, an unusual phrase) can point to a person even when names and contact details are gone. Generalisation, the rescan, and human review reduce this risk. They do not eliminate it.
- **Saved contributions cannot be individually deleted.** The link to the contributor is never stored, so a specific person's records cannot be located afterwards. Withdrawal stops future contributions only. This is stated at the time consent is requested.
- **No metrics claimed.** We do not publish or claim detection rates, accuracy figures, certifications, or third-party audits for this pipeline.

## 7. Jurisdiction

- **Entity:** Solaura Technologies, Inc. (Delaware C-corporation).
- **India (DPDP Act, 2023):** contribution is based on **specific consent** for this purpose, requested separately from other consents and withdrawable for future sessions. Any future external sharing (Tier 2) would need its own specific consent and legal review under the DPDP Act first.
- **HIPAA:** Solaura makes **no HIPAA claims.**
- See [legal-notes.md](legal-notes.md) for the wider regulatory position.

## 8. Implementation status

| Component | Status |
|-----------|--------|
| Client opt-in (default off, revocable) | Rolling out |
| Therapist opt-in (default off, revocable) | Rolling out |
| Both-party eligibility check | Rolling out |
| Automated removal of direct identifiers | Rolling out |
| Quasi-identifier generalisation | Rolling out |
| Residual-PII rescan and quarantine | Rolling out |
| Separate corpus store with no identifiers or mapping | Rolling out |
| Human review of random samples | Rolling out |
| Tier 2 opt-in (external research sharing) | Planned, not active |
| External licensing to vetted research organisations | Planned, not active. No data shared; no partners. |

This table will be updated as each component goes live. See [CHANGELOG](../CHANGELOG.md).
