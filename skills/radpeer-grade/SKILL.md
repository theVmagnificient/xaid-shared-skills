---
name: radpeer-grade
description: Compare two radiology reports at the finding level using RADPEER-adapted grading
---

Compare **Report A** against **Report B** at the finding level and grade each discrepancy.

## Args

Provide two report texts in any format:
- Paste directly: `Report A: <text> / Report B: <text>`
- Pass file paths, accession numbers, or DB query results — the skill works regardless of source

Optionally label the reports (e.g. "Prelim vs Final", "Doctor vs Validator").

---

## Methodology

### Step 1 — Extract findings from each report

Break each report into atomic findings. A finding is any discrete observation: a nodule, an effusion, a calcification, a normal structure explicitly noted, a recommendation, etc.

Include findings from all sections: Findings and Impression (Indication/Technique are metadata, skip).

List findings for Report A and Report B separately before comparing.

### Step 2 — Extract and enumerate findings (Chain-of-Thought)

Before comparing anything, explicitly list findings from each report as numbered lists:

**Report A findings:**
1. <finding category> — <location>: <exact description>
2. ...

**Report B findings:**
1. <finding category> — <location>: <exact description>
2. ...

Then explicitly map each finding to its counterpart:

**Matching:**
- A1 ↔ B2 — same nodule, different size description
- A3 ↔ (none) — only in A
- (none) ↔ B4 — only in B
- ...

Do this mapping step fully before assigning any grades. This prevents missed findings and makes reasoning auditable.

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
| 4 | Significant Overreport | False or overcalled finding in A — would cause unnecessary workup |

**Management change** = changes recommended follow-up, triggers additional imaging, biopsy, referral, or alters treatment.

### Step 4 — Output per finding

For each finding, output:

```
Finding: <category> — <anatomical location>
In A: <exact description or "not mentioned">
In B: <exact description or "not mentioned">
Discrepancy: <what differs>
Grade: <1 / 2a / 2b / 3 / 4>
Management impact: <none / <description of impact>
```

### Step 5 — Study-level summary

After all findings:

```
Total findings: N
Grade 1:  N (X%)
Grade 2a: N (X%)
Grade 2b: N (X%)
Grade 3:  N (X%)
Grade 4:  N (X%)

Overall study grade: <worst grade>
Clinical concordance (1+2a): X%

Key discrepancies (2b+):
- <finding>: <one-line summary>
```

---

## Rules

- Be exhaustive — do not skip minor or normal findings. Every observation counts.
- When in doubt about management impact, err toward the more severe grade and note uncertainty.
- Normal/negative findings explicitly stated in one report but absent in the other are grade 2a (stylistic), not underreport — unless the omission is clinically meaningful.
- Recommendations (e.g. "follow-up CT in 3 months") are findings — grade them.
- Do not penalize stylistic differences in Russian vs English or different template structures.
