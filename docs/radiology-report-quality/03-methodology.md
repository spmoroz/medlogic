# Methodology · RRQ-FR

Technical approach to quality & appropriateness analysis of French trauma X-ray reports.
Grounded in the literature (see [`references.md`](references.md)): French biomedical encoders
(CamemBERT-bio, DrBERT), NLP for follow-up / actionable-finding detection, LLM-as-judge for
radiology report evaluation, and Gleamer BoneView's published intended use.

---

## 1. Principle: two tracks by dimension nature

| Track | Dimensions | Tool | Why |
|-------|-----------|------|-----|
| **Hard** | Q2, Q3, B1, B2, SRC, SGE, MFR, ARDR | Fine-tuned encoder + rules | Well-defined; pilot labels available; cheap at inference |
| **Soft** | Q1, SEF, AMI, CQA, CCS, CPZ | LLM-as-judge with rubric | Semantic reasoning, no crisp rule |

**Source of truth for QC rules:** the Groupe 3R "Checklist Comptes Rendus Radiologiques"
(Mars 2013). Metric definitions live in [`qc-metrics-spec.md`](qc-metrics-spec.md); the
labeling rubric in [`05-annotation-guideline.md`](05-annotation-guideline.md) (Parts A/B/C).

**Data strategy:** bootstrap from the completed pilots (Project 1: 888 BoneView doubt alerts,
canonical, scaling to > 10k incl. AI-positive/negative; Project 2: 50 QC-labeled reports,
single radiologist). LLM extends weak labels; hard dimensions are distilled into a cheap
encoder on the frozen gold.

## 2. Preprocessing (shared)

1. **De-identification** - before any processing (RGPD): names, DOB, IDs, addresses.
2. **Section segmentation** - split into `Indication / Renseignements`, `Technique`,
   `Description / Résultats`, `Conclusion / Impression`, via header regex (French) + a
   fallback line classifier. Q1/Q2/Q3 read the Conclusion; B2 reads the Conclusion; B1
   reads the **order/prescription** + exam metadata.
3. **Normalization** - units, abbreviations, lowercasing for rules.

## 3. Models

### 3.1 Encoders (hard track)
- **CamemBERT-bio** (`almanach/camembert-bio-base`) - French biomedical, +2.54 F1 avg on 5 biomedical NER tasks vs camembert-base.
- **DrBERT** (`qanastek/DrBERT`) - French RoBERTa on the NACHOS medical corpus.
- ⚠️ **512-token limit** - process per section (Conclusion/order text is short) or use a long-context variant (**ModernCamemBERT-bio**).
- CamemBERT-bio vs DrBERT decided empirically on dev.

### 3.2 LLM (soft track)
- **Open-weights, on-prem** (RGPD): Mistral (strong French) or Qwen.
- Inference: vLLM (throughput) or Ollama (pilot simplicity).
- **Determinism:** low temperature, fixed seed; model version frozen and logged.
- **Structured output:** JSON schema per call (label + rationale + extracted elements).

## 4. Method per dimension

### Q1 - Ambiguity (ordinal/binary) · LLM-as-judge
- Rubric-anchored scoring: the judge must mark concrete signals (definite result? hedge
  resolved by an action? contradiction?) rather than a gut "score".
- Trauma lens: is the fracture/eligible-finding question answered or left open?
- Validate with κ + MAE vs consensus labels from the 50-report single-radiologist pilot (extended).

### Q2 - Follow-up present (binary) · encoder + rules
- Rule baseline: an *action term* near an *imaging/next-step term* (see rubric FR cues).
- Model: fine-tuned CamemBERT-bio/DrBERT. Hybrid rules+model usually beats either alone.

### Q3 - Follow-up completeness (slots) · encoder + rules
- If Q2 = yes, extract the **checklist §7** slots: `delai` (timeframe), `modalite`, `attendu`
  (what is expected of the exam). Missing any required slot ⇒ incomplete.
- Output which slots are missing (actionable feedback). *(This replaces the earlier
  region/condition slots; the checklist is the source of truth.)*

### B1 - Order conformity to BoneView intended use · rules + encoder
- Inputs: order/prescription text + exam metadata (modality, body region, patient age).
- Decision = modality is XR **and** region ∈ {limbs, pelvis, thoraco-lumbar spine, rib cage}
  **and** age ≥ 2 y. Emit reason code on out-of-scope (`R_modality`/`R_region`/`R_age`).
- Region mapping (exam code / free text → body region) is largely rule/lexicon-driven;
  an encoder classifier handles messy free-text orders.
- Watch the **rib cage vs chest** and **cervical spine** edge cases (in the rubric).

