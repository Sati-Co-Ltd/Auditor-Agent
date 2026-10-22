# Constraint Register: 2026 NHSO DENY-list checker

Upstream: `aidlc/spaces/default/intents/261022-deny-list-checker/ideation/intent-capture/intent-statement.md`.

## Technical Constraints

| ID | Constraint | Source |
|----|------------|--------|
| C-T1 | Language and tooling are fixed by the team: Python 3.12, pip with requirements.txt, FastAPI, pytest | Technical environment note |
| C-T2 | No database; rule data is loaded from files at start-up | Technical environment note |
| C-T3 | No model (LLM) calls; the check is deterministic rules only | Technical environment note; intent statement success metrics |
| C-T4 | No pandas; read files with the standard csv module, and openpyxl when Excel is needed | Technical environment note; Q3 |
| C-T5 | The DENY list and ground-truth cases must be replaceable without code changes | Q2 |
| C-T6 | Input is validated at the boundary (Pydantic); no secrets in code | Technical environment note |

## Organizational Constraints

| ID | Constraint | Source |
|----|------------|--------|
| C-O1 | Time box: the 22 Oct co-build afternoon, build window 13:45-16:30; core check, tests and API only | Q1 |
| C-O2 | This run is local only; deployment to SATI's dev environment is a separate run | Intent statement |
| C-O3 | K'Pop (SATI CEO and product owner) decides scope and priority; SATI's clinical coders own the ground truth | Intent statement (stakeholder map) |

## Regulatory Constraints

| ID | Constraint | Source |
|----|------------|--------|
| C-R1 | PDPA: no real patient data; all test cases are fictional with no names, IDs or dates of real patients | Technical environment note; Q4 |
| C-R2 | NHSO DENY rules apply to UC (NHSO) claims only; SSO, CSMBS and private return "not applicable" | Intent statement |
| C-R3 | Every finding cites its DENY rule (code and guideline source row), for auditability | Intent statement |
| C-R4 | Keep the repository private; internal dev use only | Technical environment note |
