# Scope Definition — Questions

Already settled, so not asked again: the in-scope and out-of-scope lists, the match rule for sample cases, and the fund mapping, all from the intent statement. The 22 Oct 13:45-16:30 time box, and that nothing waits on the official list or Pod B's cases, come from the feasibility assessment and constraint register. Excel reading waits until the official file arrives (constraint register C-T4).

My working must-haves for this release: load the DENY seed list with source rows; check one case (fund plus ICD-10-TM codes) for DENY risks, with a citation per finding; the UC-only rule, giving "not applicable" for SSO, CSMBS and private; ICD code and range matching; automated tests from the sample cases; and a minimal HTTP API for the check.

## Q1. [Product owner] Checking several cases at once for auditors

The intent statement says Medical Statistics auditors (secondary users) review DENY risks across cases. The must-haves check one case per request. Does this first release also check a batch of cases in one request?

A. Won't have this time: one case per request; auditors call it per case
B. Could have: add a batch check only if the core is done early in the time box
C. Must have: batch check is part of this release
X. Other (please specify)

[Answer]: B

## Q2. [Delivery] Build order within the time box

The time box is short (13:45-16:30). Which order should the build follow?

A. Risk first: code matching and the fund rule with their tests first, then citations, then the HTTP API
B. Thin end-to-end slice first: one rule (CR1 sepsis) through load, check, API and test, then widen to all rules
X. Other (please specify)

[Answer]: B. Thin end-to-end slice first (walking skeleton): CR1 sepsis through load, check, API and test, then widen.

## Consolidated Summary Confirmation

- Must have: load the DENY seed list with source rows; check one case (fund plus ICD-10-TM codes) with a citation per finding; UC-only rule with "not applicable" for SSO, CSMBS and private; ICD code and range matching; automated tests from the sample cases; a minimal HTTP API for the check.
- Could have: a batch check of several cases in one request for auditors, only if the core is done early in the time box (Q1).
- Build order: thin end-to-end slice first (walking skeleton): CR1 sepsis through load, check, API and test, then widen to all rules (Q2).

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