### B2 - Eligible finding in the conclusion · encoder / LLM extraction
- Detect BoneView-eligible finding types (**fracture, dislocation, effusion, bone lesion**)
  in the Conclusion, with body region.
- Negation handling is essential ("*pas de fracture*"). Region check reuses B1's mapping to
  mark out-of-scope findings (e.g. skull fracture → not BoneView-eligible, flagged separately).

## 5. BoneView appropriateness & concordance (why B1+B2)

Cross-tabulate per exam:

| | B2 eligible finding present | B2 absent |
|--|--|--|
| **B1 in-scope** | BoneView should run AND a catchable finding exists → concordance check vs BoneView output | BoneView should run, no eligible finding → expected negative |
| **B1 out-of-scope** | Eligible-type finding but outside intended use → coverage gap note | Correctly no BoneView use |

This yields two operational metrics: **deployment appropriateness** (are in-scope exams the
ones BoneView runs on?) and **concordance** (does BoneView agree with the radiologist's
report on eligible findings?).

### ARDR - AI-Report Discrepancy Rate (region-level)
- Studies where BoneView flags a finding (confidence > 0.7) **not** mentioned in Findings/Conclusion, by body region.
- BoneView returns a **case-level** label without localisation, so ARDR is **region-level, not lesion-level**. Define ">0.7" against the POSITIVE (>0.9) / DOUBT (0.5-0.9) bands. Text match is negation-aware. "Chest" = **rib cage** (BoneView does not read lung).

## 6. Report QC metrics (Project 2, checklist-grounded)

The QC metric set (SRC, SGE, SEF, AMI, CQA, CCS, CPZ, MFR, plus the QCS composite) is
specified in [`qc-metrics-spec.md`](qc-metrics-spec.md), with each rule mapped to a Groupe 3R
checklist section and each metric **tiered** (existing pilot pool vs next level). Key method points:

- **CCS (0-5)** = share of applicable §6 conclusion items satisfied.
- **MFR** = uncertainty present **and** follow-up incomplete (missing any of délai / modalité / attendu, §7).
- **SEF** = laterality mismatch (piloted) + within-report / vs-prior discordance (§8, next level) + dangling references.
- Non-checklist heuristics (forbidden-phrase blacklist, "no verbosity") are **not** hard rules.

### Slicing & ranking
- Every metric is computed in **5 slices**: global / per radiologist / per region / per modality / per day-of-week (min-N ≥ 20, Wilson CIs; day-of-week exploratory).
- **QCS radiologist ranking is internal-only, modality-sliced, normalized per 100 reports of the same modality**, with empirical-Bayes shrinkage. Used for improvement, not punishment.

## 7. Evaluation

| Dimension | Primary metrics |
|-----------|-----------------|
| Q1 / AMI (ordinal/flag) | Quadratic-weighted κ (model↔expert), MAE, ±1 accuracy |
| Q2 (binary) | Precision, Recall, F1, AUC |
| Q3 (completeness) | F1 on "incomplete"; per-slot (délai/modalité/attendu) accuracy |
| B1 (in-scope) | F1; per-reason-code breakdown; region-mapping accuracy |
| B2 (finding) | F1 on presence; finding-type + region accuracy; negation error rate |
| ARDR | region-level discrepancy rate + Wilson CIs, per region |
| SRC / SGE / SEF / CQA / CCS / CPZ | precision/recall (or κ / MAE for CCS) vs gold |
| Label validation | κ LLM↔radiologist (target ≥ 0.61 / 0.81), confusion matrix |
| Project 1 (>10k) | per-output PPV / NPV (report reference; no image truth) |

**Fair-evaluation principles:** frozen test split; consensus labels; model ceiling bounded
by inter-annotator agreement; error analysis + LLM-judge calibration; the incorporation-bias
caveat stated wherever concordance is reported.

## 8. Human-in-the-loop & retraining

- MVP runs as an **assistant**: it flags, the radiologist decides.
- Confirmations/rejections grow the gold set; active learning prioritizes uncertain cases.
- Scheduled encoder retraining as labels accumulate; monitor score drift over time.

## 9. Why this design

- **No training from scratch** - ready French biomedical encoders.
- **Open-weights + on-prem** - patient data, RGPD.
- **LLM first, distill later** - pilots already provide gold; LLM scales weak labels.
- **Checklist-grounded QC** - rules trace to the Groupe 3R checklist, auditable not subjective.
- **Rubric-anchored judging** - reproducibility over free-form scoring.
- **BoneView intended use as an explicit, coded yardstick** - B1/B2/ARDR are auditable, not vibes.
- **Report reference, stated honestly** - concordance not accuracy; per-output PPV/NPV at >10k.
- **Trauma XR narrow scope** - smaller vocabulary → higher accuracy on limited data.
