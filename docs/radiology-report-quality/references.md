# Литература и источники · RRQ-FR

План основан на следующих работах (подобраны через PubMed и веб-поиск, 2026-07).

## Французские биомедицинские модели (open-source)

- **CamemBERT-bio** — Touchent & de la Clergerie, LREC-COLING 2024.
  [ACL Anthology](https://aclanthology.org/2024.lrec-main.241/) ·
  [HuggingFace `almanach/camembert-bio-base`](https://huggingface.co/almanach/camembert-bio-base) ·
  [arXiv](https://arxiv.org/html/2306.15550v3)
- **DrBERT** — Labrak et al., ACL 2023. French RoBERTa на корпусе NACHOS.
  [GitHub](https://github.com/qanastek/DrBERT)
- **DrBenchmark** — бенчмарк French biomedical NLU. [arXiv](https://arxiv.org/pdf/2402.13432)
- **ModernCamemBERT-bio** — long-context биомедицинский/клинический энкодер.
  [OpenReview PDF](https://openreview.net/pdf/98fd42b52170e958e03c55d15c9efba97720db35.pdf)
- **CamemBERT 2.0** — обновлённая French LM. [arXiv](https://arxiv.org/pdf/2411.08868)

## Follow-up / actionable findings в радиологических репортах (NLP)

- Use of Machine Learning to Identify Follow-Up Recommendations in Radiology Reports.
  [PMC7534384](https://pmc.ncbi.nlm.nih.gov/articles/PMC7534384/)
- Automatic detection of actionable radiology reports using BERT.
  [PMC8436473](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8436473/)
- NLP Model for Identifying Critical Findings — Multi-Institutional Study.
  [PMC9984612](https://pmc.ncbi.nlm.nih.gov/articles/PMC9984612/)
- Identifying Imaging Follow-Up in Radiology Reports: Traditional ML vs LLM.
  [arXiv 2511.11867](https://arxiv.org/html/2511.11867)
- Extraction of Recommendation Features in Radiology with NLP (AJR).
  [AJR](https://www.ajronline.org/doi/10.2214/AJR.07.3508)

## LLM-as-judge / оценка качества радиологических репортов

- **VERT: Reliable LLM Judges for Radiology Report Evaluation.**
  [arXiv 2604.03376](https://arxiv.org/html/2604.03376)
- Beyond Scalar Scores: LLM-based Metrics for Clinical Significance Evaluation.
  [arXiv 2606.18797](https://arxiv.org/pdf/2606.18797)
- LLMs in Radiology Reporting — Systematic Review (Jan 2015–Feb 2025).
  [ScienceDirect](https://www.sciencedirect.com/science/article/pii/S2666521225000912) ·
  [medRxiv](https://www.medrxiv.org/content/10.1101/2025.03.18.25324193v1.full)
- Известные LLM-judge метрики радиологии: **RadFact, GREEN, FineRadScore** (бенчмарки RadEval, RaTE-Eval).

## Ключевые методические выводы, использованные в плане

1. **Follow-up детекция** — «imaging term» рядом с «action term»; неполноту фиксируют
   по отсутствию сопутствующих элементов (срок, модальность, анатомия). → метрики M2, M3.
2. **LLM-судья надёжнее** как строгое информационное сопоставление (binary matching
   против ground-truth), а не открытое рассуждение «оцени 1–5». → метрика M4.
3. **Готовые французские энкодеры** (CamemBERT-bio, DrBERT) снимают необходимость
   обучения с нуля; лимит 512 токенов решается посекционно / long-context вариантами.
4. LLM-метрики ближе к экспертной оценке и дают интерпретируемую обратную связь,
   чем n-gram метрики. → выбор LLM-as-judge для «мягких» метрик M1, M4.
