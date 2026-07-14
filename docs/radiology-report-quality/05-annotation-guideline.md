# Annotation Guideline (Rubric) · RRQ-FR

**Pilot scope: Trauma X-ray (trauma XR) reports · Source language: French**

This is the labeling manual for annotators. Goal: label reports **reproducibly** -
two annotators reading the same report should assign the same labels.

The rubric covers three parts:

- **Part A - Report quality** (piloted: 50 trauma XR reports (single radiologist), dimensions *ambiguity* + *follow-up*)
- **Part B - BoneView appropriateness** (piloted: 888 reports - order conformity + eligible findings)
- **Part C - Report-form QC metrics** grounded in the Groupe 3R checklist (SRC, SGE, SEF, AMI, CQA, CCS, CPZ; ARDR is device-linked), tiered existing-pilot vs next-level

> Rubric text is in English (project working language). Reports are in **French**,
> so anchor examples are given in English with representative **French cues** in
> parentheses; a French lexical-cue appendix is at the end for the NLP pipeline.

---

## General rules (all dimensions)

- Label what the text **says**, not what should have been true clinically.
- Work from report **sections**: `Indication / Renseignements cliniques`, `Technique`,
  `Description / Résultats`, `Conclusion / Impression`.
- Score the **Conclusion / Impression** unless a dimension states otherwise.
- When torn between two levels, pick the **worse** one and flag `litigious = yes`.
- Trauma XR context: the central clinical question is usually **"is there a fracture
  (or dislocation/effusion/eligible bone finding) or not?"** - keep that lens.

---

# PART A - Report quality

## Q1 · Ambiguity of the conclusion (`ambiguity`) · primary: unambiguous / ambiguous

**Question:** does the conclusion give the referring clinician a clear answer, or does
it leave the key trauma question unresolved?

**A conclusion is AMBIGUOUS when any of these hold:**
- It hedges on the presence of an eligible finding **without resolving it** and
  **without an actionable next step** (e.g. "possible fracture" with no recommendation).
- It defers entirely to clinic without committing ("to be correlated clinically" as the
  *only* content - see Q2 note).
- It is internally inconsistent (description vs conclusion, or self-contradiction).
- It is empty / non-informative.

