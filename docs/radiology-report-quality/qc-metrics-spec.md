# QC Metrics Specification (Project 2 - LLM Peer-Review)

Operational specification for the report-quality metric set. Every metric is computed
over French trauma XR reports, from defined report sections, and reported in **slices**.

> **Source of truth.** All *rules* derive from the Groupe 3R "Checklist Comptes Rendus
> Radiologiques" (Cellule Quality Improvement, Mars 2013), reproduced in Appendix A. Where
> an earlier metric definition conflicted with the checklist, the checklist wins (see the
> reconciliation notes). Reports are section-structured (Correspondants / Titre / Indications
> / Technique / Contenu-Findings / Conclusions). Current annotated gold = 50 reports, single
> radiologist (a second reader is required for inter-rater agreement).

---

## 1. Metric tiers

| Tier | Metrics | Status |
|------|---------|--------|
| **Existing pilot pool** | SRC, SGE, SEF (laterality), AMI (ambiguity), MFR, ARDR | Already piloted (QR pipeline / annotation / 888 BoneView) |
| **Next level** | CQA, CCS (0-5), CPZ, SEF (full discordance §8), QCS | New, to implement |

---

## 2. Slicing model (applies to every metric)

Aggregated across five slice dimensions:

| Slice | Values |
|-------|--------|
| Global | all reports |
| Per radiologist | signing radiologist |
| Per anatomical region | hand, wrist, foot, ankle, knee, shoulder, elbow, hip/pelvis, spine, rib cage, ... |
| Per modality | XR (pilot); extensible later |
| Per day of week | Mon ... Sun |

**Ranking (decided):**
- **Internal use only** (quality improvement, not punishment; not published externally).
- **Sliced by modality**, and **normalized per 100 reports of the same modality** (a modality-specific rate so readers are compared on like-for-like volume).
- Minimum denominator per slice (default n >= 20; show n always); Wilson 95% CIs on rates.
- Empirical-Bayes **shrinkage** toward the modality mean for low-volume readers.
- Day-of-week and rare regions are **exploratory** slices, not KPIs (multiplicity control).

---

## 3. Master table

| Code | Metric | Tier | Output | Source section(s) | Checklist ref | Method | Dir |
|------|--------|------|--------|-------------------|---------------|--------|-----|
| SRC | Structured Report Compliance | pilot | % | Whole report | §10 + structure | Rule | up |
| SGE | Spelling / Grammar Errors | pilot | count/report | Conclusion (+ Findings) | §10 | LLM | down |
| SEF | Semantic Error Flag Rate | pilot / next | % reports | Titre + Findings + Conclusion (+ priors) | §2,§5,§6 laterality; §8 discordance | LLM | down |
| AMI | Ambiguity Index | pilot | rate | Conclusion | §6 clear conclusion | Hybrid | down |
| MFR | Missing Follow-up Rate | pilot | % reports | Conclusion | §6 proposal + §7 | LLM | down |
| ARDR | AI-Report Discrepancy Rate | pilot | % studies (region-level) | Findings + Conclusion vs BoneView | (device, not checklist) | Hybrid | down |
| CQA | Clinical Question Addressed | next | % reports | Indication vs Conclusion | §3 + §6 "repondu a la question" | LLM | up |
| CCS | Conclusion Completeness Score | next | 0-5 | Conclusion (+ Contenu) | §6 (applicable items) | LLM rubric | up |
| CPZ | Conclusion Prioritization | next | pass/fail | Conclusion vs Indication | §6 "priorise les conclusions" | LLM | up |
| QCS | Quality Composite Score | next | 0-100 | composite | - | Formula | up |

---

## 4. Per-metric specification (grounded in the checklist)

### SRC - Structured Report Compliance  ·  pilot  ·  checklist structure + §10
- Report contains the expected sections/headings (Indication / Technique / Findings / Conclusion; Correspondants and Titre present) and the layout is respected (§10 "la mise en page est-elle bien respectee").
- Rule-based. Output % fully compliant. **Target >= 99%.** SRC failure sets `blocked_by_structure` and suppresses section-dependent metrics.

