# QC Metrics Specification (Project 2 - LLM Peer-Review)

Operational specification for the report-quality metric set. Every metric is computed
over French trauma XR reports, from defined report sections, and reported in **slices**.

> Conventions: data language French, spec language English. Reports are section-structured
> (Indication / Technique / Findings / Conclusion). Current annotated gold = 50 reports,
> single radiologist (a second reader is required for inter-rater agreement).

---

## 1. Slicing model (applies to every metric)

Each metric is aggregated across five slice dimensions:

| Slice | Values |
|-------|--------|
| Global | all reports |
| Per radiologist | signing radiologist |
| Per anatomical region | hand, wrist, foot, ankle, knee, shoulder, elbow, hip/pelvis, spine, rib cage, ... |
| Per modality | XR (pilot); extensible later |
| Per day of week | Mon ... Sun |

**Reporting rules for slices (required to avoid false signal):**
- **Minimum denominator:** do not report or rank a slice with n < 20 reports (configurable). Show n on every slice.
- **Uncertainty:** rates carry **Wilson 95% CIs**; scores carry mean +/- SD or CI.
- **Ranking stability:** per-radiologist ranking uses **shrinkage** (empirical-Bayes toward the global mean) so low-volume readers are not over-penalized by noise.
- **Case-mix:** per-radiologist comparisons are **adjusted for anatomy / modality / indication mix** (a reader with harder or more ambiguous referrals should not rank worse for that reason). See risks (Section 5).
- **Multiplicity:** 5 slice dimensions x many metrics x time is many comparisons. Pre-specify which slices are **official KPIs** vs **exploratory**; day-of-week is exploratory by default.

---

## 2. Master table

| Code | Metric | Output | Source section(s) | Method | Direction | Target |
|------|--------|--------|-------------------|--------|-----------|--------|
| SRC | Structured Report Compliance | % | Whole report | Rule | higher better | >= 99% |
| SGE | Spelling / Grammar Errors Index | count / report | Conclusion (+ Findings) | LLM | lower better | trend |
| SEF | Semantic Error Flag Rate | % of reports | Title + Findings + Conclusion | LLM | lower better | trend |
| CQA | Clinical Question Addressed Rate | % of reports | Indication vs Conclusion | LLM | higher better | trend |
| CCS | Conclusion Completeness Score | 0-5 | Conclusion (+ context) | LLM rubric | higher better | trend |
| CPZ | Conclusion Prioritization | pass / fail | Conclusion vs Indication | LLM | higher better | trend |
| AMI | Ambiguity Index (Forbidden Terms) | rate / density | Conclusion | Hybrid | lower better | trend |
| MFR | Missing Follow-up Rate | % of reports | Conclusion | LLM (Q1 and not Q2) | lower better | trend |
| ARDR | AI-Report Discrepancy Rate | % of studies | Findings + Conclusion vs BoneView | Hybrid | lower better | trend |
| QCS | Quality Composite Score | 0-100 | composite | Formula | higher better | ranking |

---

## 3. Per-metric specification

### SRC - Structured Report Compliance
- **Question:** does the report conform to the template structure and standard headings?
- **Sections required:** Indication / Technique / Report template / Findings / Conclusion, with correct headings.
- **Method:** rule-based header/section detection (a superset of the earlier 4-section gate).
- **Output:** % of reports fully compliant. **Target >= 99%.**
- **Slices:** all. Gate: SRC failure can suppress downstream semantic metrics (a missing Conclusion invalidates Conclusion-based metrics) - record `blocked_by_structure`.

### SGE - Spelling / Grammar Errors Index
- **Question:** how many uncorrected typos / speech-recognition errors remain?
- **Method:** LLM counts typographic and grammatical errors (gender/number agreement), **ignoring** medical jargon and standard radiology abbreviations to avoid false positives.
- **Output:** error count per report; roll up to mean per slice. Scale bands: None / Minor (1) / Moderate (2) / Major (3+).
- **Risk:** jargon-vs-typo separation is error-prone at small model sizes (Section 5).

### SEF - Semantic Error Flag Rate
- **Question:** are there internal contradictions or wording defects?
- **Detects:** (a) contradictions between **Title <-> Findings <-> Conclusion**; (b) dangling references ("cette lesion" with no antecedent); (c) **laterality mismatch** (droite/gauche) across sections.
- **Method:** LLM extracts entities + laterality per section, then compares; reports a flag + type.
- **Output:** % of reports with >= 1 semantic flag; also per-type counts (contradiction / reference / laterality).
- **Note:** laterality mismatch is the highest patient-safety item; report it as its own sub-rate.

