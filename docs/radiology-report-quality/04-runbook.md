# Runbook (Order of actions) · RRQ-FR

Step-by-step operational runbook for the 3-month plan. Each step: **who → what → output**.
Mark status inline (☐ / ☑). Dimensions: Q1 ambiguity · Q2 follow-up present · Q3 completeness ·
B1 order in-scope · B2 eligible finding.

---

## Block A - Setup (before data)

| # | Action | Owner | Output | ☐ |
|---|--------|-------|--------|---|
| A1 | Kick-off, assign roles (passport §4) | Sponsor | Roles fixed | ☐ |
| A2 | Confirm RGPD legal basis | DPO + lead | Approval to process de-identified data | ☐ |
| A3 | Freeze rubric Q1–Q3, B1–B2 | Clinical lead | `05-annotation-guideline.md` signed | ☐ |
| A4 | Provision on-prem GPU | ML engineer | Inference/training environment | ☐ |
| A5 | Confirm trauma XR export (reports + orders) | Data + lead | Export spec | ☐ |

> **A2 is a blocker.** No data work before DPO approval.

## Block B - Data (Week 1)

| # | Action | Owner | Output | ☐ |
|---|--------|-------|--------|---|
| B1 | Export trauma XR reports + orders from RIS | Data engineer | Raw corpus | ☐ |
| B2 | De-identify (names, DOB, IDs, addresses) | Data engineer | De-identified corpus | ☐ |
| B3 | Manual de-id audit on a sample (n≥50) | Lead | Audit report, 0 leaks | ☐ |
| B4 | Section segmentation | ML engineer | Structured corpus | ☐ |
| B5 | Verify segmentation (≥95% correct) | ML engineer | Segmentation metric | ☐ |
| B6 | Corpus profile (lengths, region mix, age availability) | ML engineer | Profile report | ☐ |

## Block C - Gold consolidation (Week 2)

| # | Action | Owner | Output | ☐ |
|---|--------|-------|--------|---|
| C1 | Ingest pilot labels: 50 single-radiologist (Q1,Q2) + 888 (B1,B2) | ML engineer | Seed gold sets | ☐ |
| C2 | Add Q3 labels where Q2=yes; relabel for rubric consistency | Annotators | Completed A-track labels | ☐ |
| C3 | Second-annotator pass on a subset (all dimensions) | 2 radiologists | Double-labeled subset | ☐ |
| C4 | Compute IAA (κ per dimension) | ML engineer | IAA report | ☐ |
| C5 | Adjudicate disagreements | Clinical lead | Consensus labels | ☐ |
| C6 | If κ(Q1) < target → refine rubric, re-label | Lead | Updated rubric | ☐ |
| C7 | Freeze test split | ML engineer | train/dev/test | ☐ |

## Block D - LLM baseline (Weeks 3–4)

| # | Action | Owner | Output | ☐ |
|---|--------|-------|--------|---|
| D1 | Deploy open-weights LLM on-prem (vLLM/Ollama) | ML engineer | Working inference | ☐ |
| D2 | Judge prompts Q1/Q2/Q3 (JSON output) | ML engineer | Prompts v1 | ☐ |
| D3 | B1 intended-use classifier (rules + LLM fallback) | ML engineer | B1 baseline | ☐ |
| D4 | B2 finding extraction constrained to eligible types + region | ML engineer | B2 baseline | ☐ |
| D5 | Score on dev vs gold; iterate (version prompts) | ML engineer | Baseline metrics table | ☐ |

## Block E - Specialized models (Weeks 5–8)

| # | Action | Owner | Output | ☐ |
|---|--------|-------|--------|---|
| E1 | LLM weak-labeling of large corpus (Q2,Q3,B1,B2) | ML engineer | Weak labels | ☐ |
| E2 | Sample-verify weak labels | Annotator | Clean train | ☐ |
| E3 | Fine-tune encoder - Q2 (follow-up present) | ML engineer | Model Q2 | ☐ |
| E4 | Train slot extraction + completeness rule - Q3 | ML engineer | Model Q3 + rules | ☐ |
| E5 | Train B1 in-scope classifier + region mapping + reason codes | ML engineer | Model B1 | ☐ |
| E6 | Train B2 eligible-finding detector + negation handling | ML engineer | Model B2 | ☐ |
| E7 | Hybrid rules+model; handle 512-token limit | ML engineer | Final hard models | ☐ |
| E8 | Evaluate on test | ML engineer | F1 Q2/Q3/B1/B2 | ☐ |

## Block F - Evaluation & MVP (Weeks 9–12)

| # | Action | Owner | Output | ☐ |
|---|--------|-------|--------|---|
| F1 | Final run on frozen test | ML engineer | Raw predictions | ☐ |
| F2 | Compile KPI (F1, κ, MAE, type/region acc.) | ML engineer | Quality report | ☐ |
| F3 | Calibrate LLM judge (reproducibility, bias) | ML engineer | Calibration report | ☐ |
| F4 | Error analysis (FP/FN typology; rib vs chest, cervical) | ML + lead | Weak-spot list | ☐ |
| F5 | Assemble pipeline + flagged quality report | ML engineer | MVP pipeline | ☐ |
| F6 | BoneView appropriateness/concordance dashboard (B1×B2×BoneView) | ML engineer | Dashboard | ☐ |
| F7 | Review UI (flag + rationale → confirm/reject) | ML engineer | Review tool | ☐ |
| F8 | Acceptance audit: ≥70% flags valid | Clinical lead | Acceptance report | ☐ |
| F9 | Human-in-the-loop + retraining process | Lead + ML | Process doc | ☐ |

---

## Status checkpoints (for standups)

- **CP1 (end W1):** rubric + legal basis + segmentation ready?
- **CP2 (end W2):** gold consolidated, IAA measured, test frozen?
- **CP3 (end W4 · M1):** LLM baseline numbers for all five dimensions?
- **CP4 (end W8 · M2):** encoders hit F1 targets (Q2/Q3/B1/B2)?
- **CP5 (end W12 · M3):** MVP + appropriateness dashboard + audit?
