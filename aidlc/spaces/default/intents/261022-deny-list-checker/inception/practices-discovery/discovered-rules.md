# Discovered Rules (DRAFT)

> **Status: DRAFT, pending the practices interview.** Only hard constraints a
> human has already stated are listed here. Every rule below comes from the
> team's technical environment note
> (`aidlc/spaces/default/knowledge/aidlc-shared/technical-environment.md`),
> which the team wrote; item numbers refer to that note. Suggested defaults
> from `org.md` are deliberately not listed here; they live in
> `team-practices.md` until the person affirms them.

## Mandated

- ALWAYS write the service in Python 3.12 and manage dependencies with pip and `requirements.txt`. (Technical environment note, item 1)
- ALWAYS build the HTTP API with FastAPI, wrapped with Mangum so the same app runs on AWS Lambda. (Technical environment note, item 2)
- ALWAYS target AWS `ap-southeast-1` in the SATI dev account, and deploy with GitHub Actions assuming an IAM role through OIDC. (Technical environment note, item 3)
- ALWAYS test with pytest, using the cases in `inputs/sample-cases.csv`. (Technical environment note, item 4)
- ALWAYS load rule data from files at start-up instead of a database. (Technical environment note, item 5)
- ALWAYS keep the check deterministic, implemented as Python rules. (Technical environment note, item 5)
- ALWAYS read input files with the standard `csv` module, and with `openpyxl` when Excel is needed. (Technical environment note, item 5)
- ALWAYS use fictional test cases. (Technical environment note, item 5)
- ALWAYS validate request input with Pydantic at the API boundary. (Technical environment note, item 6)
- ALWAYS keep the repository private and the service for internal dev use only. (Technical environment note, item 6)

## Forbidden

- NEVER use a database of any kind. (Technical environment note, item 5)
- NEVER call an LLM or any other model; the check must be deterministic. (Technical environment note, item 5)
- NEVER use pandas. (Technical environment note, item 5)
- NEVER use real patient data, in code, fixtures, logs or examples (PDPA). (Technical environment note, item 5)
- NEVER put secrets in code. (Technical environment note, item 6)
- NEVER store or use AWS access keys for deployment; use the OIDC role instead. (Technical environment note, item 3)
