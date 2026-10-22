# Team Practices (DRAFT)

> **Status: DRAFT, pending the practices interview.** Everything below is a
> suggested default taken from `aidlc/spaces/default/memory/org.md`, adjusted
> only where the team's technical environment note or the approved ideation
> artifacts already say something. Nothing here is affirmed yet. Items marked
> **(to confirm)** are the open points the interview resolves. This is a
> greenfield repo with no code and no commits, so there is no history to infer
> practices from.

## Way of Working

We work trunk-based on `main`. Each Construction Bolt runs on a short-lived
branch cut from `main` and is squash-merged back into `main` as one commit
named after the Bolt, so `main` reads as one commit per delivered slice. We do
not keep long-lived `develop` or release branches. A commit we ask for
outside a Bolt goes on the branch we are already on.

- Base branch: `main`; merge target: `main`; merge style: squash.
- The work is a single co-build afternoon (22 Oct, 13:45-16:30), so branches
  live for minutes to hours, not days.
- Pull requests and reviewer sign-off before merge: **(to confirm)** whether a
  GitHub pull request is required for each merge in this run, or whether a
  local squash-merge after the Bolt approval is enough.

## Walking Skeleton

We build a thin end-to-end slice first. The first slice is the CR1 sepsis
rule, taken all the way through: load the DENY seed list from
`inputs/deny-list-seed-2569.csv`, check one case, return the finding through
the HTTP API, and prove it with a pytest case from `inputs/sample-cases.csv`.
Only after that slice runs and the person approves it do we widen to every
rule in the seed list.

- This matches the sequencing the person already chose at scope definition,
  and the `mvp` scope has the skeleton switched on.
- The skeleton is proven by a real run, not a design review: the
  verification command must exercise load, check, API and test together.
- Suggested verification command: `pytest` from the repo root, including an
  API-level test that calls the check endpoint through FastAPI's test client
  with the CR1 sample case (GT-001). **(to confirm)** the exact command, and
  whether the person also wants a live `uvicorn` request as part of the
  check.

## Testing Posture

Tests are a deliverable in every Bolt, not an afterthought.

- **Methodology**: test-after
- **Methodology evidence**: Suggested default only. `org.md` sets test-after when no posture is affirmed; no team methodology has been stated yet, and the workflow's test strategy is Minimal. The sample cases in `inputs/sample-cases.csv` already act as written-down expected results, so a test-first variant (writing the sample-case tests before the matcher) is a reasonable alternative the interview should offer. (to confirm)
- **Ordering**: For each testable layer (seed-list loader, case checker, HTTP API), we implement the layer first, then write and run that layer's tests before moving to the next layer.
- Tooling: pytest, with test cases read from `inputs/sample-cases.csv`; a
  case passes when its expected `deny_code` is contained in the findings (from
  the scope document). FastAPI's test client for API-level tests.
- Volume: Minimal test strategy, meaning requirement-driven tests with a
  happy path per component, plus the sample-case suite; the construction
  guardrail adds at least two error or edge cases per test file (for example
  a non-UC fund returning "not applicable", and an invalid request rejected by
  Pydantic validation).
- Coverage: the `mvp` default adds an 80% line-coverage floor, measured with
  `pytest-cov`. **(to confirm)** that the team accepts the 80% floor and the
  extra `pytest-cov` dev dependency.
- CI before merge: the `mvp` default runs the suite in CI before merge.
  **(to confirm)** whether a GitHub Actions test workflow is part of this
  local-only run, or whether this run gates merges on a local `pytest` run
  and CI arrives with the separate deployment run.
- Test data is fictional only; no real patient data ever enters fixtures.

## Deployment

This run is local only. Nothing is deployed from this run; deploying to the
SATI dev account is a separate, later run.

- For the later run, the target is AWS `ap-southeast-1` in the SATI dev
  account: the FastAPI app wrapped with Mangum and run on AWS Lambda,
  deployed by GitHub Actions assuming an IAM role through OIDC, with no AWS
  access keys stored anywhere.
- Suggested default for that later run: deploy on merge to `main` into the
  SATI dev environment, with a smoke test (one known sample case through the
  deployed API) after each deploy, and rollback by pointing the Lambda alias
  back to the previous published version.
- There is no production environment for this tool yet (internal dev use
  only). If one is added, it gets its own manual approval gate, per the org
  default.
- **(to confirm)** that this run deploys nothing, and whether the
  deploy-on-merge-to-dev default is the right starting point for the later
  run or should be left undecided until then.

## Code Style

We follow idiomatic Python 3.12 and let the tool configuration in the repo
decide formatting and linting, not ad hoc agent suggestions.

- Naming: `snake_case` for functions, variables and modules, `PascalCase` for
  classes and Pydantic models, upper case for constants.
- Type hints on public functions; request and response bodies are Pydantic
  models.
- Dependencies are pinned in `requirements.txt` (runtime) with dev-only tools
  kept separate, for example in `requirements-dev.txt`. **(to confirm)** the
  split.
- Formatter and linter: suggested Ruff for both linting and formatting,
  configured in `pyproject.toml`, run before each merge. The org default
  names Black as the Python formatter; Ruff's formatter is Black-compatible
  and needs one less tool. **(to confirm)** Ruff alone, Black plus Ruff, or no
  enforced formatter for this afternoon.
