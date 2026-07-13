# Project Passport · RRQ-FR

**Automated quality & appropriateness analysis of French trauma X-ray reports**

| Field | Value |
|-------|-------|
| **Name** | RRQ-FR — Radiology Report Quality (French) |
| **Owner / sponsor** | MedLogic |
| **Document version** | 2.0 · 2026-07-13 |
| **Status** | Pilots complete → scaling to MVP |
| **Pilot modality** | Trauma X-ray (trauma XR) |
| **Data language** | French |
| **Timeline** | **3 months** to a validated MVP |

---

## 1. Problem & rationale

Trauma X-ray report quality directly affects clinical decisions and patient safety. An
ambiguous conclusion or a missing/incomplete follow-up recommendation is a known source
of diagnostic error and litigation (unresolved actionable findings). In parallel, MedLogic
deploys **Gleamer BoneView** (fracture/dislocation/effusion/bone-lesion AI); its value
depends on being applied to the **right exams** (intended-use conformity) and on measuring
**concordance** with the radiologist's report.

**Hypothesis:** an NLP/LLM system can reproducibly flag quality issues and measure BoneView
appropriateness at full volume (not sampled audit), with agreement close to expert-to-expert
agreement, enabling continuous QA in a human-in-the-loop mode.

## 2. Goal

Deliver and validate an MVP that, on a stream of French trauma XR reports, scores the two
workstreams below and flags reports needing attention, at a quality suitable for
human-in-the-loop operation — **within 3 months**, building on the completed pilots.

## 3. Scope

### Workstream A — Report quality *(pilot: ~100 trauma XR cases labeled)*
- **Q1** Ambiguity of the conclusion
- **Q2** Follow-up recommendation present
- **Q3** Follow-up completeness (missing slots)

### Workstream B — BoneView appropriateness *(pilot: 888 reports labeled)*
- **B1** Order conformity to BoneView intended use (in-scope / out-of-scope)
- **B2** BoneView-eligible finding present in the conclusion (type + region)

### In scope (v1)
Trauma XR modality; French; text only (not images); batch processing of reports & orders;
a quality/appropriateness report with flags for radiologist review.

### Out of scope (v1 — candidates for v2+)
Other modalities (CT/MRI/US); image analysis; real-time RIS/PACS integration during
dictation; auto-rewriting of conclusions; conclusion↔description discordance (optional
extension, not in the pilots); other languages.

## 4. Stakeholders & roles

| Role | Responsibility | Who |
|------|----------------|-----|
| Sponsor | Decisions, budget, priorities | MedLogic |
| Product owner | Requirements, prioritization, acceptance | — |
| Clinical lead (radiologist) | Metric definitions, adjudication | — |
| Annotators (2+ radiologists) | Gold-standard labeling | — |
| ML engineer / DS | Models, pipeline, evaluation | — |
| Data engineer | Export, de-identification, storage | — |
| DPO / compliance | RGPD/GDPR, legal basis | — |

## 5. Data

- **Source:** historical trauma XR reports + imaging orders/prescriptions (RIS export).
- **Already available (pilots):** ~100 trauma XR reports labeled for Q1/Q2; 888 reports
  labeled for B1/B2. These seed the gold sets.
- **Scale-up:** thousands of unlabeled reports available for weak-labeling and encoder training.
- **Privacy:** patient data → mandatory de-identification before processing; on-prem /
  open-weights inference; RGPD legal basis confirmed with DPO **before** data work.

## 6. Technical approach (short)

- **Q2, Q3, B1, B2** (hard): fine-tune `CamemBERT-bio` / `DrBERT` + rules, bootstrapped from pilot labels.
- **Q1** (soft): LLM-as-judge (open-weights) with a structured rubric.
- LLM weak-labeling to scale annotation; distill hard dimensions into a cheap encoder.
- Details — [`03-methodology.md`](03-methodology.md).

## 7. Success criteria (KPI)

| Project metric | MVP target | How measured |
|----------------|-----------|--------------|
| Inter-annotator agreement (IAA) | κ ≥ 0.6 (Q1), κ ≥ 0.75 (Q2, Q3, B1, B2) | On gold set, before modeling |
| Q1 (ambiguity) | κ model↔expert ≥ 0.6 | Agreement vs consensus |
| Q2 (follow-up present) | F1 ≥ 0.90 | vs gold |
| Q3 (follow-up completeness) | F1 ≥ 0.80 | vs gold |
| B1 (order in-scope) | F1 ≥ 0.90 | vs gold |
| B2 (eligible finding) | F1 ≥ 0.85; region/type accuracy reported | vs gold |
| Usefulness | ≥ 70% of flags judged valid on review | Radiologist audit of a sample |

> Thresholds are starting targets, finalized after IAA is measured (model ceiling is bounded
> by human agreement).

## 8. Risks & mitigations

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Low IAA on "ambiguity" | High | Rubric iteration; calibration sessions; lead adjudication |
| 3-month timeline tight | Medium | Pilots already done; reuse labels; narrow to trauma XR |
| Privacy / legal basis | High (blocker) | DPO sign-off before data; de-identification; on-prem |
| LLM judge over-rates / unstable | Medium | Rubric-anchored scoring; fixed seed/temp; calibration |
| BoneView intended-use edge cases (rib cage vs chest, cervical) | Medium | Explicit reason codes + anchor examples in rubric |
| 512-token limit (CamemBERT-bio/DrBERT) | Low-med | Per-section processing; long-context variant (ModernCamemBERT-bio) |

## 9. High-level plan & milestones (3 months)

| Month | Milestone | Deliverable |
|-------|-----------|-------------|
| **M1** | Gold consolidated, definitions frozen, LLM baseline | Frozen gold sets (A+B), IAA report, baseline metrics |
| **M2** | Specialized models trained | Q2/Q3 + B1/B2 encoders + rules, test metrics |
| **M3** | Evaluation + MVP | KPI report, MVP pipeline, review UI, acceptance audit |

Detail — [`02-roadmap.md`](02-roadmap.md).

## 10. Resource budget (rough)

- **People:** clinical lead (part-time), 1–2 radiologist annotators (part-time), ML engineer (core), data engineer (part-time).
- **Infra:** on-prem GPU for open-weights LLM inference + encoder fine-tuning (1× 24–48 GB GPU is enough for the pilot).
- **Software:** open-source (HuggingFace, PyTorch, vLLM/Ollama, labeling tool). No license cost.

## 11. Assumptions & dependencies

- Access to sufficient reports **and orders** is legally cleared.
- ≥ 2 radiologists for labeling + an adjudicator.
- On-prem inference infrastructure provisioned.
- RIS export formats for reports and orders are known and stable.
