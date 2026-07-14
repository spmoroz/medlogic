# Project Passport · RRQ-FR

**LLM-based quality & appropriateness intelligence for French trauma X-ray reports**

| Field | Value |
|-------|-------|
| **Name** | RRQ-FR - Radiology Report Quality (French) |
| **Owner / sponsor** | the group |
| **Document version** | 3.0 · 2026-07-14 |
| **Status** | Pilots complete → scaling + productionizing |
| **Pilot modality** | Trauma X-ray (trauma XR) |
| **Data language** | French |
| **Rules source of truth** | Groupe 3R "Checklist Comptes Rendus Radiologiques" (Mars 2013) |

---

## 1. Problem & rationale

Trauma X-ray report quality directly affects clinical decisions and patient safety. Ambiguous
conclusions and missing/incomplete follow-up are known sources of diagnostic error and
litigation. In parallel, the group deploys **Gleamer BoneView** (fracture / dislocation /
effusion / bone-lesion AI); its value depends on being applied to the **right exams** and on
its **concordance** with the radiologist's report.

**Hypothesis:** LLM systems can reproducibly (a) adjudicate BoneView alerts against the report
and (b) peer-review report quality, at full volume, with agreement close to expert-to-expert,
enabling continuous QA and a context-aware AI deployment in a human-in-the-loop mode.

## 2. Two projects, two goals

### Projects
- **Project 1 - AI vs report concordance (BoneView adjudication).** LLM applies an arbitrated
  rule set to each report: intended-use fit (from Indications) and ground truth (from
  Conclusion), to measure whether/where BoneView alerts correspond to intended-use findings.
- **Project 2 - LLM peer-review (report QC).** LLM scores report-form quality against the
  Groupe 3R checklist (metric set in [`qc-metrics-spec.md`](qc-metrics-spec.md)).

### Goals
- **Goal 1 - Operations:** deploy both tools in routine use (context-aware BoneView alerting; automated report QC in the reporting workflow).
- **Goal 2 - Publication:** two papers - (1) **LLM adjudication of BoneView**; (2) **LLM-based report QC**.

## 3. Scope

### Project 1 - BoneView adjudication *(pilot: 888 doubt alerts, canonical)*
- Intended-use fit (in / out / indeterminate) from the Indication; ground truth (anomaly / doubt / negative) from the Conclusion.
- Metrics: **B1** order/indication conformity, **B2** eligible finding, **ARDR** AI-report discrepancy (region-level).
- **Extension:** scale to **> 10k** reports including AI-**positive** and AI-**negative** cases (device-output-balanced), for per-output **PPV / NPV**.

### Project 2 - Report QC *(pilot: 50 reports, single radiologist)*
- Report-quality dimensions: **Q1** ambiguity, **Q2** follow-up present, **Q3** follow-up completeness (checklist §7: délai / modalité / attendu).
- Checklist QC metric set (Part C): **SRC, SGE, SEF, AMI, CQA, CCS, CPZ** (+ **QCS** composite), tiered *existing pilot pool* vs *next level*.
- All metrics computed in **slices** (global / radiologist / region / modality / day-of-week).

### In scope (v1)
Trauma XR; French; text only (not images); batch processing of reports & orders; a
quality/appropriateness report with flags for radiologist review; an internal radiologist
QCS ranking.

### Out of scope (v1 - candidates for v2+)
Other modalities (CT/MRI/US) beyond slicing hooks; image analysis; real-time RIS/PACS
integration during dictation; auto-rewriting of conclusions; other languages.

## 4. Reference standard & honesty constraints

- **No image-based adjudication is available.** The **radiologist report is the reference**;
  Project 1 results are framed as **concordance / appropriateness**, not standalone diagnostic
  accuracy. The incorporation-bias caveat (AI visible during reporting) is permanent.
- Enriching by device output (>10k) yields **per-output PPV/NPV**, not population sensitivity/specificity.
- The current QC gold is **single-radiologist (50)**; a **second reader** is required for
  inter-annotator agreement (κ) on QC metrics.

## 5. Stakeholders & roles

| Role | Responsibility | Who |
|------|----------------|-----|
| Sponsor | Decisions, budget, priorities | the group |
| Product owner | Requirements, prioritization, acceptance | - |
| Clinical lead (radiologist) | Metric definitions, adjudication | - |
| Annotators (2+ radiologists) | Gold-standard labeling | - |
| ML engineer / DS | Models, pipeline, evaluation | - |
| Data engineer | Export, de-identification, storage | - |
| DPO / compliance | RGPD/GDPR, legal basis | - |