### SGE - Spelling / Grammar Errors  ·  pilot  ·  §10 "Absence de faute d'orthographe"
- LLM counts typos / speech-recognition errors and grammar (agreement), **ignoring** medical jargon and standard abbreviations. Output count/report; bands None / Minor(1) / Moderate(2) / Major(3+).

### SEF - Semantic Error Flag Rate  ·  pilot (laterality) + next (discordance §8)
- Detects (a) **laterality mismatch** droite/gauche across Titre / Contenu / Conclusion (§2, §5, §6 all ask "cote droit ou gauche ... si applicable") - **piloted**; (b) contradictions **within** the report (§8 "Absence de discordances au sein du compte-rendu") and **vs prior reports** for the same patient (§8) - **next level**; (c) dangling references ("cette lesion" without antecedent).
- Output % reports with >= 1 flag + per-type counts. Report laterality as its own sub-rate.

### AMI - Ambiguity Index  ·  pilot  ·  §6 "conclusion claire"
- Inversely tracks §6 "a-t-on donne une conclusion claire (diagnostic voire diagnostic differentiel)". Flags conclusions with no clear diagnosis, or hedging (doute / suspicion / possible / non exclu) left unresolved.
- **Reconciliation note:** the "forbidden phrase" list ("a correler a la clinique", "sans changement") is a **heuristic**, NOT a checklist rule. It may prioritize candidates, but a phrase alone is not a defect - context decides. Do not fail a report solely for a term the checklist does not forbid.

### MFR - Missing Follow-up Rate  ·  pilot  ·  §6 proposal + §7
- Uncertainty present in the Conclusion **and** the follow-up proposal is missing or incomplete. Per the checklist, a follow-up proposal (§7 "si proposition d'examen supplementaire faite") is complete only with **all three**: **delai** (timeframe), **modalite**, and **ce qu'on attend de cet examen** (expected purpose).
- **Reconciliation note:** the expected-purpose slot ("attendu") is required by §7 and was missing from the earlier definition - it is now part of follow-up completeness (and of Q3 in the rubric).
- Output % of reports; **specify denominator** (all reports vs uncertain-only) - do not conflate.

### ARDR - AI-Report Discrepancy Rate  ·  pilot  ·  device-linked (region-level)
- Studies where BoneView flags a finding (confidence > 0.7) **not** mentioned in Findings/Conclusion, **broken down by body region**.
- **Region-level by design** (confirmed): BoneView returns a case-level label without localisation, so ARDR is a **region-level** discrepancy, not lesion-level. Define the ">0.7" cut against the POSITIVE(>0.9)/DOUBT(0.5-0.9) bands. "Chest" = **rib cage** (BoneView does not read lung). Text match is negation-aware.

### CQA - Clinical Question Addressed  ·  next  ·  §3 + §6 "a-t-on repondu a la question posee"
- LLM compares the Indication's question (§3 "le probleme/la question est-il exprime") to the Conclusion; label answered / partial / not answered. Route uninformative indications to a separate bucket, not to "failed".

### CCS - Conclusion Completeness Score (0-5)  ·  next  ·  §6 applicable items
- Score = proportion of **applicable** §6 conclusion items satisfied, mapped to 0-5. §6 items (each gated by "si applicable"): lesion **size**; **localisation**; **side** droite/gauche; **quantification** (arthrosis degree / effusion importance); **comparison** with prior; **stability** vs prior; **date** of prior; **question answered**; **clear conclusion** (diagnosis / differential); **prioritized** conclusions; **adequate proposal** to clinician.
- `CCS = round(5 * satisfied_applicable / total_applicable)`; record which items missed.
- **Reconciliation note:** the earlier "one line per diagnosis / no verbosity" criterion is **not in the checklist** and is dropped from the canonical score (may be tracked separately as non-canonical, if wanted).

### CPZ - Conclusion Prioritization  ·  next  ·  §6 "a-t-on priorise les conclusions"
- Checks the checklist rule literally: first answer the question, then diagnoses in decreasing importance. Output pass/fail.

