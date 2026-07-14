# Runbook (Order of actions) · RRQ-FR

Step-by-step operational runbook for the 3-month plan. Each step: **who → what → output**.
Mark status inline (☐ / ☑). Dimensions: Q1-Q3 (report quality) · B1/B2/ARDR (BoneView
adjudication) · Part C checklist QC (SRC, SGE, SEF, AMI, CQA, CCS, CPZ, QCS). Two goals:
operations + two papers. QC rules follow the Groupe 3R checklist.

---

## Block A - Setup (before data)

| # | Action | Owner | Output | ☐ |
|---|--------|-------|--------|---|
| A1 | Kick-off, assign roles (passport §4) | Sponsor | Roles fixed | ☐ |
| A2 | Confirm RGPD legal basis | DPO + lead | Approval to process de-identified data | ☐ |
| A3 | Freeze rubric Parts A/B/C vs Groupe 3R checklist | Clinical lead | `05-annotation-guideline.md` + `qc-metrics-spec.md` signed | ☐ |
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
| C2 | Add Q3 (délai/modalité/attendu) + Part C QC labels (SRC/SGE/SEF/AMI/CQA/CCS/CPZ) | Annotators | Completed QC labels | ☐ |
| C3 | Second-radiologist pass on a subset (QC gold is currently single-rater) | 2 radiologists | Double-labeled subset | ☐ |
| C4 | Compute κ (LLM↔radiologist and human↔human) per metric | ML engineer | κ / confusion-matrix report | ☐ |
| C5 | Adjudicate disagreements | Clinical lead | Consensus labels | ☐ |
| C6 | If κ(Q1) < target → refine rubric, re-label | Lead | Updated rubric | ☐ |
| C7 | Freeze test split | ML engineer | train/dev/test | ☐ |

## Block D - LLM baseline (Weeks 3–4)

| # | Action | Owner | Output | ☐ |
|---|--------|-------|--------|---|
| D1 | Deploy open-weights LLM on-prem (vLLM/Ollama) | ML engineer | Working inference | ☐ |
| D2 | Judge prompts Q1-Q3 + Part C QC (SRC/SGE/SEF/AMI/CQA/CCS/CPZ), JSON output | ML engineer | Prompts v1 | ☐ |
| D3 | B1 intended-use classifier (rules + LLM fallback) | ML engineer | B1 baseline | ☐ |
| D4 | B2 finding extraction + ARDR region-level report-vs-BoneView match | ML engineer | B2 + ARDR baseline | ☐ |
| D5 | Score on dev vs gold; iterate (version prompts) | ML engineer | Baseline metrics table | ☐ |

## Block E - Specialized models (Weeks 5–8)

| # | Action | Owner | Output | ☐ |
|---|--------|-------|--------|---|
| E1 | LLM weak-labeling of large corpus (Q2,Q3,B1,B2, rule-heavy QC) | ML engineer | Weak labels | ☐ |
| E2 | Assemble Project 1 **>10k** cohort (add AI-positive + AI-negative to 888) | ML + data | Device-output-balanced cohort | ☐ |
| E3 | Fine-tune encoder - Q2 (follow-up present) | ML engineer | Model Q2 | ☐ |
| E4 | Train Q3 slot extraction (**délai/modalité/attendu**, §7) + completeness rule | ML engineer | Model Q3 + rules | ☐ |
| E5 | Train B1 in-scope classifier + region mapping + reason codes | ML engineer | Model B1 | ☐ |
| E6 | Train B2 eligible-finding detector + negation handling | ML engineer | Model B2 | ☐ |
| E7 | SRC/SGE/MFR/ARDR rule+encoder; validate SEF/CQA/CCS/CPZ (LLM judge) vs gold | ML engineer | QC metric models | ☐ |
| E8 | Evaluate on test; per-output PPV/NPV on >10k | ML engineer | F1 + PPV/NPV | ☐ |

## Block F - Evaluation & MVP (Weeks 9–12)

| # | Action | Owner | Output | ☐ |
|---|--------|-------|--------|---|
| F1 | Final run on frozen test | ML engineer | Raw predictions | ☐ |
| F2 | Compile KPI (F1, κ, MAE, type/region acc.) | ML engineer | Quality report | ☐ |
| F3 | Calibrate LLM judge (reproducibility, bias) | ML engineer | Calibration report | ☐ |
| F4 | Error analysis (FP/FN typology; rib vs chest, cervical) | ML + lead | Weak-spot list | ☐ |
| F5 | Assemble pipeline + flagged quality report | ML engineer | MVP pipeline | ☐ |
| F6 | Dashboards: BoneView appropriateness/concordance (B1×B2×ARDR×device) + QC metrics **in slices** | ML engineer | Dashboards | ☐ |
| F7 | QCS ranking (internal, modality-sliced, per-100-same-modality, shrinkage) | ML engineer | Ranking module | ☐ |
| F8 | Review UI (flag + rationale → confirm/reject) | ML engineer | Review tool | ☐ |
| F9 | Acceptance audit: ≥70% flags valid | Clinical lead | Acceptance report | ☐ |
| F10 | Draft two papers (BoneView adjudication; report QC) | Lead + ML | Two paper drafts | ☐ |
| F11 | Human-in-the-loop + retraining process | Lead + ML | Process doc | ☐ |

---

## Status checkpoints (for standups)

- **CP1 (end W1):** rubric (Parts A/B/C, checklist) + legal basis + segmentation ready?
- **CP2 (end W2):** gold consolidated, two-radiologist κ measured, test frozen?
- **CP3 (end W4 · M1):** LLM baseline numbers for Q1-Q3, B1/B2/ARDR, and the QC metric set?
- **CP4 (end W8 · M2):** encoders hit F1 targets; >10k cohort + per-output PPV/NPV?
- **CP5 (end W12 · M3):** ops pilot + QCS dashboard + audit + two paper drafts?
