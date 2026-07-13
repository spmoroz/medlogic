# Project: Radiology Report Quality Analysis (French) — RRQ-FR

**Pilot modality:** Trauma X-ray (trauma XR)
**Data language:** French · **Owner:** MedLogic
**Timeline:** 3 months to a validated MVP
**Package version:** 2026-07-13

---

## What this is

A project documentation package for building an NLP/LLM system that automatically
assesses the quality and appropriateness of French trauma X-ray reports across two
workstreams that MedLogic has already piloted.

### Workstream A — Report quality *(piloted: ~100 trauma XR cases)*

| Code | Dimension | Task type | Tool |
|------|-----------|-----------|------|
| Q1 | Ambiguity of the conclusion | Binary (+3-point option) | LLM-as-judge |
| Q2 | Follow-up recommendation present | Binary classification | Encoder + rules |
| Q3 | Follow-up completeness | Slot extraction + completeness | Encoder + rules |

### Workstream B — BoneView appropriateness *(piloted: 888 reports)*

| Code | Dimension | Task type | Tool |
|------|-----------|-----------|------|
| B1 | Order conformity to BoneView intended use | In-scope / out-of-scope classification | Rules + encoder |
| B2 | BoneView-eligible finding in the conclusion | Finding presence + type + region | Encoder / LLM extraction |

> **BoneView (Gleamer) intended use** is the yardstick for Workstream B: conventional
> radiographs of **limbs, pelvis, thoracic & lumbar spine, rib cage**, patients **≥ 2 years**,
> detecting **fractures, dislocations, joint effusions, bone lesions**.

## Package contents

| File | Purpose | Audience |
|------|---------|----------|
| [`01-project-passport.md`](01-project-passport.md) | **Project passport**: goal, scope, stakeholders, KPIs, risks, milestones | Lead, sponsor |
| [`02-roadmap.md`](02-roadmap.md) | **Roadmap**: 3-month plan, phases, milestones, deliverables | Whole team |
| [`03-methodology.md`](03-methodology.md) | **Methodology**: technical approach, models, LLM-as-judge, evaluation | ML engineers, DS |
| [`04-runbook.md`](04-runbook.md) | **Order of actions**: step-by-step operational runbook | Doers |
| [`05-annotation-guideline.md`](05-annotation-guideline.md) | **Annotation rubric** with trauma XR anchor examples (Q1–Q3, B1–B2) | Radiologist annotators |
| [`06-working-forms.md`](06-working-forms.md) | **Working forms** for colleagues + ready-to-use CSV templates | Annotators, data curator |
| [`slides.html`](slides.html) | **5-slide executive summary** (MedLogic brand) | Sponsor, stakeholders |
| [`references.md`](references.md) | Literature and BoneView sources grounding the plan | All |

## Key principle

> **Definitions and the gold standard come before the model.** Both pilots have already
> produced labeled data (Q1/Q2 on ~100 trauma XR cases; B1/B2 on 888 reports). The 3-month
> plan consolidates those into frozen gold sets, then bootstraps models and evaluates.

## Stack in one line

- **Soft dimensions (Q1):** LLM-as-judge on open-weights model (Mistral / Qwen) with a strict rubric and structured output.
- **Hard dimensions (Q2, Q3, B1, B2):** fine-tuned `CamemBERT-bio` / `DrBERT` + rules, bootstrapped from the pilot labels.
- **Privacy:** on-prem / open-weights (patient data, RGPD/GDPR).
