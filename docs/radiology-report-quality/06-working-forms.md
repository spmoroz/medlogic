# Working forms for colleagues · RRQ-FR

Ready-to-use templates for day-to-day work. CSV versions are in [`forms/`](forms/) -
import into Excel / Google Sheets / a labeling tool.

Dimensions: **Q1** ambiguity · **Q2** follow-up present · **Q3** completeness ·
**B1** order in-scope · **B2** eligible finding.

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
| `Q3_missing_slots` | modality,timeframe,region,condition | Comma-sep; empty if complete |
| `B1_order_in_scope` | in / out | Order vs BoneView intended use |
| `B1_reason` | R_modality / R_region / R_age / - | Reason if out-of-scope |
| `B2_eligible_finding` | present / absent | Eligible finding in conclusion |
| `B2_finding_types` | fracture,dislocation,effusion,lesion | One or more; empty if absent |
| `B2_body_region` | text | e.g. wrist, ankle, pelvis |
| `B2_out_of_scope_finding` | yes / no | Eligible-type finding but out-of-scope region |
| `comment` | text | Any notes |
| `litigious` | yes / no | Borderline → adjudication |

**Filled example:**

| report_id | annotator | Q1 | Q2 | Q3 | Q3_missing | B1 | B1_reason | B2 | B2_types | B2_region | litigious |
|-----------|-----------|----|----|----|-----------|----|-----------|----|----------|-----------|-----------|
| TXR-00412 | A | ambiguous | yes | incomplete | timeframe,modality | in | - | present | fracture | wrist | no |
| TXR-00517 | A | unambiguous | no | NA | - | out | R_region | absent | - | - | yes |

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
