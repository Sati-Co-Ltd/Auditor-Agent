# Intent Statement: 2026 NHSO DENY-list checker

## Problem Statement

Medical-record coders find 2026 NHSO DENY risks only after NHSO rejects a claim, because the 2026 DENY list is not in the SATI Auditor Agent's knowledge base. [Q1]

The request is to build what `vision.md` describes, as a new project, with test cases from `inputs/sample-cases.csv`. [desc]

## Target Customer

| Who | How they benefit | Source |
|-----|------------------|--------|
| Medical-record coders at Sriphat Medical Center (primary) | Check a case before claim submission and see its DENY risks in time to act | [Q1] |
| Medical Statistics auditors (secondary) | Review DENY risks across cases | [Q1] |
| Doctors (indirect) | Affected by DENY risks without using the checker themselves | [Q9] |

## Success Metrics

| Metric | Target | Source |
|--------|--------|--------|
| Sample cases in `inputs/sample-cases.csv` (ground truth agreed by SATI's coders) giving the expected result: DENY risk Y/N, and the expected DENY code contained in the findings | 100% | [Q2] [Q8] |
| Cases with a non-UC fund (for example SSO) that return "not applicable" | 100% | [Q10] |
| DENY findings that cite their DENY rule (DENY code and source row in the DENY list) | 100% | [Q2] |
| Model calls per case | 0 (deterministic rules only) | [Q2] |
| DENY risks caught before claim submission, counted after go-live | Measured; no target set yet | [Q2] |

- Sample case GT-001 stays CR1-only for now. [Q8]
- The 3-case ground truth is thin proof; Pod B will extend it to 15-25 cases. [Q10]

## Initiative Trigger

NHSO's AI IP Pre-Audit training of 23 Jul 2569 set out DENY codes per diagnosis in the coding guideline's DENY_CODE_AI_IP_Audit column, and coders today only learn of these risks after rejection. [Q1] [Q3]

## DENY List Source

- The first release uses a seed list taken from the ICD-10-TM coding guideline's DENY_CODE_AI_IP_Audit column, each entry citing its guideline row as source; it is replaced by SATI's official 2026 list when that arrives. [Q3]
- The seed list is in the project as `inputs/deny-list-seed-2569.csv`, each row with its guideline row number as `source_row`. [Q6]
- It holds 10 distinct DENY codes in 13 rows; no codes are missing, and D61 and I25 each appear twice. [Q7]

## Initial Scope Signal

**Workflow-selected scope:** `mvp` (workflow-selected). [scope]

**User-confirmed product boundary — in this first version:** [Q9]

- The DENY check for a case, with a citation for every finding [Q9]
- The UC-only rule: NHSO DENY rules apply to UC claims only [Q9]
- Automated tests [Q9]
- An HTTP API [Q9]

**User-confirmed product boundary — not in this first version:** [Q5]

- Real patient data or the hospital information system (HIS) [Q5]
- A user interface [Q5]
- An LLM agent [Q5]
- Documentation-completeness rules [Q5]
- Other funds' rulebooks [Q5]
- Deployment to SATI's dev environment: this run is local only, and deployment is done in a separate run [Q5]

## Notes for Requirements Analysis

- Fund names: "UC" in the sample cases and "UC (NHSO)" in the seed list are the same fund. [Q10]
- One finding or several per code is decided in Requirements Analysis. [Q8]

## Assumptions & Open Questions

- [assumption] Whether the checker returns one finding or several when one ICD-10-TM code maps to several DENY codes or seed-list rows is not yet decided; Requirements Analysis decides it.
- [assumption] Whether a DENY risk should also show the guideline's stricter criteria text is an open question.
