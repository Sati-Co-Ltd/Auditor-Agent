# Vision: 2026 NHSO DENY-list checker for the SATI Auditor Agent

## What and for whom
SATI's Auditor Agent reviews inpatient cases against the ICD-10-TM coding guideline and returns
findings for doctors (documentation gaps), coders (coding gaps) and NHSO DENY risks. NHSO's AI IP
pre-audit marks some diagnosis codes with DENY codes (e.g. CR1 for sepsis): claims with these codes
are checked against stricter criteria and are often rejected. Today the 2026 DENY list is not in the
Auditor's knowledge base, so coders find DENY risks only after a claim is rejected.

We are building a small DENY-list checker for SATI's medical-record auditors and coders at
Sriphat Medical Center. Given a case's fund and coded ICD-10-TM diagnoses, it returns any 2026 NHSO
DENY risks, each citing the DENY rule (code and source row) it came from, so coders can fix the
documentation before the claim is submitted.

## Users
- Medical-record coders (primary): check a case before claim submission.
- Medical Statistics auditors: review DENY risks across cases.
- Doctors (indirect): receive requests to complete documentation.

## Success
- DENY risks caught before claim submission instead of after rejection.
- Every finding traceable to its DENY rule (auditable citation).
- Correct on the sample cases agreed by SATI's coders (ground truth).
- Low cost per case: deterministic rules, no model call needed for this check.

## In scope for the first release
- Load the 2026 NHSO DENY list with the source row of every entry.
- Check a case: fund + ICD-10-TM codes -> DENY risks with citations, or "no DENY risk".
- Match ICD code ranges correctly (A41.9 is inside A40-A41).
- Apply NHSO DENY rules to UC (NHSO) claims only; "not applicable" for SSO, CSMBS and private.
- Automated tests from the sample cases; a minimal HTTP API; deployment to SATI's dev environment.

## Not in the first release
- Real patient data or the hospital information system (HIS).
- A user interface; an LLM agent; documentation-completeness rules; other funds' rulebooks.

## Open questions
- Is the official 2026 DENY list available as a file, or only as training slides?
- When one code has several DENY rules, return one finding or several?
- Should a DENY risk also show the guideline's stricter criteria text?