### QCS - Quality Composite Score (0-100)  ·  next
- Weights as provided: **MFR 30, ARDR 25, CCS 20, SEF 15, SRC 10**.
- Sub-scores (polarity fixed): lower-is-better rates -> `s = 100*(1-rate)` (MFR, ARDR, SEF); `SRC` as %; `CCS -> 100*CCS/5`. `QCS = sum(w_i*s_i)`.
- **Level & normalization:** computed per radiologist **within a modality**, normalized **per 100 reports of the same modality**, **internal use only**, with min-N + shrinkage.
- **Coverage gap:** AMI, SGE, CQA, CPZ are diagnostic-only (not in the weighted composite) unless you decide to fold them in.

---

## 5. Critical review - remaining risks

1. **Overlap / double counting.** MFR and AMI both touch uncertainty; CCS embeds CQA (question answered) and CPZ (prioritization); SEF overlaps standalone laterality. If several enter QCS, the same defect counts twice. Keep QCS components orthogonal or state the overlap.
2. **Ranking validity.** Even internal and per-100-same-modality, per-radiologist comparison still carries **anatomy / indication case-mix**; a reader with harder referrals scores worse. Modality slicing helps but does not fully adjust - keep shrinkage + min-N, and read rankings as signals, not verdicts.
3. **Gaming.** Ranking on MFR/AMI can incentivize dropping "doute" from conclusions. Internal-only + improvement framing mitigates; monitor hedging rates over time.
4. **ARDR feasibility.** Region-level and threshold-sensitive (see spec); do not claim lesion-level accuracy.
5. **Model tier.** SEF (discordance), CQA (intent), CCS/CPZ (clinical-weight ordering) are hard - align on the Opus-class judge or validate Mistral 7B against it first.
6. **Validation debt.** Every LLM-judged metric needs kappa vs radiologists; current gold is single-rater (50). Prioritize SEF, CQA, CCS for two-reader validation.

## 6. Decisions

**Resolved:** ranking is internal-only, modality-sliced, normalized per 100 reports of the same modality; ARDR is region-level; follow-up completeness includes the expected-purpose slot (§7); non-checklist criteria (forbidden-phrase blacklist as a hard rule; "no verbosity") are demoted to heuristics/dropped.

**Open:** QCS coverage (fold in AMI/SGE/CQA/CPZ or keep diagnostic-only); ARDR exact confidence-band mapping; MFR denominator (all vs uncertain-only); min-N and shrinkage strength.

## 7. Additional checklist items not yet metricized (candidate future metrics)

From the checklist, available but not yet turned into metrics: §1 correspondants correct; §2 titre matches the exams performed; §4 technique (contrast / side-effects mentioned when applicable); §5 measurements, grading definition in parentheses, "reste descriptif sans conclure prematurement"; §9 images correspond to the patient and adequate format.

---

## Appendix A - Groupe 3R checklist (source of truth, Mars 2013)

1. **Correspondants** - referring physicians' names/addresses (prescriber + cc) correct.
2. **Titre** - matches the exams performed; side (droit/gauche) specified if applicable.
3. **Indications** - problem/question expressed; order text repeated; priors considered if available.
4. **Technique** - contrast product mentioned if administered; side-effects mentioned if present.
5. **Contenu (Findings)** - adequate lesion measurements; localisation; side; grading/stage/qualifier (discret/modere/important) with definition in parentheses; stays descriptive without concluding prematurely - all "si applicable".
6. **Conclusions** - lesion size; localisation; side; quantified anomaly (arthrosis degree / effusion importance); comparison with prior; stable or not vs prior; date of prior; **question answered**; **clear conclusion** (diagnosis / differential) to orient management; **conclusions prioritized** (answer first, then decreasing importance); **adequate proposal** to clinician - all "si applicable / si necessaire".
7. **Si proposition d'examen supplementaire faite** - specify the **delai**; the **modalite**; **what is expected** of the exam.
8. **Discordances** - none within the report; none vs prior reports for the same patient.
9. **Images** - final images match the patient; adequate format (contrast, size).
10. **Mise en page** - no spelling errors; layout respected.
