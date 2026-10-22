# Technical environment
1. Language: Python 3.12. Package manager: pip + requirements.txt.
2. Web framework: FastAPI (wrapped for AWS Lambda with Mangum).
3. Cloud: AWS ap-southeast-1, SATI dev account; deploy with GitHub Actions + OIDC role (no AWS keys).
4. Tests: pytest, cases from inputs/sample-cases.csv.
5. Prohibited: | Any database | not needed | load files at start-up | ; | LLM calls | must be deterministic | Python rules | ; | pandas | heavy for Lambda | openpyxl + csv | ; | real patient data | PDPA | fictional cases |
6. Security: internal dev only; validate input with Pydantic; no secrets in code; keep the repo private.
