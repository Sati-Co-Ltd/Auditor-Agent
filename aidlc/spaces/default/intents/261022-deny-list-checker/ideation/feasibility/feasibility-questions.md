# Feasibility & Constraints — Questions

Already known, so not asked again: there are no integrations in this first version (no HIS, no real patient data, local run only), from the intent statement. The tech stack, cloud account, security baseline and prohibited items are set in the team's technical environment note (`aidlc/spaces/default/knowledge/aidlc-shared/technical-environment.md`).

## Q1. [Delivery] Timeline and effort limits

Is there a date or effort budget this first local release must fit? It decides how much we build beyond the core check.

A. A fixed date (please say when)
B. A time box (for example within this workshop or session); keep to the core check, tests and API
C. No fixed timeline or budget
D. Not yet defined
X. Other (please specify)

[Answer]: B. Time-boxed to the 22 Oct co-build afternoon (build window 13:45-16:30); core check, tests and API only.

## Q2. [Product owner] What waits on outside inputs

Two inputs come from outside this run: SATI's official 2026 DENY list (replacing the seed list) and Pod B's extension of the ground truth to 15-25 cases. Does this first release wait for either of them?

A. No: ship on the seed list and the 3 sample cases; both arrive later and are loaded without code changes
B. Wait for Pod B's 15-25 cases before calling the release done
C. Wait for the official 2026 list before calling the release done
D. Not yet defined
X. Other (please specify)

[Answer]: A

## Q3. [Product owner] Format of the official 2026 list when it arrives

The seed list is a CSV. If the official 2026 list arrives as an Excel file, the checker must read Excel too. What format do you expect?

A. CSV with the same columns as `inputs/deny-list-seed-2569.csv`
B. Excel (.xlsx); support both CSV and Excel
C. Not known yet; support CSV now and add Excel when the file arrives
X. Other (please specify)

[Answer]: C

## Q4. [Compliance] Personal data in the test cases

The team's technical environment note bans real patient data (PDPA) and asks for fictional cases. Are the 3 sample cases, and Pod B's future cases, fictional with no patient identifiers?

A. Yes: all test cases are fictional, with no names, IDs or dates of real patients
B. They are de-identified real cases; treat them as sensitive
C. Not yet confirmed
X. Other (please specify)

[Answer]: A

## Consolidated Summary Confirmation

- Timeline: time-boxed to the 22 Oct co-build afternoon (build window 13:45-16:30); core check, tests and API only (Q1).
- Outside inputs: the release does not wait; it ships on the seed list and the 3 sample cases, and the official 2026 list and Pod B's 15-25 cases are loaded later without code changes (Q2).
- Official list format: not known yet; support CSV now and add Excel when the file arrives (Q3).
- Personal data: all test cases are fictional, with no names, IDs or dates of real patients (Q4).

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
