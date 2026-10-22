# RAID Log: 2026 NHSO DENY-list checker

Upstream: `aidlc/spaces/default/intents/261022-deny-list-checker/ideation/intent-capture/intent-statement.md`.

## Risks

| ID | Risk | Likelihood | Impact | Owner | Response |
|----|------|------------|--------|-------|----------|
| R1 | Scope grows beyond the 13:45-16:30 build window | Medium | High | K'Pop (product owner) | Hold to core check, tests and API (Q1) |
| R2 | ICD code-range matching is wrong for an edge case | Medium | High | Build team | Matching rules in requirements; tests for range, single code and list |
| R3 | Official 2026 list differs in shape or format from the seed list | Medium | Medium | Build team | List kept as data; Excel support added when the file arrives (Q3) |
| R4 | 3 sample cases are too few to prove correctness | High | Medium | Pod B | Extend ground truth to 15-25 cases (intent statement) |

## Assumptions

| ID | Assumption | Status | Owner |
|----|------------|--------|-------|
| A1 | One finding or several per code is decided in Requirements Analysis; meanwhile a sample case passes when its expected code is contained in the findings | Accepted (intent statement) | K'Pop |
| A2 | Whether a finding shows the guideline's stricter criteria text is open | Open (intent statement) | K'Pop |
| A3 | The seed list is replaced by SATI's official 2026 list when it arrives | Accepted (intent statement) | K'Pop |

## Issues

| ID | Issue | Status | Response |
|----|-------|--------|----------|
| I1 | Fund labels differ between inputs ("UC" in sample cases, "UC (NHSO)" in the seed list) | Open | Map UC = UC (NHSO) in requirements (intent statement) |
| I2 | D61 and I25 each appear in two seed-list rows | Open | Settled with A1 in Requirements Analysis (intent statement) |

## Dependencies

| ID | Dependency | Provider | Needed for this release? |
|----|------------|----------|--------------------------|
| D1 | SATI's official 2026 DENY list | SATI | No: release ships on the seed list (Q2) |
| D2 | Ground truth extended to 15-25 fictional cases | Pod B | No: release ships on the 3 sample cases (Q2) |
| D3 | Deployment to SATI's dev environment | Separate run | No: this run is local only (intent statement) |
