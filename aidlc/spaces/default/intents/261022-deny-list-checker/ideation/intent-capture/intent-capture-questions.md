# Intent Capture & Framing — Questions

## Sources

- [desc] Initial description: "Read ./vision.md and build what it describes. Test cases are ./inputs/sample-cases.csv. This is a new project."
- [scope] Workflow-selected scope: `mvp`.

## Q1. [Product owner] Problem and primary user

The vision (`vision.md`) says: today the 2026 NHSO DENY list is not in the SATI Auditor Agent's knowledge base, so medical-record coders at Sriphat Medical Center only find DENY risks after NHSO rejects a claim. Is this the problem we are solving, and who is it mainly for?

A. Yes: coders find DENY risks only after rejection; the primary users are medical-record coders checking a case before claim submission, with Medical Statistics auditors as secondary users
B. Yes, but the primary users are the Medical Statistics auditors reviewing DENY risks across cases
C. The problem is mainly lost revenue from rejected claims; users as in A
D. Not yet defined
X. Other (please specify)

[Answer]: A

## Q2. [Product owner] What success looks like

How will we know the first release works? (select all that apply)

A. 100% of the sample cases in `inputs/sample-cases.csv` (agreed by SATI's coders as ground truth) give the expected result: DENY risk Y/N and the expected DENY code
B. Every DENY finding cites its DENY rule (DENY code and source row in the DENY list)
C. No model call per case: the check is deterministic rules only
D. A count of DENY risks caught before submission, measured after go-live (no target yet)
E. Not yet defined
X. Other (please specify)

[Answer]: A, B, C, D

## Q3. [Product owner] Where the 2026 DENY list comes from

The checker needs the 2026 NHSO DENY list, but only `inputs/sample-cases.csv` (3 cases) is in the project; there is no DENY list file yet. The vision lists this as an open question. What will the first release use?

A. An official 2026 DENY list file will be provided (CSV or Excel); please say where
B. Only training slides exist; we will transcribe them by hand into a CSV, giving each entry a source row
C. Start with a small seed list built from what we know (for example CR1 for sepsis, ICD-10-TM A40-A41) in a CSV with source rows, and swap in the full list when it arrives
D. Not yet defined
X. Other (please specify)

[Answer]: C. Seed list from the ICD-10-TM coding guideline's DENY_CODE_AI_IP_Audit column (NHSO AI IP Pre-Audit training, 23 Jul 2569): 11 conditions / 12 DENY codes, each with its guideline row as the source. Replace with SATI's official 2026 list when it arrives.

## Q4. [Product owner] Who decides and who needs to hear

Who decides scope and priority for this checker, and who agrees the ground-truth cases?

A. SATI's coders agree the ground-truth cases; the SATI project owner decides scope and priority
B. The Medical Statistics team decides both
C. Not identified yet
X. Other (please specify)

[Answer]: A. K'Pop (SATI CEO and product owner) decides scope and priority; SATI's clinical coders agree the ground-truth cases.

## Q5. [Product owner] What this first version leaves out

The workflow scope is already chosen as `mvp`. The vision says the first release does NOT include: real patient data or the hospital information system (HIS); a user interface; an LLM agent; documentation-completeness rules; other funds' rulebooks. Is that the full list of what this first version leaves out?

A. Yes, exactly that list
B. That list, and also leave out deployment to SATI's dev environment (run locally only)
C. That list, and also leave out the HTTP API (library and tests only)
D. None — no exclusions beyond the vision
X. Other (please specify)

[Answer]: B. This mvp run is local only (the mvp profile has no Operation phase); deployment to SATI dev is done in a separate express run.

## Q6. [Product owner] Where the seed list's 12 DENY rows come from

Your answer to Q3 says the seed list comes from the guideline's DENY_CODE_AI_IP_Audit column (11 conditions, 12 DENY codes). That guideline is not in the project yet: only `inputs/sample-cases.csv` is there, and the only DENY rule we know for sure is CR1 for sepsis (A41.9 in sample case GT-001). Without the other rows, the checker cannot cite a real source row for them. How will those rows reach the project?

A. I will add the guideline (or an extract of its DENY rows: condition, ICD-10-TM codes or range, DENY code, row number) to `inputs/` before Construction starts
B. I will paste the 12 rows here in chat
C. Build the first release with CR1 only, and add the other 11 rows when the extract arrives
D. Not yet defined
X. Other (please specify)

[Answer]: A. Done: added inputs/deny-list-seed-2569.csv (13 rows: 11 conditions, 12 DENY codes incl. CR44_1 and CR13_1) extracted from the guideline, each with its guideline row number as source_row.

## Consolidated Summary Confirmation

- Problem: coders find 2026 NHSO DENY risks only after a claim is rejected; primary users are medical-record coders checking a case before claim submission, Medical Statistics auditors secondary (Q1).
- Success: 100% of sample cases match the expected result (DENY Y/N and DENY code); every finding cites its DENY rule (code and source row); deterministic rules, no model call; a count of DENY risks caught before submission, measured after go-live, no target yet (Q2).
- DENY list: a seed list from the ICD-10-TM coding guideline's DENY_CODE_AI_IP_Audit column (NHSO AI IP Pre-Audit training, 23 Jul 2569), each entry with its guideline row as source, replaced by SATI's official 2026 list when it arrives (Q3); already added as `inputs/deny-list-seed-2569.csv`, 13 rows (Q6).
- Decision-makers: K'Pop (SATI CEO and product owner) decides scope and priority; SATI's clinical coders agree the ground-truth cases (Q4).
- Left out of this first version: real patient data and HIS, a user interface, an LLM agent, documentation-completeness rules, other funds' rulebooks, and deployment to SATI dev (this run is local only; deployment is a separate run) (Q5).

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct

## Assumption Confirmation

The intent statement and stakeholder map carry these open items (the second was added at the approval gate, where the person asked to record it as open; see Q9):

- Whether the checker returns one finding or several when one ICD-10-TM code maps to several DENY codes or seed-list rows is not yet decided; Requirements Analysis decides it.
- Whether a DENY risk should also show the guideline's stricter criteria text is an open question.
- No communication requirement or reporting cadence has been stated for the product owner or the clinical coders.
- How much say the primary, secondary and indirect users have beyond using the checker has not been stated.

A. Accept assumptions
B. Convert to follow-up questions

[Answer]: A. Accept assumptions

## Q7. [Product owner] How many DENY codes the seed list holds

The review found 10 distinct DENY codes in `inputs/deny-list-seed-2569.csv`, not the 12 in your Q3 answer. Are codes missing, or is the count 10?

A. No codes are missing: the seed list holds 10 distinct DENY codes in 13 rows (D61 and I25 appear twice)
B. Two codes are missing from the extract; I will add them
X. Other (please specify)

[Answer]: A. no codes are missing; correct it to 10 distinct DENY codes in 13 rows (D61 and I25 appear twice).

## Q8. [Product owner] How a sample case's expected DENY code is matched

Sepsis (A40-A41) carries CR1 and CR37, but sample case GT-001 expects CR1. What counts as a match for the 100% sample-case metric?

A. The expected code is contained in the findings; GT-001 stays CR1-only for now and Requirements Analysis decides one-vs-several findings
B. The findings must equal the expected codes exactly
X. Other (please specify)

[Answer]: A. the match rule is 'the expected code is contained in the findings'; GT-001 stays CR1-only for now and Requirements Analysis decides one-vs-several findings.

## Q9. [Product owner] What is in this first version, and who else is affected

The statement listed only exclusions. What is in scope, what stays open, and are doctors stakeholders?

A. In scope: the check, citations, the UC-only rule, tests, the HTTP API; the stricter-criteria-text question stays open; doctors are indirect users
X. Other (please specify)

[Answer]: A. add the in-scope line (check, citations, UC-only rule, tests, HTTP API), record the stricter-criteria-text question as open, add doctors as indirect users.

## Q10. [Product owner] Fund names, non-UC result, and ground-truth size

The sample cases say "UC" while the seed list says "UC (NHSO)", no metric covers non-UC funds, and there are only 3 sample cases. How should these be handled?

A. UC = UC (NHSO), noted for Requirements; add the metric "non-UC funds return not applicable"; the 3-case ground truth is thin and Pod B will extend it to 15-25 cases
X. Other (please specify)

[Answer]: A. note the fund-name mapping (UC = UC (NHSO)) for Requirements, add the metric 'non-UC funds return not applicable', and flag the 3-case ground truth as thin: Pod B will extend it to 15-25 cases.
