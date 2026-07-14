# Working forms for colleagues · RRQ-FR

Ready-to-use templates for day-to-day work. CSV versions are in [`forms/`](forms/) -
import into Excel / Google Sheets / a labeling tool.

Dimensions: **Q1** ambiguity · **Q2** follow-up present · **Q3** completeness ·
**B1** order in-scope · **B2** eligible finding · **Part C** checklist QC (SRC, SGE, SEF,
AMI, CQA, CCS, CPZ, ARDR). Part C rules follow the Groupe 3R checklist; see
[`qc-metrics-spec.md`](qc-metrics-spec.md).

---

## Form 1 - Report annotation (for radiologists)

Each annotator fills this **independently**. One report = one row.

| Field | Type / values | Notes |
|-------|---------------|-------|
| `report_id` | text | Anonymous report ID |
| `annotator_id` | text | Who labels (A / B) |
| `Q1_ambiguity` | unambiguous / ambiguous (or 0/1/2) | Conclusion clarity |
| `Q2_followup` | yes / no | Follow-up recommendation present |
| `Q3_completeness` | complete / incomplete / NA | NA if Q2=no |
| `Q3_missing_slots` | delai,modalite,attendu | Checklist §7 slots; empty if complete |
| `B1_order_in_scope` | in / out | Order vs BoneView intended use |
| `B1_reason` | R_modality / R_region / R_age / - | Reason if out-of-scope |
| `B2_eligible_finding` | present / absent | Eligible finding in conclusion |
| `B2_finding_types` | fracture,dislocation,effusion,lesion | One or more; empty if absent |
| `B2_body_region` | text | e.g. wrist, ankle, pelvis |
| `B2_out_of_scope_finding` | yes / no | Eligible-type finding but out-of-scope region |
| `src_compliant` | yes / no | Part C - structure/layout ok (§10) [pilot] |
| `sge_band` | none/minor/moderate/major | Spelling/grammar (§10) [pilot] |
| `sef_laterality` | none / mismatch | Droite/gauche consistent (§2,§5,§6) [pilot] |
| `sef_discordance` | none/within/vs_prior | Discordance (§8) [next] |
| `ami_clear_conclusion` | yes / no | Clear conclusion present (§6) [pilot] |
| `cqa` | answered/partial/not/NA | Question answered (§3,§6) [next] |
| `ccs` | 0-5 | Conclusion completeness (§6 applicable) [next] |
| `ccs_missing` | text | Which §6 items missed |
| `cpz` | pass/fail/NA | Conclusions prioritized (§6) [next] |
| `ardr_region_mentioned` | yes/no/NA | Report mentions finding in BoneView-flagged region |
| `comment` | text | Any notes |
| `litigious` | yes / no | Borderline → adjudication |

**Filled example (Part A/B/C shown compactly):**

| report_id | Q1 | Q2 | Q3 | B1 | B2 | src | sge | sef_lat | ami_clear | cqa | ccs | cpz |
|-----------|----|----|----|----|----|-----|-----|---------|-----------|-----|-----|-----|
| TXR-00412 | ambiguous | yes | incomplete | in | present | yes | minor | none | no | partial | 3 | fail |
| TXR-00517 | unambiguous | no | NA | out | absent | yes | none | none | yes | answered | 5 | pass |

---

## Form 2 - Adjudication / consensus (for the clinical lead)

For reports where A and B disagree.

| Field | Type / values |
|-------|---------------|
| `report_id` | text |
| `Q1_A` / `Q1_B` / `Q1_final` | ambiguity value |
| `Q2_A` / `Q2_B` / `Q2_final` | yes/no |
| `Q3_A` / `Q3_B` / `Q3_final` | complete/incomplete/NA |
| `B1_A` / `B1_B` / `B1_final` | in/out |
| `B2_A` / `B2_B` / `B2_final` | present/absent |
| `disagreement_reason` | text |
| `guideline_action` | should the rubric be refined? (yes/no + what) |

> `guideline_action` is the source of rubric improvements between iterations.

---

## Form 3 - Model error log (for ML + lead, evaluation phase)

For FP/FN analysis during system evaluation.

| Field | Type / values |
|-------|---------------|
| `report_id` | text |
| `dimension` | Q1/Q2/Q3/B1/B2 |
| `predicted` | model value |
| `gold` | reference |
| `error_type` | FP / FN / off-by-one (Q1) |
| `likely_cause` | segmentation / hedging / negation / region-mapping / rare phrasing |
| `example_fragment` | quote (de-identified) |
| `fix_idea` | rule / prompt / data |

---

## Form 4 - Weekly status (for standups)

| Field | Value |
|-------|-------|
| `week` | Week N |
| `phase` | M1 / M2 / M3 |
| `checkpoint` | CP1…CP5 (see runbook) |
| `done` | what shipped |
| `blocker` | blockers (incl. legal/data) |
| `current_metrics` | IAA / F1 / κ so far |
| `next_steps` | plan for the week |
| `risks` | new/changed risks |

---

## How to use

1. Copy the CSV from [`forms/`](forms/) into a working sheet.
2. Annotators A and B fill **Form 1** independently (no peeking at each other).
3. ML engineer computes IAA (κ) per field.
4. Disagreements → **Form 2**, lead adjudicates → consensus labels.
5. During model evaluation → **Form 3**.
6. At standups → **Form 4**.
