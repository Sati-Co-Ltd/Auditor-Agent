# Practices Discovery — Questions

These set how this team works on this project. The suggested answers (option A) come from the framework's defaults and the team's technical environment note; the draft is at `team-practices.md` in this folder.

## Q1. [Release engineer] Branching and merging

How should finished work land on the main branch?

A. Trunk-based on `main`: each piece of work is squash-merged into `main` as one commit after it is approved; a local merge is enough, no GitHub pull request needed in this run
B. Trunk-based on `main`, but every merge goes through a GitHub pull request
C. Commit straight to `main`, no separate branches
X. Other (please specify)

[Answer]:

## Q2. [Release engineer] Build a thin end-to-end slice first?

A walking skeleton is a minimal version that runs the whole way through, built first to prove the pieces connect before the real features go in. You already chose this at scope definition: CR1 sepsis through load, check, API and test. How should we prove the slice works?

A. Yes: the proof is `pytest` passing, including an API-level test that sends sample case GT-001 and gets CR1 back
B. Yes: the proof is `pytest` plus a live request to the running local server for GT-001
C. No slice first; build the full rule set directly
X. Other (please specify)

[Answer]:

## Q3. [Quality engineer] When tests are written, and how much

Which testing approach should this run follow? (select all that apply)

A. Build each part (list loading, the check, the API), then write and run its tests
B. Write the sample-case tests first, then build until they pass
C. Keep an 80% line-coverage floor, measured with `pytest-cov`
D. A GitHub Actions workflow runs the tests on every push in this run
E. No CI in this run: a local `pytest` run is the merge gate until the deployment run
X. Other (please specify)

[Answer]:

## Q4. [Release engineer] Deployment

This run is local only. What should we record for deployment?

A. Nothing is deployed in this run; the separate deployment run decides how to deploy to SATI dev
B. Nothing deployed now; record the starting plan for the later run: deploy on merge to SATI dev, a smoke test after each deploy, roll back by pointing the Lambda alias at the previous version
X. Other (please specify)

[Answer]:

## Q5. [Developer] Code style tools

How should Python code style be checked?

A. Ruff for both linting and formatting, with dev tools in a separate `requirements-dev.txt`
B. Black for formatting plus Ruff for linting, dev tools in `requirements-dev.txt`
C. No enforced formatter or linter this afternoon; idiomatic Python (snake_case) only
X. Other (please specify)

[Answer]:
