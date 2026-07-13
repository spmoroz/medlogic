# References & sources · RRQ-FR

The plan is grounded in the following work (PubMed + web, 2026-07).

## Gleamer BoneView — intended use (yardstick for Workstream B)

- **BoneView (Gleamer)** — X-rays of adults and children **≥ 2 years**; **limbs, pelvis,
  thoracic & lumbar spine, rib cage**; detects **fractures, effusions, dislocations, bone
  lesions**; labels POSITIVE (>90%), DOUBT (50–90%), NEGATIVE.
  [Health AI Register](https://healthairegister.com/radiology/products/gleamer-ai-boneview-trauma) ·
  [Gleamer BoneView](https://www.gleamer.ai/copilot/boneview)
- FDA 510(k) clearance (fracture detection, appendicular skeleton + rib cage + thoraco-lumbar spine).
  [K222176 summary (FDA)](https://www.accessdata.fda.gov/cdrh_docs/pdf22/K222176.pdf) ·
  [Diagnostic Imaging](https://www.diagnosticimaging.com/view/gleamer-boneview-fda-clearance-for-ai-pediatric-fracture-detection)

## French biomedical models (open-source)

- **CamemBERT-bio** — Touchent & de la Clergerie, LREC-COLING 2024.
  [ACL Anthology](https://aclanthology.org/2024.lrec-main.241/) ·
  [HuggingFace `almanach/camembert-bio-base`](https://huggingface.co/almanach/camembert-bio-base)
- **DrBERT** — Labrak et al., ACL 2023. [GitHub](https://github.com/qanastek/DrBERT)
- **DrBenchmark** — French biomedical NLU benchmark. [arXiv](https://arxiv.org/pdf/2402.13432)
- **ModernCamemBERT-bio** — long-context biomedical/clinical encoder.
  [OpenReview](https://openreview.net/pdf/98fd42b52170e958e03c55d15c9efba97720db35.pdf)
- **CamemBERT 2.0** — updated French LM. [arXiv](https://arxiv.org/pdf/2411.08868)

## Follow-up / actionable findings in radiology reports (NLP)

- Machine Learning to Identify Follow-Up Recommendations in Radiology Reports.
  [PMC7534384](https://pmc.ncbi.nlm.nih.gov/articles/PMC7534384/)
- Automatic detection of actionable radiology reports using BERT.
  [PMC8436473](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8436473/)
- NLP Model for Identifying Critical Findings — Multi-Institutional Study.
  [PMC9984612](https://pmc.ncbi.nlm.nih.gov/articles/PMC9984612/)
- Identifying Imaging Follow-Up: Traditional ML vs LLM. [arXiv 2511.11867](https://arxiv.org/html/2511.11867)

## LLM-as-judge / radiology report quality evaluation

- **VERT: Reliable LLM Judges for Radiology Report Evaluation.** [arXiv 2604.03376](https://arxiv.org/html/2604.03376)
- Beyond Scalar Scores: LLM Metrics for Clinical Significance Evaluation. [arXiv 2606.18797](https://arxiv.org/pdf/2606.18797)
- LLMs in Radiology Reporting — Systematic Review. [ScienceDirect](https://www.sciencedirect.com/science/article/pii/S2666521225000912)
- Known LLM-judge radiology metrics: **RadFact, GREEN, FineRadScore** (RadEval, RaTE-Eval).

## Key methodological takeaways used in the plan

1. **Follow-up detection** — *action term* near an *imaging term*; incompleteness is
   flagged by the absence of accompanying elements (timeframe, modality, region). → Q2, Q3.
2. **LLM judging is more reliable** as strict information matching (binary verification vs
   ground truth) than open-ended "rate 1–5". → Q1 rubric-anchored scoring.
3. **Ready French encoders** (CamemBERT-bio, DrBERT) remove the need to train from scratch;
   the 512-token limit is handled per-section or with long-context variants.
4. **BoneView intended use** is a published, explicit specification (regions/findings/age),
   so B1/B2 can be coded as auditable rules rather than subjective judgments.