### CQA - Clinical Question Addressed Rate
- **Question:** did the Conclusion directly answer the referring clinician's question in the Indication?
- **Method:** LLM compares Indication intent to Conclusion content; label answered / partially / not answered.
- **Output:** % answered (and % partial). Only meaningful when the Indication is informative - route uninformative indications to a separate bucket rather than counting them as failures.

### CCS - Conclusion Completeness Score (0-5)
- **Question:** how complete is the Conclusion on form (Checklist qualite sur la forme)?
- **Rubric (1 point each):** (1) clinical question answered; (2) prior comparison stated; (3) recommendations present where warranted; (4) findings prioritized; (5) one line per diagnosis (no verbosity).
- **Method:** LLM rubric scoring with per-item evidence.
- **Output:** 0-5 (report mean per slice). **Overlaps** with CQA (item 1) and CPZ (item 4) - see Section 5.

### CPZ - Conclusion Prioritization
- **Question:** are diagnoses listed in decreasing clinical importance, starting with the answer to the clinical question?
- **Method:** LLM orders the Conclusion's diagnoses by clinical weight and checks the reported order.
- **Output:** pass / fail (or rank-correlation). Overlaps CCS item 4.

### AMI - Ambiguity Index (Forbidden Terms)
- **Question:** how often does the report use empty / responsibility-shifting phrasing?
- **Detects:** "sans changement / no change" without further description; "a correler / correlate with clinical context" used to shift responsibility; long descriptions with no clear diagnosis; **doute / suspicion without a follow-up suggestion**.
- **Method:** hybrid - a French forbidden-phrase lexicon flags candidates, an LLM confirms whether the use is empty **in context** (not a blind blacklist; see Section 5).
- **Output:** flagged-report rate and/or forbidden-term density per 100 words.
- **Overlaps** MFR on the "doute/suspicion without follow-up" clause.

### MFR - Missing Follow-up Rate
- **Question:** where uncertainty is expressed, is a follow-up plan missing?
- **Definition:** the Conclusion expresses diagnostic uncertainty ("doute", "suspicion", "possible", "a confronter", "non exclu") **AND** there is **no** explicit recommendation for additional imaging (MRI, CT, control, specialist opinion) **AND no follow-up time**.
- **Method:** LLM = detect uncertainty (Q1) then check for a concrete follow-up action + timeframe (Q2/Q3). MFR = share where uncertainty is present but follow-up is absent.
- **Output:** % of reports (denominator = all reports, or = uncertain reports; **specify which** - both are useful and must not be conflated).

### ARDR - AI-Report Discrepancy Rate
- **Question:** does BoneView flag a finding the report does not mention?
- **Definition:** studies where BoneView flags a finding (**confidence > 0.7**) that is **not** mentioned in Findings/Conclusion. Broken down by body region (hand, foot, rib cage, ...).
- **Inputs:** BoneView output **and** report text (this is the one metric needing the device output).
- **Method:** map BoneView flag -> body region; check the report (negation-aware) for a matching finding in that region; discrepancy if absent.
- **Constraints (important):** BoneView returns a **case-level** label without localisation, and its bands are POSITIVE (>0.9) / DOUBT (0.5-0.9) / NEGATIVE; a **">0.7"** threshold sits **inside the DOUBT band**, so define it exactly. ARDR is therefore **region-level, not lesion-level**. "Chest" here means **rib cage** (BoneView does not read lung/soft tissue).
- **Output:** % of studies with a discrepancy, per region.

### QCS - Quality Composite Score (0-100)
- **Definition:** weighted composite for radiologist ranking. Weights as provided: **MFR 30%, ARDR 25%, CCS 20%, SEF 15%, SRC 10%**.
- **Normalization (must be pinned):** each component maps to a 0-100 sub-score with explicit polarity, then QCS = sum(w_i * s_i):
  - Rate metrics where lower is better (MFR, ARDR, SEF): `s = 100 * (1 - rate)`.
  - SRC (already %): `s = SRC`.
  - CCS (0-5): `s = 100 * CCS / 5`.
