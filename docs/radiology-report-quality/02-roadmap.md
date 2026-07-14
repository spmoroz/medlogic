# Roadmap · RRQ-FR (3 months)

A 12-week plan for a team of 1 ML engineer + clinical lead + 1–2 annotators. Because both
pilots are **already done** (Q1/Q2 on 50 trauma XR reports (single radiologist); B1/B2 on 888 reports), the plan
starts from *consolidation*, not from scratch.

```
Month 1  ──▶  Month 2  ──▶  Month 3
Consolidate     Specialized      Evaluate + MVP
gold + baseline  models           + human-in-the-loop
```

Dimensions: **Q1-Q3** (ambiguity, follow-up present, completeness) · **B1/B2/ARDR** (BoneView
adjudication) · **Part C** checklist QC (SRC, SGE, SEF, AMI, CQA, CCS, CPZ, QCS). QC rules
follow the Groupe 3R checklist. Two goals: **operations** (deploy both tools) + **publication**
(two papers). Project 1 scales the 888 doubt-alert cohort to **> 10k** incl. AI-positive/negative.

---

## MONTH 1 - Consolidate gold + LLM baseline

### Week 1 - Definitions & data hygiene
- Freeze the rubric ([`05-annotation-guideline.md`](05-annotation-guideline.md), Parts A/B/C) and the QC metric spec against the **Groupe 3R checklist**.
- Confirm RGPD legal basis with DPO (blocker). De-identify report + order corpus + BoneView output.
- Section segmentation (Correspondants / Titre / Indication / Technique / Findings / Conclusion).
- **DoD:** rubric signed; ≥95% sections parsed; DPO ok.

### Week 2 - Gold consolidation
- Ingest pilot labels: 50 QC reports (single radiologist; Q1/Q2 + Part C) and 888 BoneView doubt alerts (B1/B2).
- Fill gaps: add Q3 (délai/modalité/attendu) and next-level QC labels (CQA/CCS/CPZ/SEF discordance).
- **Two-radiologist pass** on a subset for **κ (LLM↔radiologist and human↔human)** - the QC gold is currently single-rater.
- **DoD:** κ measured; disagreements adjudicated to consensus; test split frozen.

### Weeks 3–4 - LLM baseline (all dimensions)
- Deploy open-weights, judge-grade LLM on-prem (vLLM/Ollama).
- Judge prompts with **structured JSON output** for Q1-Q3 and the checklist QC metrics (SRC/SGE/SEF/AMI/CQA/CCS/CPZ).
- B1: rule-first intended-use classifier (modality/region/age) + LLM fallback. B2: eligible-finding extraction. ARDR: region-level report-vs-BoneView match.
- Score on dev vs gold; iterate prompts (version-controlled).
- **Milestone M1:** baseline numbers for Q1-Q3, B1/B2/ARDR, and the QC metric set on dev.

---

## MONTH 2 - Specialized models

### Weeks 5–6 - Weak labeling, data build & Project 1 scale-up
- Run LLM baseline over the large unlabeled corpus → weak labels for Q2/Q3/B1/B2 and rule-heavy QC metrics.
- **Project 1 extension:** assemble the **> 10k** device-output-balanced cohort (add AI-positive & AI-negative to the 888 doubt alerts).
- Sample-verify weak labels; assemble train/dev.
- **DoD:** clean training sets; >10k cohort assembled.

### Weeks 7–8 - Fine-tune encoders
- `CamemBERT-bio` / `DrBERT`:
  - **Q2** binary follow-up classifier.
  - **Q3** slot extraction (**délai / modalité / attendu**, checklist §7) + completeness rule.
  - **B1** in-scope classifier (region/modality/age) - hybrid with rules + reason codes.
  - **B2** eligible-finding detector (fracture/dislocation/effusion/lesion) + region tagging.
  - **SRC / SGE / MFR / ARDR** rule+encoder; SEF/CQA/CCS/CPZ stay LLM-judge (validate vs gold).
- Handle 512-token limit (per-section / long-context variant).
- **Milestone M2:** Q2 F1 ≥ 0.90, Q3 F1 ≥ 0.80, B1 F1 ≥ 0.90, B2 F1 ≥ 0.85; per-output PPV/NPV on the >10k cohort (or documented gap + plan).

---

## MONTH 3 - Evaluation + MVP

### Weeks 9–10 - Evaluation & calibration
- Final run on the **frozen test split**; report the incorporation-bias caveat (report reference, no image truth).
- Metrics per task: F1/P/R (Q2/Q3/B1/B2, SRC/SGE/SEF/CQA/CPZ), κ+MAE (Q1/CCS), region-level ARDR, per-output PPV/NPV (>10k), κ label-validation.
- Calibrate the LLM judge (over-rating, paraphrase robustness, reproducibility).
- Error analysis: FP/FN typology; weakest points (rib-cage vs chest for B1; SEF/CQA reasoning).
- **DoD:** quality report agreed with clinical lead.

### Weeks 11–12 - Deploy + publish
- Single pipeline: de-identification → segmentation → {encoders + LLM judge} → aggregated report with flags.
- Dashboards: BoneView appropriateness/concordance (B1×B2×ARDR×device) and QC metrics **in slices** (global/radiologist/region/modality/day-of-week) with the **internal, modality-sliced, per-100-same-modality QCS ranking**.
- Review UI: radiologist sees flag + rationale → confirm/reject; confirmations grow the gold set.
- **Two paper drafts:** (1) LLM adjudication of BoneView; (2) LLM-based report QC.
- **Milestone M3:** operations pilot in the reporting workflow; ≥ 70% flags valid on audit; two submittable drafts.

---

## Milestone summary

| Milestone | When | Key gate |
|-----------|------|----------|
| M1 · Gold + baseline | End of Month 1 | κ measured (incl. two-radiologist); baseline for all metrics |
| M2 · Scale + models | End of Month 2 | F1 targets (Q2/Q3/B1/B2); >10k cohort + per-output PPV/NPV |
| M3 · Deploy + publish | End of Month 3 | ops pilot; ≥ 70% flags valid; QCS dashboard; two paper drafts |

## Parallelization / fast path

- Project 1 (BoneView adjudication) and Project 2 (QC) run in parallel - different labels, shared pipeline.
- If a demo is needed early: the LLM baseline (Weeks 3-4) already covers all dimensions
  end-to-end; encoders and the >10k extension in Month 2 are the cost/robustness/power upgrade.
