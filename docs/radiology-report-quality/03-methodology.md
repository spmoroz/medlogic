# Methodology · RRQ-FR

Technical approach to quality & appropriateness analysis of French trauma X-ray reports.
Grounded in the literature (see [`references.md`](references.md)): French biomedical encoders
(CamemBERT-bio, DrBERT), NLP for follow-up / actionable-finding detection, LLM-as-judge for
radiology report evaluation, and Gleamer BoneView's published intended use.

---

## 1. Principle: two tracks by dimension nature

| Track | Dimensions | Tool | Why |
|-------|-----------|------|-----|
| **Hard** | Q2, Q3, B1, B2 | Fine-tuned encoder + rules | Well-defined; pilot labels available; cheap at inference |
| **Soft** | Q1 (ambiguity) | LLM-as-judge with rubric | Semantic, needs "understanding", no crisp rule |

**Data strategy:** bootstrap from the completed pilots. LLM extends weak labels; hard
dimensions are distilled into a cheap encoder on the frozen gold.

## 2. Preprocessing (shared)

1. **De-identification** — before any processing (RGPD): names, DOB, IDs, addresses.
2. **Section segmentation** — split into `Indication / Renseignements`, `Technique`,
   `Description / Résultats`, `Conclusion / Impression`, via header regex (French) + a
   fallback line classifier. Q1/Q2/Q3 read the Conclusion; B2 reads the Conclusion; B1
   reads the **order/prescription** + exam metadata.
3. **Normalization** — units, abbreviations, lowercasing for rules.

## 3. Models

### 3.1 Encoders (hard track)
- **CamemBERT-bio** (`almanach/camembert-bio-base`) — French biomedical, +2.54 F1 avg on 5 biomedical NER tasks vs camembert-base.
- **DrBERT** (`qanastek/DrBERT`) — French RoBERTa on the NACHOS medical corpus.
- ⚠️ **512-token limit** — process per section (Conclusion/order text is short) or use a long-context variant (**ModernCamemBERT-bio**).
- CamemBERT-bio vs DrBERT decided empirically on dev.

### 3.2 LLM (soft track)
- **Open-weights, on-prem** (RGPD): Mistral (strong French) or Qwen.
- Inference: vLLM (throughput) or Ollama (pilot simplicity).
- **Determinism:** low temperature, fixed seed; model version frozen and logged.
- **Structured output:** JSON schema per call (label + rationale + extracted elements).

## 4. Method per dimension

### Q1 — Ambiguity (ordinal/binary) · LLM-as-judge
- Rubric-anchored scoring: the judge must mark concrete signals (definite result? hedge
  resolved by an action? contradiction?) rather than a gut "score".
- Trauma lens: is the fracture/eligible-finding question answered or left open?
- Validate with κ + MAE vs consensus labels from the ~100-case pilot (extended).

### Q2 — Follow-up present (binary) · encoder + rules
- Rule baseline: an *action term* near an *imaging/next-step term* (see rubric FR cues).
- Model: fine-tuned CamemBERT-bio/DrBERT. Hybrid rules+model usually beats either alone.

### Q3 — Follow-up completeness (slots) · encoder + rules
- If Q2 = yes, extract slots (`modality`, `timeframe`, `body_region`, `condition`).
- Missing required slot ⇒ incomplete. Output which slots are missing (actionable feedback).

### B1 — Order conformity to BoneView intended use · rules + encoder
- Inputs: order/prescription text + exam metadata (modality, body region, patient age).
- Decision = modality is XR **and** region ∈ {limbs, pelvis, thoraco-lumbar spine, rib cage}
  **and** age ≥ 2 y. Emit reason code on out-of-scope (`R_modality`/`R_region`/`R_age`).
- Region mapping (exam code / free text → body region) is largely rule/lexicon-driven;
  an encoder classifier handles messy free-text orders.
- Watch the **rib cage vs chest** and **cervical spine** edge cases (in the rubric).

### B2 — Eligible finding in the conclusion · encoder / LLM extraction
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

## 6. Evaluation

| Dimension | Primary metrics |
|-----------|-----------------|
| Q1 (ordinal) | Quadratic-weighted κ (model↔expert), MAE, ±1 accuracy |
| Q2 (binary) | Precision, Recall, F1, AUC |
| Q3 (completeness) | F1 on "incomplete"; per-slot extraction accuracy |
| B1 (in-scope) | F1; per-reason-code breakdown; region-mapping accuracy |
| B2 (finding) | F1 on presence; finding-type + region accuracy; negation error rate |

**Fair-evaluation principles:** frozen test split; consensus labels; model ceiling bounded
by IAA; error analysis + LLM-judge calibration in the report.

## 7. Human-in-the-loop & retraining

- MVP runs as an **assistant**: it flags, the radiologist decides.
- Confirmations/rejections grow the gold set; active learning prioritizes uncertain cases.
- Scheduled encoder retraining as labels accumulate; monitor score drift over time.

## 8. Why this design

- **No training from scratch** — ready French biomedical encoders.
- **Open-weights + on-prem** — patient data, RGPD.
- **LLM first, distill later** — pilots already provide gold; LLM scales weak labels.
- **Rubric-anchored judging** for Q1 — reproducibility over free-form scoring.
- **BoneView intended use as an explicit, coded yardstick** — B1/B2 are auditable, not vibes.
- **Trauma XR narrow scope** — smaller vocabulary → higher accuracy on limited data.
