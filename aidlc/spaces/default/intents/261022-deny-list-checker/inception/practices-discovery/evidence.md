# Practices Discovery Evidence (DRAFT)

> **Status: DRAFT, lead draft only.** This run is lead-only: no support
> agents were dispatched and there are no contribution files. The interview
> decisions are added at integration.

## Sources Inspected

| Source | What it gave us |
|--------|-----------------|
| `aidlc/spaces/default/memory/org.md` | Suggested defaults for all five areas: trunk-based with squash to `main`; walking skeleton when the scope has it on; test-after with an 80% line-coverage floor and CI before merge for `mvp`; deploy on merge to staging with a manual production gate; Black/Ruff-style tool-driven code style. Treated as suggestions, not team facts. |
| `aidlc/spaces/default/memory/team.md` | Empty template; no affirmed practices, so this is a first run, not a re-run. |
| `aidlc/spaces/default/memory/project.md` | Empty template; no project-level specialisation. |
| `aidlc/spaces/default/knowledge/aidlc-shared/technical-environment.md` | Team-written hard constraints: Python 3.12, pip + `requirements.txt`, FastAPI + Mangum on Lambda, AWS `ap-southeast-1` SATI dev account, GitHub Actions + OIDC, pytest with `inputs/sample-cases.csv`; prohibitions (database, LLM calls, pandas, real patient data); security (Pydantic validation, no secrets in code, private repo, internal dev only). These became `discovered-rules.md`. |
| `aidlc/spaces/default/intents/261022-deny-list-checker/ideation/scope-definition/scope-document.md` | The person's choice of a thin end-to-end slice first (CR1 sepsis through load, check, API and test, then widen); the test pass rule (expected code contained in the findings); deployment to SATI dev is out of scope for this run. |
| `aidlc/spaces/default/intents/261022-deny-list-checker/ideation/feasibility/constraint-register.md` | Time box 22 Oct 13:45-16:30 (C-O1); local-only run (C-O2); PDPA fictional data (C-R1); private repo (C-R4). |
| `.claude/scopes/aidlc-mvp.md` | `skeleton: on` for the `mvp` scope. |
| `inputs/sample-cases.csv` (header and first rows) | Columns `case_id, fund, coded_diagnoses, expect_deny, deny_code, why`; GT-001 is the UC sepsis case for CR1, GT-002 an SSO case. Confirms the suite can be data-driven from the file. |
| Git repository | Branch `main`, no commits, everything untracked. No CI config, no linter or formatter config, no code. Nothing to infer from history. |

## Inferences

- **Way of Working**: trunk-based with squash to `main` is a natural fit: one
  person or a small mob, a single afternoon, no release cadence to protect.
  Inferred, not stated.
- **Walking Skeleton**: on. This is the one area already decided by the
  person (scope Q2) and backed by the `mvp` scope; the draft only proposes a
  verification command.
- **Testing Posture**: test-after is the org default and the draft keeps it,
  but the sample-case CSV already holds expected results, which makes a
  test-first variant cheap. The 80% coverage floor and CI-before-merge come
  from the `mvp` default and may not fit a local-only afternoon.
- **Deployment**: nothing deploys in this run (C-O2). The later run's target
  and mechanism are fixed by the technical environment note; the cadence
  (deploy on merge to dev) and the smoke test and rollback approach are
  suggestions.
- **Code Style**: no tool config exists; Ruff for lint and format is a
  suggestion to keep the toolchain small. The org default names Black.
- Product rules from the constraint register (UC-only, citation per finding,
  replaceable data files) are requirements, not team practices, so they are
  not listed in `discovered-rules.md`; they belong to requirements analysis.

## Open Points for the Interview

1. **Way of Working**: keep trunk-based with squash-merge to `main`? Is a
   GitHub pull request needed for each merge in this run, or is a local
   squash-merge after the Bolt approval enough?
2. **Walking Skeleton**: confirm the thin slice (CR1 sepsis through load,
   check, API, test) and the command that proves it, suggested `pytest` with
   an API-level test for GT-001; optionally a live `uvicorn` request too.
3. **Testing Posture**: test-after (build each layer, then test it) or write
   the sample-case tests first? Keep the 80% line-coverage floor with
   `pytest-cov`? Does CI (a GitHub Actions test workflow) belong in this run,
   or does a local `pytest` run gate merges until the deployment run?
4. **Deployment**: confirm this run deploys nothing; for the later SATI dev
   run, start from deploy on merge to `main` with a smoke test and Lambda
   alias rollback, or leave it for that run to decide?
5. **Code Style**: Ruff for lint and format, Black plus Ruff, or no enforced
   formatter this afternoon? Split dev tools into `requirements-dev.txt`?

## Unresolved Uncertainty

- Whether the team has a GitHub repository set up yet; it affects the PR and
  CI answers.
- Whether `pytest-cov` and a linter are acceptable as dev dependencies; the
  technical environment note lists only pytest.
- Methodology, coverage, CI, and code-style choices remain suggestions until
  the person answers.
