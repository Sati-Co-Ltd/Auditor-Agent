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
| Batch check of several cases in one request, for auditors | Could (only if the core is done early) | Q1 |

## Out of Scope (this release)

| Item | Reason | Source |
|------|--------|--------|
| Real patient data, HIS integration | PDPA; fictional cases only | Intent statement; constraint register C-R1 |
| User interface, LLM agent | Not in first release; deterministic rules only | Intent statement; constraint register C-T3 |
| Documentation-completeness rules, other funds' rulebooks | Not in first release | Intent statement |
| Deployment to SATI's dev environment | Separate run; this one is local only | Intent statement; constraint register C-O2 |
| Reading the official list from Excel | Added when the official file arrives | Constraint register C-T4 |
| Showing the guideline's stricter criteria text | Still an open question | Intent statement |

## Sequencing

Thin end-to-end slice first (walking skeleton): CR1 sepsis through load, check, HTTP API and test, then widen to every rule in the seed list (Q2). This fits the 22 Oct 13:45-16:30 time box from the constraint register (C-O1).

## Scope vs Timeline Check

The six must-haves are small and have no outside dependencies (feasibility assessment), so they fit the time box. The batch check is held back as a Could so it can be dropped without missing the release (Q1).
