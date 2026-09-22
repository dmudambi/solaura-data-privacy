# Training & Dataset Transparency

**Honest disclosure of how Solaura's AI insight model was developed and what data informed it.**

> This document is written for therapists, clinicians, and external readers who want to understand how our AI generates session insights.

## Executive Summary

Solaura Therapy's insight generation uses **prompt-tuned foundation models** (currently OpenAI GPT-4) with carefully engineered prompts. We do **not** fine-tune model weights on private patient data. No therapy transcripts from users are used to train models.

## How Our AI Works

### What We Do

| Approach | Description |
|----------|-------------|
| **Prompt engineering** | We craft detailed system prompts that guide the foundation model to generate therapeutic insights |
| **Litmus inventory evaluation** | We test prompts against evaluation scenarios to verify quality and safety |
| **Foundation model API** | We use commercial LLM APIs (OpenAI) which have their own training data |

### What We Do NOT Do

| Approach | We Do NOT Do This |
|----------|-------------------|
| ❌ Fine-tuning on private data | We do not train or fine-tune model weights using private user transcripts |
| ❌ Weight training | We do not modify the underlying model weights with any user data |
| ❌ Data collection for training | User sessions are not collected as training data for models |

## Evaluation Domains

Our prompt engineering was tested against scenarios in the following high-level conversation domains:

| Domain Category | Description |
|-----------------|-------------|
| **Personal reflection** | Self-exploration conversations about life events, decisions, and feelings |
| **Therapy-adjacent debrief patterns** | Structured reflection formats similar to therapy session summaries |
| **Goal-oriented dialogue** | Conversations focused on setting and working toward personal objectives |
| **Emotional processing** | Discussions involving identification and exploration of emotional states |

> **Transparency note**: These are general domain categories for prompt evaluation. We do not publish specific evaluation datasets. If specific public datasets (such as AnnoMI-style motivational interviewing corpora) were used in evaluation, this section will be updated to reflect that accurately. Currently, evaluation uses internally-developed litmus scenarios rather than external published datasets.

## Foundation Model Training (OpenAI)

The underlying GPT-4 model was trained by OpenAI on their proprietary training data. Key points:

- Solaura does not have access to or control over OpenAI's training data
- OpenAI's training data predates any Solaura user data
- Per OpenAI's API data usage policy, **API inputs are not used to train their models** by default
- See [OpenAI's data usage policy](https://openai.com/policies/api-data-usage-policies) for their current practices

## How Solaura Uses AI in Practice

### For Therapists and Clinicians

Solaura Therapy is designed as a **clinician support tool**, not a replacement for clinical judgment:

| Feature | How It Works |
|---------|--------------|
| **Session debriefs** | AI generates draft summaries and insights from session transcripts |
| **Clinician review** | All AI-generated content is presented for clinician review before use |
| **Edit/reject/approve** | Clinicians can edit, reject, or approve any AI-generated insight |
| **Private until shared** | AI insights remain private to the clinician until explicitly shared |

### Data Flow

```
Session Recording → Transcript → AI Generates Draft Insight
                                         ↓
                               Clinician Reviews
                                         ↓
                              Edit / Reject / Approve
                                         ↓
                        Share with client (if approved)
```

### Privacy by Default

- AI-generated insights are **private until explicitly shared** by the clinician
- Clients do not see AI outputs unless the clinician chooses to share
- Clinicians maintain full control over what is shared

## What We Explicitly Do NOT Claim

| Claim | Our Position |
|-------|--------------|
| "Trained on therapy transcripts" | ❌ **False** — We use prompt engineering, not weight training |
| "Fine-tuned on patient data" | ❌ **False** — No fine-tuning occurs |
| "HIPAA compliant" | ❌ **Not claimed** — See [Legal Notes](legal-notes.md) |
| "Model trained on your data" | ❌ **False** — Foundation model training is separate from our service |

## Technical Accuracy Notes

### Terminology Clarification

- **Prompt tuning / prompt engineering**: Crafting input prompts that guide a pre-trained model's outputs. Does not modify model weights.
- **Fine-tuning**: Training a model's weights on additional data. We do not do this.
- **Foundation model**: A large pre-trained model (like GPT-4) that we use via API without modification.

### Evaluation vs. Training

Our "litmus inventory" approach means we **evaluate** prompt effectiveness against test scenarios. This is distinct from **training**, which would involve updating model weights. Evaluation helps us improve prompts; it does not train the model.

## Updates to This Document

This document will be updated if our AI approach changes, specifically:

- If we adopt fine-tuning in the future (with appropriate disclosures)
- If we change foundation model providers
- If we use specific public datasets in evaluation (will be cited)

---

## Questions?

For questions about our AI practices:
- Open an issue on this repository
- Contact us through the app

*Last updated: See git history for this file*

---

*Maintained by Solaura Technologies, Inc.*
