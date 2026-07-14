# Radiology Report Intelligence (French trauma XR) - RRQ-FR

**Pilot modality:** Trauma X-ray (trauma XR)
**Data language:** French · **Owner:** the group
**Rules source of truth:** Groupe 3R "Checklist Comptes Rendus Radiologiques" (Mars 2013)
**Package version:** 2026-07-14

---

## What this is

Documentation for an LLM-based system that (1) adjudicates a commercial fracture AI
(Gleamer BoneView) against the radiologist report and (2) peer-reviews report quality, on
French trauma X-ray reports.

### Two projects
- **Project 1 - AI vs report concordance (BoneView adjudication).** LLM applies an arbitrated rule set (intended-use fit from Indications; ground truth from Conclusion). Canonical pilot: **888 doubt alerts**, scaling to **> 10k** incl. AI-positive/negative. Metrics: **B1, B2, ARDR**.
- **Project 2 - LLM peer-review (report QC).** LLM scores report-form quality against the Groupe 3R checklist. Dimensions **Q1-Q3** plus the checklist metric set **SRC, SGE, SEF, AMI, CQA, CCS, CPZ** (+ **QCS**). Pilot: **50 reports, single radiologist**.

### Two goals
1. **Operations** - deploy both tools in routine use.
2. **Publication** - two papers: LLM adjudication of BoneView; LLM-based report QC.

> **BoneView intended use** (yardstick): conventional radiographs of **limbs, pelvis,
> thoracic & lumbar spine, rib cage**, patients **≥ 2 years**, detecting **fractures,
> dislocations, joint effusions, bone lesions**. No image-based reference is available, so
> Project 1 results are **report-concordance**, not standalone accuracy.

## Quality dimensions & metrics

| Code | Dimension / metric | Project | Tier |
|------|--------------------|---------|------|
| Q1 | Ambiguity of the conclusion | 2 | pilot |
| Q2 | Follow-up present | 2 | pilot |
| Q3 | Follow-up completeness (délai/modalité/attendu, §7) | 2 | pilot |
| SRC | Structured report compliance | 2 | pilot |
| SGE | Spelling / grammar | 2 | pilot |
| SEF | Semantic error (laterality; discordance §8) | 2 | pilot / next |
| AMI | Ambiguity index | 2 | pilot |
| MFR | Missing follow-up rate | 2 | pilot |
| CQA | Clinical question addressed | 2 | next |
| CCS | Conclusion completeness (0-5, §6) | 2 | next |
| CPZ | Conclusion prioritization | 2 | next |
| QCS | Quality composite score (internal ranking) | 2 | next |
| B1 | Order conformity to BoneView intended use | 1 | pilot |
| B2 | BoneView-eligible finding in conclusion | 1 | pilot |
| ARDR | AI-report discrepancy (region-level) | 1 | pilot |

## Package contents

| File | Purpose |
|------|---------|
| [`01-project-passport.md`](01-project-passport.md) | Project passport: two projects/goals, scope, KPIs, risks, milestones |
| [`02-roadmap.md`](02-roadmap.md) | Roadmap: 3-month plan, phases, deliverables |
| [`03-methodology.md`](03-methodology.md) | Methodology: two tracks, QC metrics, slicing/ranking, evaluation |
| [`04-runbook.md`](04-runbook.md) | Step-by-step operational runbook |
| [`05-annotation-guideline.md`](05-annotation-guideline.md) | Annotation rubric, Parts A/B/C (checklist-grounded) |
| [`06-working-forms.md`](06-working-forms.md) | Working forms + CSV templates (incl. Part C fields) |
| [`qc-metrics-spec.md`](qc-metrics-spec.md) | QC metrics spec: definitions, checklist mapping, slicing, ranking, risks |
| [`program-deck.html`](program-deck.html) | Joint 13-slide program deck (both projects + method + adjustments) |
| [`llm-projects-critical-review.html`](llm-projects-critical-review.html) | 8-slide critical review of the two projects |
| [`slides.html`](slides.html) | Earlier 5-slide summary (superseded by the program deck) |
| [`references.md`](references.md) | Literature, BoneView sources, and the Groupe 3R checklist |

## Key principle

> **Definitions and the gold standard come before the model**, and QC rules trace to the
> **Groupe 3R checklist** (source of truth). Pilots already produced labeled data (50 QC
> reports, single radiologist; 888 BoneView doubt alerts). A second radiologist reader is
> required to establish inter-annotator agreement before any operational QC ranking.

## Stack in one line

- **Soft (Q1, SEF, CQA, CCS, CPZ):** LLM-as-judge, rubric-anchored, judge-grade model.
- **Hard (Q2, Q3, B1, B2, SRC, SGE, MFR, ARDR):** fine-tuned `CamemBERT-bio` / `DrBERT` + rules.
- **Privacy:** on-prem / open-weights (patient data, RGPD/GDPR).
