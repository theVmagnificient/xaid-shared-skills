---
name: radpeer-grade-abdomen
description: Compare two CT abdomen/pelvis radiology reports at the finding level using RADPEER-adapted grading
---

Compare **Report A** against **Report B** for a **CT abdomen / abdomen+pelvis**
study at the finding level and grade each discrepancy.

This skill is the abdomen variant of the `radpeer-grade` family. The detailed
clinical rules (liver lesion management per ACR 2017, Bosniak renal cyst
classification, adrenal adenoma washout, pancreatic incidentaloma follow-up,
AAA thresholds, etc.) are loaded as a knowledge block by the caller before
this prompt — apply them as authoritative.

## Args

Provide two report texts in any format:
- Paste directly: `Report A: <text> / Report B: <text>`
- Pass file paths, accession numbers, or DB query results — the skill works regardless of source

Optionally label the reports (e.g. "Prelim vs Final", "Doctor vs Validator",
"Validator vs SimonMed").

---

## Methodology

### Step 1 — Extract findings from each report

Break each report into atomic findings. A finding is any discrete observation:
a liver lesion, a renal cyst, an adrenal nodule, free fluid, a lymph node, a
vascular calcification, a normal organ explicitly noted, a recommendation, etc.

Cover all major abdominal/pelvic structures: liver, gallbladder, pancreas,
spleen, adrenals, kidneys, bowel, mesentery, peritoneum, vasculature
(aorta, IVC, mesenteric vessels), pelvic organs, lymph nodes, bones.

Include findings from all sections: Findings and Impression (Indication/Technique are metadata, skip).

List findings for Report A and Report B separately before comparing.

### Step 2 — Extract and enumerate findings (Chain-of-Thought)

Before comparing anything, explicitly list findings from each report as numbered lists:

**Report A findings:**
1. <finding category> — <organ/location>: <exact description>
2. ...

**Report B findings:**
1. <finding category> — <organ/location>: <exact description>
2. ...

Then explicitly map each finding to its counterpart:

**Matching:**
- A1 ↔ B2 — same renal cyst, different Bosniak classification
- A3 ↔ (none) — only in A
- (none) ↔ B4 — only in B
- ...

Do this mapping step fully before assigning any grades. This prevents missed
findings and makes reasoning auditable.

Three possible outcomes per finding:
- **Matched** — both reports mention it (grade the agreement)
- **Only in A** — A mentions it, B does not (potential overreport)
- **Only in B** — B mentions it, A does not (potential underreport)

### Step 3 — Grade each finding pair

| Grade | Label | Definition |
|-------|-------|------------|
| 1 | Concordant | Same clinical meaning — wording may differ but interpretation is equivalent |
| 2a | Minor Stylistic | Different wording or format only, no clinical difference |
| 2b | Minor Clinical | Finding added, modified, or characterized differently — does NOT change patient management |
| 3 | Significant Underreport | Finding missed or downgraded in A — would change patient management |
| 4a | Minor Overreport | False or overcalled finding in A — clinically minor, no management impact (e.g. simple cyst falsely called complex, small benign-appearing nodule mentioned that is not present) |
| 4b | Significant Overreport | False or overcalled finding in A — would cause additional follow-up, workup, or treatment (e.g. spurious mass requiring MRI, falsely upgraded Bosniak that triggers surgery referral) |

**Management change** = changes recommended follow-up, triggers additional
imaging (MRI, CT follow-up, US), biopsy, surgical referral, or alters treatment.

### Step 4 — Output per finding

For each finding, output:

```
Finding: <category> — <organ/anatomical location>
In A: <exact description or "not mentioned">
In B: <exact description or "not mentioned">
Discrepancy: <what differs>
Grade: <1 / 2a / 2b / 3 / 4a / 4b>
Management impact: <none / <description of impact>>
```

### Step 5 — Study-level summary

After all findings:

```
Total findings: N
Grade 1:  N (X%)
Grade 2a: N (X%)
Grade 2b: N (X%)
Grade 3:  N (X%)
Grade 4a: N (X%)
Grade 4b: N (X%)

Overall study grade: <worst grade>
Clinical concordance (1+2a): X%

Key discrepancies (2b+):
- <finding>: <one-line summary>
```

---

## Abdomen-specific rules

- Liver lesions: size in mm, density (hypo/iso/hyper), enhancement pattern
  when contrast-enhanced. Apply ACR 2017 incidental liver lesion algorithm
  from the knowledge block. Indeterminate >1 cm in oncology patient triggers
  MRI — omitting that recommendation is a clinically meaningful gap.
- Renal lesions: Bosniak classification on contrast-enhanced studies; size on
  any study. Bosniak IIF/III/IV vs simple cyst is a clinically meaningful
  distinction (grade 2b at minimum).
- Adrenal nodules: size + density (HU). <10 HU on noc = adenoma; otherwise
  apply washout protocol guidance from the knowledge block.
- Pancreatic findings: any solid lesion, ductal dilatation >3 mm, or cystic
  lesion needs explicit characterization and management recommendation.
- AAA: aneurysm threshold (>3 cm aorta) and follow-up interval per size
  (apply guideline from the knowledge block). Missing AAA at threshold is
  grade 3.
- Free fluid: small / moderate / large categorization is clinically relevant.
- Bowel: wall thickening, obstruction, pneumatosis — never stylistic.
- Lymph nodes: short-axis ≥10 mm is positive in most stations; mesenteric
  rules differ — apply knowledge block.

## General rules

- Be exhaustive — do not skip minor or normal findings. Every observation counts.
- When in doubt about management impact, err toward the more severe grade and note uncertainty.
- Normal/negative findings explicitly stated in one report but absent in the
  other are grade 2a (stylistic), not underreport — unless the omission is
  clinically meaningful (per body-area rules above).
- Recommendations (e.g. "follow-up MRI in 6 months") are findings — grade them.
- Do not penalize stylistic differences in Russian vs English or different template structures.
- Incidental thoracic findings (lung bases visible on abdomen CT) — grade
  them using chest rules if the knowledge block loads them.