## 6. Data

- **Source:** historical trauma XR reports + imaging orders/prescriptions + BoneView output (RIS/device export).
- **Already available:** 50 QC-labeled reports (single radiologist); 888 BoneView doubt alerts (canonical).
- **Scale-up:** > 10k reports incl. AI-positive/negative (Project 1); thousands unlabeled for weak-labeling (Project 2).
- **Privacy:** patient data → mandatory de-identification before processing; on-prem / open-weights inference; RGPD legal basis confirmed with DPO **before** data work.

## 7. Technical approach (short)

- **Hard dimensions (Q2, Q3, B1, B2, and rule-heavy QC metrics):** fine-tune `CamemBERT-bio` / `DrBERT` + rules, bootstrapped from LLM weak labels.
- **Soft dimensions (Q1, SEF, CQA, CCS, CPZ):** LLM-as-judge, rubric-anchored, on a judge-grade model.
- Rules for QC derive from the Groupe 3R checklist. Details: [`03-methodology.md`](03-methodology.md), [`qc-metrics-spec.md`](qc-metrics-spec.md).

## 8. Success criteria (KPI)

| Project metric | MVP target | How measured |
|----------------|-----------|--------------|
| Label validation (LLM↔radiologist) | κ ≥ 0.61 (min) / 0.81 (goal) | Two-radiologist study; confusion matrix |
| Q1 (ambiguity) | κ model↔expert ≥ 0.6 | Agreement vs consensus |
| Q2 (follow-up present) | F1 ≥ 0.90 | vs gold |
| Q3 (follow-up completeness) | F1 ≥ 0.80 | vs gold (délai/modalité/attendu) |
| B1 (order in-scope) | F1 ≥ 0.90 | vs gold |
| B2 (eligible finding) | F1 ≥ 0.85; region/type accuracy | vs gold |
| ARDR | region-level rate + CIs | report vs BoneView, per region |
| Project 1 (>10k) | per-output PPV / NPV | device-output-balanced cohort |
| QC metrics (SRC/SGE/SEF/CQA/CCS/CPZ) | precision/recall vs gold | validated on the labeled set |
| Usefulness | ≥ 70% of flags judged valid | Radiologist audit of a sample |

> Thresholds are starting targets, finalized after κ is measured (model ceiling is bounded by human agreement).

## 9. Risks & mitigations

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Report reference not independent of BoneView (incorporation bias) | High | Frame as concordance, not accuracy; permanent caveat; consecutive/unfiltered sampling in >10k |
| QC gold is single-rater (no κ yet) | High | Second-reader study before any operational ranking |
| Ranking penalizes case mix / gets gamed | Medium | Internal-only, modality-sliced, per-100-same-modality, shrinkage + min-N; improvement not punishment |
| LLM judge over-rates / unstable | Medium | Rubric-anchored scoring; fixed seed/temp; calibration |
| Metric overlap / double counting in QCS | Medium | Keep QCS components orthogonal; document overlaps |
| Model too small (Mistral 7B) for SEF/CQA/CCS | Medium | Validate against / upgrade to the Opus-class judge |
| BoneView case-level output (ARDR is region-level) | Medium | Region-level claim only; define >0.7 vs POSITIVE/DOUBT bands |
| 512-token limit (CamemBERT-bio/DrBERT) | Low-med | Per-section processing; long-context variant |

## 10. High-level plan & milestones

| Month | Milestone | Deliverable |
|-------|-----------|-------------|
| **M1** | Gold consolidated (checklist rubric frozen), two-radiologist κ, LLM baseline | Frozen gold, κ report, baseline metrics |
| **M2** | Scale + specialized models | >10k extension (P1); encoders + QC metrics; per-output PPV/NPV |
| **M3** | Deploy + publish | Operations pilot, QCS dashboard, review UI; two paper drafts |

Detail: [`02-roadmap.md`](02-roadmap.md).

## 11. Resource budget (rough)

- **People:** clinical lead (part-time), 2 radiologist annotators (part-time), ML engineer (core), data engineer (part-time).
- **Infra:** on-prem GPU for open-weights LLM inference + encoder fine-tuning.
- **Software:** open-source (HuggingFace, PyTorch, vLLM/Ollama, labeling tool). No license cost.

## 12. Assumptions & dependencies

- Access to reports, orders, and BoneView output is legally cleared.
- ≥ 2 radiologists for labeling + an adjudicator.
- On-prem inference infrastructure provisioned.
- RIS/device export formats are known and stable.
