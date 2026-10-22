# Feasibility Assessment: 2026 NHSO DENY-list checker

Inputs: `aidlc/spaces/default/intents/261022-deny-list-checker/ideation/intent-capture/intent-statement.md` (intent statement), `feasibility-questions.md` (Q1-Q4), and the team's technical environment note (`aidlc/spaces/default/knowledge/aidlc-shared/technical-environment.md`). No market research ran for this workflow.

## Verdict

**Feasible within the time box, with low technical risk.** The first release is a deterministic lookup: a case's fund and coded diagnoses are compared with a small rule list, and each match is returned with its citation. Nothing needs a model, a database, or an outside system, and every input the release needs is already in the project.

## Technical Viability

| Area | Assessment | Basis |
|------|------------|-------|
| Rule data | Available now: `inputs/deny-list-seed-2569.csv`, 13 rows, 10 distinct DENY codes, each with a guideline source row | Intent statement; Q2 |
| Test truth | Available now: 3 fictional sample cases in `inputs/sample-cases.csv` (one positive, two negative) | Intent statement; Q4 |
| Core logic | Low complexity: fund check (UC only), code-to-range matching (A41.9 inside A40-A41), citation output | Intent statement |
| Integrations | None in this release: no HIS, no real patient data, no deployment | Intent statement |
| Determinism | Achievable: rules only, no model call; this matches the team's ban on model calls | Technical environment note |
| Later data swaps | The official 2026 list and Pod B's 15-25 cases must load without code changes; the list's format is unknown (CSV now, Excel later) | Q2; Q3 |

## Risk Analysis

| Risk | Likelihood | Impact | Response |
|------|------------|--------|----------|
| The time box (22 Oct, 13:45-16:30) is too short if scope grows | Medium | High | Hold to the core check, tests and API (Q1); anything else waits for a later run |
| Code-range matching gets an edge case wrong (single code vs range vs list such as "E86,E87.7") | Medium | High | Requirements state the matching rules; tests cover a range, a single code and a list |
| The official 2026 list arrives in a different shape or format than the seed list | Medium | Medium | Keep the list as data loaded at start-up; add Excel reading when the file arrives (Q3) |
| 3 ground-truth cases are thin proof of correctness | High | Medium | Accepted for this release; Pod B extends to 15-25 cases (intent statement) |
| Fund labels differ ("UC" vs "UC (NHSO)") | High | Low | Requirements fix the mapping UC = UC (NHSO) (intent statement) |

## Assumptions & Open Questions

- One finding or several per code is still decided in Requirements Analysis (intent statement).
- Whether a finding shows the guideline's stricter criteria text is still open (intent statement).
