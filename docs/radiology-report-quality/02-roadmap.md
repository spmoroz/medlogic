# Roadmap · RRQ-FR (3 months)

A 12-week plan for a team of 1 ML engineer + clinical lead + 1–2 annotators. Because both
pilots are **already done** (Q1/Q2 on 50 trauma XR reports (single radiologist); B1/B2 on 888 reports), the plan
starts from *consolidation*, not from scratch.

```
Month 1  ──▶  Month 2  ──▶  Month 3
Consolidate     Specialized      Evaluate + MVP
gold + baseline  models           + human-in-the-loop
```

Dimensions: **Q1** ambiguity · **Q2** follow-up present · **Q3** follow-up completeness ·
**B1** order in-scope · **B2** eligible finding.

---

## MONTH 1 - Consolidate gold + LLM baseline

### Week 1 - Definitions & data hygiene
- Freeze the rubric ([`05-annotation-guideline.md`](05-annotation-guideline.md)) against pilot experience.
- Confirm RGPD legal basis with DPO (blocker). De-identify report + order corpus.
- Section segmentation (Indication / Technique / Description / Conclusion).
- **DoD:** rubric signed; ≥95% sections parsed; DPO ok.

### Week 2 - Gold consolidation
- Ingest pilot labels: 50 trauma XR reports, single radiologist (Q1, Q2) and 888 reports (B1, B2).
- Fill gaps: add Q3 (follow-up completeness) labels where Q2=yes; spot-relabel for consistency with the frozen rubric.
- Second-annotator pass on a subset for **IAA (κ)** on every dimension.
- **DoD:** IAA measured; disagreements adjudicated to consensus; test split frozen.

### Weeks 3–4 - LLM baseline (all dimensions)
- Deploy open-weights LLM on-prem (vLLM/Ollama).
- Judge prompts with **structured JSON output** for Q1, Q2, Q3.
- B1: rule-first intended-use classifier (modality/region/age) + LLM fallback.
- B2: LLM finding extraction (type + region) constrained to BoneView-eligible classes.
- Score on dev vs gold; iterate prompts (version-controlled).
- **Milestone M1:** baseline numbers for Q1–Q3, B1–B2 on dev.

---

## MONTH 2 - Specialized models

### Weeks 5–6 - Weak labeling & data build
- Run LLM baseline over the large unlabeled corpus → weak labels for Q2, Q3, B1, B2.
- Sample-verify weak labels with an annotator; assemble train/dev.
- **DoD:** clean training sets per hard dimension.

### Weeks 7–8 - Fine-tune encoders
- `CamemBERT-bio` / `DrBERT`:
  - **Q2** binary follow-up classifier.
  - **Q3** slot extraction (modality/timeframe/region/condition) + completeness rule.
  - **B1** in-scope classifier (region/modality/age) - hybrid with rules + reason codes.
  - **B2** eligible-finding detector (fracture/dislocation/effusion/lesion) + region tagging.
- Handle 512-token limit (per-section / long-context variant).
- **Milestone M2:** Q2 F1 ≥ 0.90, Q3 F1 ≥ 0.80, B1 F1 ≥ 0.90, B2 F1 ≥ 0.85 on frozen test (or documented gap + plan).

---

## MONTH 3 - Evaluation + MVP

### Weeks 9–10 - Evaluation & calibration
- Final run on the **frozen test split**.
- Metrics per task: F1/P/R (Q2, Q3, B1, B2), κ + MAE (Q1); B2 also region/type accuracy.
- Calibrate the LLM judge (over-rating, paraphrase robustness, reproducibility).
- Error analysis: FP/FN typology; where the system is weakest (e.g. rib-cage vs chest for B1).
- **DoD:** quality report agreed with clinical lead.

### Weeks 11–12 - Pipeline + human-in-the-loop
- Single pipeline: de-identification → segmentation → {encoders Q2/Q3/B1/B2, LLM judge Q1} → aggregated report with flags.
- BoneView cross-tab: B1 (should it run) × B2 (eligible finding) × BoneView output → appropriateness & concordance dashboard.
- Review UI: radiologist sees flag + rationale → confirm/reject; confirmations grow the gold set.
- **Milestone M3 (MVP):** run on a fresh stream; ≥ 70% of flags judged valid on audit; human-in-the-loop process documented.

---

## Milestone summary

| Milestone | When | Key gate |
|-----------|------|----------|
| M1 · Gold + baseline | End of Month 1 | IAA measured; baseline numbers for all dimensions |
| M2 · Models | End of Month 2 | F1 targets met (Q2/Q3/B1/B2) |
| M3 · MVP | End of Month 3 | ≥ 70% flags valid; appropriateness/concordance dashboard |

## Parallelization / fast path

- Report-quality (A) and BoneView (B) tracks run in parallel - different labels, shared pipeline.
- If a demo is needed early: the LLM baseline (Weeks 3–4) already covers all five dimensions
  end-to-end; encoders in Month 2 are the cost/robustness upgrade.
