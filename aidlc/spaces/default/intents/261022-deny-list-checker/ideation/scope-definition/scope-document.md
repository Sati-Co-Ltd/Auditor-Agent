# Scope Document: 2026 NHSO DENY-list checker

Upstream: `aidlc/spaces/default/intents/261022-deny-list-checker/ideation/intent-capture/intent-statement.md` (intent statement), `aidlc/spaces/default/intents/261022-deny-list-checker/ideation/feasibility/feasibility-assessment.md` (feasibility assessment), `aidlc/spaces/default/intents/261022-deny-list-checker/ideation/feasibility/constraint-register.md` (constraint register), and `scope-definition-questions.md` (Q1-Q2).

## Minimum Viable Scope

A medical-record coder gives one case's fund and coded ICD-10-TM diagnoses and gets back either the case's 2026 NHSO DENY risks, each citing its DENY rule (code and source row), "no DENY risk", or "not applicable" for a non-UC fund. It runs locally, is checked against the sample cases, and is reachable through a minimal HTTP API. (Intent statement; scope Q&A summary.)

## In Scope (this release)

| Capability | Priority | Source |
|------------|----------|--------|
| Load the DENY seed list (`inputs/deny-list-seed-2569.csv`) with the source row of every entry | Must | Intent statement |
| Check one case: fund plus ICD-10-TM codes, giving DENY risks with citations or "no DENY risk" | Must | Intent statement |
| UC-only rule: "UC" = "UC (NHSO)"; SSO, CSMBS and private return "not applicable" | Must | Intent statement; constraint register C-R2 |
| ICD code and range matching (A41.9 is inside A40-A41; lists such as "E86,E87.7") | Must | Intent statement; feasibility assessment |
| Automated tests from `inputs/sample-cases.csv`; a case passes when its expected code is contained in the findings | Must | Intent statement |
| Minimal HTTP API for the single-case check | Must | Intent statement |
| Risk-level flag on every response: `DENY_RISK` / `NO_RISK` / `NOT_APPLICABLE` | Must | Scope expansion 22 Oct |
| Batch check of several cases in one request; response includes risk-level per case and a risky-cases-only filter | Must | Scope expansion 22 Oct (promoted from Could) |
| ICD-9 procedure code check against guideline criteria (requires `private-inputs/icd9-criteria.csv`) | Could | Scope expansion 22 Oct |
| Lab and evidence check against guideline criteria (requires `private-inputs/lab-criteria.csv`) | Could | Scope expansion 22 Oct |
| Guideline reference (GL) citation on every failed criterion, alongside the DENY citation | Could | Scope expansion 22 Oct (promoted from Won't, IB-9) |

## Out of Scope (this release)

| Item | Reason | Source |
|------|--------|--------|
| Real patient data, HIS integration | PDPA; fictional cases only | Intent statement; constraint register C-R1 |
| Separate web front-end or UI application | API returns structured risk data; a future run builds the UI layer | Intent statement |
| LLM agent calls | Check is deterministic rules only | Constraint register C-T3 |
| Documentation-completeness rules, other funds' rulebooks | Not in first release | Intent statement |
| Deployment to SATI's dev environment | Separate run; this one is local only | Intent statement; constraint register C-O2 |
| Reading the official list from Excel | Added when the official file arrives | Constraint register C-T4 |

## Sequencing

Thin end-to-end slice first (walking skeleton): CR1 sepsis through load, check, HTTP API and test, then widen to every rule in the seed list (Q2). This fits the 22 Oct 13:45-16:30 time box from the constraint register (C-O1).

## Scope vs Timeline Check

The six must-haves are small and have no outside dependencies (feasibility assessment), so they fit the time box. The batch check is held back as a Could so it can be dropped without missing the release (Q1).