**A conclusion is UNAMBIGUOUS when:**
- It states a definite result (fracture present / no fracture), OR
- It hedges but **resolves the uncertainty with a concrete action** (e.g. "undisplaced
  fracture not excluded → CT recommended").

### Primary label (binary) + optional 3-point severity

| Severity | Label | Definition | Anchor example (EN · FR cue) |
|----------|-------|-----------|------------------------------|
| 0 | **Unambiguous** | Clear answer or hedge resolved by action | "No fracture. Normal trauma series." (*Pas de fracture. Examen normal.*) / "Non-displaced radial head fracture; comparative views advised." |
| 1 | **Minor ambiguity** | Mostly clear, minor hedging, still actionable | "Probable fracture of the 5th metatarsal base; correlate with point tenderness." (*Fracture probable… à corréler à la douleur localisée.*) |
| 2 | **Ambiguous** | Unresolved hedge, no action, or contradictory | "Images that may correspond to a fracture line, to be correlated." (*Images pouvant correspondre à un trait de fracture, à corréler.*) |

> Binary mapping: severity 0–1 → **unambiguous**, severity 2 → **ambiguous**.
> Report **both** if your gold set uses the 3-point scale.

**Edge cases (trauma XR):**
- Clean normal report ("*Pas de lésion osseuse traumatique.*") → **unambiguous (0)**.
  A clear negative is not ambiguous.
- "*Doute sur une fracture, avis spécialisé recommandé*" → hedge **with** action →
  **unambiguous (1)** (uncertainty is resolved by a next step).
- "*Pas d'argument formel pour une fracture*" with no further guidance → borderline;
  usually **minor ambiguity (1)** - flag if unsure.

---

## Q2 · Follow-up recommendation present (`followup_present`) · yes / no

**Question:** does the report recommend a further action / imaging / review?

**Counts as YES (French cues):**
- Repeat/other imaging: "*contrôle radiographique à 10 jours*", "*compléter par un scanner*",
  "*IRM recommandée*", "*nouveau cliché après immobilisation*".
- Specialist review as an action: "*avis orthopédique recommandé*".
- Explicit surveillance: "*surveillance conseillée*", "*à réévaluer*".

**Counts as NO:**
- No mention of any further action.
- Diagnosis/description only.
- **"*à corréler à la clinique*" alone is NOT a follow-up** (it is not an order for an
  action or exam). Flag `litigious` if borderline.

| Label | Anchor example (EN · FR cue) |
|-------|------------------------------|
| **Yes** | "Follow-up radiograph in 10–14 days if symptoms persist." (*Contrôle radiographique à 10–14 jours si persistance.*) |
| **No** | "Undisplaced distal radius fracture." with no further advice (*Fracture non déplacée du radius distal.*) |

---

## Q3 · Follow-up completeness (`followup_completeness`) · complete / incomplete / NA

> Label **only when Q2 = yes.** Otherwise `NA`.

**Required slots** (mark presence of each):

> **Slots per the Groupe 3R checklist §7** ("Si proposition d'examen supplémentaire faite"):
> the three required slots are **délai**, **modalité**, and **ce qu'on attend** (expected purpose).
> (This replaces the earlier region/condition slots; the checklist is the source of truth.)

| Slot | Meaning | Example (FR) |
|------|---------|--------------|
| `delai` (timeframe) | Within what delay | *à 10 jours, à 6 semaines, à 3 mois* |
| `modalite` (modality) | By which exam | *radiographie, TDM/scanner, IRM, échographie* |
| `attendu` (expected purpose) | What is expected of the exam | *pour confirmer la fracture, pour évaluer l'extension* |

**Rule:** any missing **required** slot (`delai`, `modalite`, `attendu`) ⇒ **incomplete**.

| Label | Example (EN · FR) | Missing |
|-------|-------------------|---------|
| **Complete** | "CT in 2 weeks to confirm the fracture." (*Scanner à 2 semaines pour confirmer la fracture.*) | - |
| **Incomplete** | "Follow-up recommended." (*Contrôle recommandé.*) | delai, modalite, attendu |
| **Incomplete** | "Repeat CT advised." (*Nouveau scanner conseillé.*) | delai, attendu |

Record **which slots are missing** - it is actionable feedback for the reporting physician.

---

# PART B - BoneView appropriateness

> BoneView (Gleamer) intended use - the yardstick for Part B:
>
> | Attribute | In scope |
> |-----------|----------|
> | Modality | Conventional **radiograph (XR)** only |
> | Body regions | **Limbs** (whole appendicular skeleton), **pelvis**, **thoracic & lumbar spine**, **rib cage** |
> | Findings detected | **Fracture, dislocation, joint effusion, bone lesion** |
> | Patient age | **≥ 2 years** |
> | Out of scope | Skull, face, **cervical spine**, chest/abdomen soft-tissue reads, CT/MRI/US, age < 2 y |

## B1 · Order conformity to BoneView intended use (`order_in_scope`) · in-scope / out-of-scope

**Question (from the ORDER / prescription + exam metadata):** is this exam something
BoneView is intended to run on?

**Decision (all three must hold for in-scope):**
1. Modality is a **radiograph** (not CT/MRI/US).
2. Body region ∈ {limbs, pelvis, thoraco-lumbar spine, rib cage}.
3. Patient age **≥ 2 years** (if age available; if unknown, mark `age_unknown` and judge on modality+region).

**Label + reason code when out-of-scope:** `R_modality`, `R_region`, `R_age`.

| Label | Example order (EN · FR cue) | Reason |
|-------|------------------------------|--------|
| **In-scope** | "XR left ankle, trauma" (*Radiographie cheville gauche, traumatisme*) | - |
| **In-scope** | "XR pelvis after fall" (*Bassin, chute*) | - |
| **Out-of-scope** | "XR cervical spine" (*Rachis cervical*) | R_region (cervical excluded) |
| **Out-of-scope** | "XR skull" (*Crâne*) | R_region |
| **Out-of-scope** | "Chest XR, dyspnea" (*Thorax, dyspnée*) | R_region (soft-tissue chest, not rib trauma) |
| **Out-of-scope** | "CT wrist" (*Scanner poignet*) | R_modality |
| **Out-of-scope** | limb XR, infant 1 y | R_age |

> Note on rib cage vs chest: an order for **rib series / rib trauma** is in-scope;
> an order for **chest (lung) evaluation** is out-of-scope even though it images the thorax.
> Judge by the clinical intent in the order.

## B2 · BoneView-eligible finding in the conclusion (`eligible_finding`) · present / absent

**Question (from the report CONCLUSION):** does it describe a finding BoneView is designed
to detect, in an **in-scope region**?

- Eligible finding types: **fracture, dislocation, joint effusion, bone lesion**.
- Record `finding_types` (one or more) and `body_region`.
- A finding in an out-of-scope region (e.g. skull fracture) → `absent` for BoneView
  purposes, but note it in `comment` (record `out_of_scope_finding = yes`).

| Label | Example conclusion (EN · FR cue) | finding_types / region |
|-------|----------------------------------|------------------------|
| **Present** | "Non-displaced fracture of the distal radius." (*Fracture non déplacée du radius distal.*) | fracture / wrist |
| **Present** | "Lipohemarthrosis of the knee, suggesting occult fracture." (*Épanchement/lipohémarthrose du genou.*) | effusion (±fracture) / knee |
| **Present** | "Anterior shoulder dislocation." (*Luxation antérieure de l'épaule.*) | dislocation / shoulder |
| **Absent** | "No fracture, no dislocation." (*Pas de fracture ni luxation.*) | - |
| **Absent (note)** | "Undisplaced skull fracture." | out-of-scope region → `out_of_scope_finding=yes` |

**Why B1 and B2 together:** B1 says *should BoneView have run*; B2 says *was there an
eligible finding to catch*. Cross-tabulated with BoneView's own output they give
appropriateness and concordance of the deployment.

---

# PART C - Report-form QC metrics (Groupe 3R checklist)

> **Source of truth:** the Groupe 3R "Checklist Comptes Rendus Radiologiques" (Mars 2013).
> Label only what the checklist asks. Items gated by *si applicable / si nécessaire* are
> marked **NA** when not applicable (not "fail"). Full metric spec: [`qc-metrics-spec.md`](qc-metrics-spec.md).
> **Tier** shows which metrics are already piloted vs next level.

| Metric | Gold field | What to label (checklist rule) | Tier |
|--------|-----------|--------------------------------|------|
| SRC (structure) | `src_compliant` (yes/no) | All expected sections/headings present, layout respected (§10) | pilot |
| SGE (spelling) | `sge_band` (none/minor/moderate/major) | Uncorrected typos / speech-recognition / grammar, ignoring medical jargon (§10) | pilot |
| SEF laterality | `sef_laterality` (none/mismatch) | Droite/gauche consistent across Titre / Contenu / Conclusion (§2,§5,§6) | pilot |
| SEF discordance | `sef_discordance` (none/within/vs_prior) | Contradiction within report, or vs a prior report same patient (§8) | next |
| AMI (ambiguity) | `ami_clear_conclusion` (yes/no) | Is there a clear conclusion / diagnosis or differential? (§6) | pilot |
| CQA (question) | `cqa` (answered/partial/not/NA) | Was the referring question answered? (§3 + §6) | next |
| CCS (completeness) | `ccs` (0-5) + `ccs_missing` | Share of applicable §6 conclusion items satisfied (see below) | next |
| CPZ (prioritization) | `cpz` (pass/fail/NA) | Conclusions prioritized: answer first, then decreasing importance (§6) | next |

**CCS (0-5) - applicable §6 items** (mark each yes/no/NA, then `CCS = round(5 × satisfied / applicable)`):
size · localisation · side · quantification (arthrosis degree / effusion importance) · comparison with prior ·
stability vs prior · date of prior · question answered · clear conclusion (diagnosis/differential) ·
prioritized · adequate proposal to clinician.

> **Not in the checklist** (do NOT label as defects): "one line per diagnosis / no verbosity", and any
> forbidden-phrase blacklist ("à corréler", "sans changement") - these are heuristics, not checklist rules.

**ARDR** is computed from BoneView output + report text (region-level), not pure annotation; the human
label needed is only whether the report mentions a finding in the flagged **region** (`ardr_region_mentioned`).

---

## Per-report record (all dimensions)

Minimum fields per report (see [`06-working-forms.md`](06-working-forms.md)):

```
report_id, annotator_id,
# Part A
ambiguity (0/1/2 or unambiguous/ambiguous),
followup_present (yes/no),
followup_completeness (complete/incomplete/NA), followup_missing_slots (delai/modalite/attendu),
# Part B
order_in_scope (in/out), order_reason (R_modality/R_region/R_age/-),
eligible_finding (present/absent), finding_types, body_region, out_of_scope_finding,
# Part C (checklist QC)
src_compliant, sge_band, sef_laterality, sef_discordance, ami_clear_conclusion,
cqa, ccs (0-5), ccs_missing, cpz, ardr_region_mentioned,
comment, litigious (yes/no)
```

## Calibration (before main labeling)

Run a joint session on 10–15 trauma XR reports: each annotator labels independently,
then reconcile disagreements and refine this rubric. Re-measure agreement (κ) per
dimension. Freeze the test split before any modeling.

## Appendix - French lexical cues (for the NLP pipeline, not exhaustive)

- **Fracture:** *fracture, trait de fracture, fêlure, arrachement osseux, tassement*
- **Dislocation:** *luxation, subluxation, déboîtement*
- **Effusion:** *épanchement, hémarthrose, lipohémarthrose*
- **Bone lesion:** *lésion osseuse, lyse, image lytique/condensante, lésion suspecte*
- **Follow-up / action:** *contrôle, à réévaluer, surveillance, compléter par, nouveau cliché, avis (orthopédique/spécialisé), IRM/scanner recommandé(e)*
- **Negation:** *pas de, absence de, sans, pas d'argument pour, non visualisé(e)*
- **Hedging (ambiguity signal):** *possible, probable, ne peut être exclu(e), suspicion de, à corréler, doute*