- **Level:** computed per radiologist (or per slice) from aggregated components; therefore subject to **min-N, shrinkage, and case-mix adjustment** (Section 1, Section 5).
- **Coverage gap:** AMI, SGE, CQA, CPZ are **not** in the weighted composite - decide whether they are diagnostic-only or folded in (Section 6).

---

## 4. Data & label requirements

| Need | Detail |
|------|--------|
| Report sections | Reliable segmentation (Indication/Technique/Findings/Conclusion) - SRC also depends on this |
| BoneView output | Case-level label + confidence + body region, joined to the study (ARDR only) |
| Gold labels | Per-metric human labels for validation; current gold = 50 reports, single radiologist |
| Lexicons | French cue lists (uncertainty, follow-up/action, forbidden phrases, laterality, negation) |
| Metadata | Radiologist ID, region, modality, date (for slicing); denominators per slice |

---

## 5. Critical review - risks & validity traps

1. **Overlap / double counting.** MFR and AMI both penalize "uncertainty without follow-up"; CCS embeds CQA (item 1) and CPZ (item 4); SEF overlaps standalone laterality. If several of these enter QCS or a dashboard uncritically, the same defect is counted multiple times. **Define each metric orthogonally, or state the overlap deliberately.**
2. **Radiologist ranking is a validity and governance risk.** Raw QCS ranking penalizes **case mix**: a reader with harder anatomy, more ambiguous referrals, or more BoneView-flagged studies scores worse for reasons outside their control. Ranking needs **case-mix adjustment** (as Project 1 anatomy-adjusts) plus **shrinkage** and **min-N**. Without these, the ranking is misleading.
3. **Perverse incentives (gaming).** Publicly ranking on MFR/AMI rewards **dropping uncertainty language** ("doute") to avoid the flag - which harms care. Use QCS for improvement, not punishment; keep it blinded/aggregate; monitor for suppression of hedging over time.
4. **Forbidden-term blacklists over-flag.** "A correler a la clinique" and "sans changement" are often clinically legitimate. AMI must judge **in context** (hybrid lexicon + LLM), not by term presence alone, or false positives will dominate.
5. **ARDR is region-level and threshold-sensitive.** Case-level BoneView output without localisation, plus a 0.7 cut inside the DOUBT band, means ARDR measures **region-level** discrepancy at best. Fix the threshold definition, keep the region taxonomy inside BoneView scope (rib cage, not lung), and handle negation in the text match.
6. **Model tier.** SEF (cross-section contradiction), CQA (intent matching), CCS/CPZ (clinical-weight ordering) are hard reasoning tasks. Mistral 7B is likely insufficient; align on the Opus-class judge used in Project 1, or validate 7B against it before trusting these metrics.
7. **Multiplicity & small slices.** 5 slice dimensions over time generate many comparisons; day-of-week and rare regions are noisy. Pre-register official KPIs vs exploratory slices; apply min-N and CIs everywhere.
8. **Validation debt.** Every LLM-judged metric needs kappa vs radiologists (current gold is single-rater). Prioritize SEF, CQA, CCS for a two-reader validation before any operational ranking.

---

## 6. Open decisions (for sign-off)

1. **QCS coverage & weights:** keep MFR30/ARDR25/CCS20/SEF15/SRC10, and are AMI, SGE, CQA, CPZ diagnostic-only or folded in?
2. **Ranking:** case-mix adjustment yes/no; min-N per slice; shrinkage strength.
3. **ARDR:** exact confidence-threshold definition (map to POSITIVE/DOUBT bands); region taxonomy; lesion- vs region-level claim.
4. **MFR denominator:** all reports vs uncertain-only reports.
5. **Official vs exploratory slices:** which of global / radiologist / region / modality / day-of-week are KPIs.
6. **Governance:** is QCS for improvement only, and blinded, to avoid gaming?

---

## 7. Mapping to the earlier rubric codes

| New code | Earlier code(s) | Relationship |
|----------|-----------------|--------------|
| MFR | Q1 (ambiguity) AND NOT Q2 (follow-up) | composed |
| AMI | Q1 + forbidden-phrase layer | superset of ambiguity |
| CQA, CCS, CPZ | new (form/content quality) | extend Q-set |
| SEF | laterality + conclusion<->description discordance | consolidates |
| SRC | structural gate (expanded to 5 sections) | superset |
| ARDR | B2 / concordance, from the report-miss angle | device-linked |
| B1 (order in-scope) | unchanged | BoneView appropriateness (Project 1) |
