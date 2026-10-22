# High-Level Architecture: 2026 NHSO DENY-list Checker

> Analysed from: `vision.md`, `intent-statement.md`, `scope-document.md`, `intent-backlog.md`,
> `constraint-register.md`, `feasibility-assessment.md`, `raid-log.md`, `inputs/sample-cases.csv`
> — 22 Oct 2026

---

## 1. Problem in One Line

Coders discover NHSO DENY risks **after** rejection. This system flags them **before** submission by
checking a case's fund and ICD-10-TM codes against the 2026 DENY list — deterministically, with zero
LLM calls.

---

## 2. Users & Their Goals

| User | Goal |
|------|------|
| Medical-record coder (primary) | Submit a case → get DENY risks → fix documentation → resubmit clean |
| Medical Statistics auditor (secondary) | Batch-scan all cases → filter for risky ones → prioritise review |
| Doctor (indirect) | Receive documentation-gap requests from coders |

---

## 3. Functional Requirements (Must)

| # | Requirement |
|---|-------------|
| FR-1 | Load the DENY seed list (`deny-list-seed-2569.csv`) at startup; each row includes source row reference |
| FR-2 | Accept one case: `fund` + list of `ICD-10-TM codes` → return findings or status |
| FR-3 | UC-only rule: NHSO DENY rules apply to **UC / UC (NHSO)** only; return `NOT_APPLICABLE` for SSO, CSMBS, private |
| FR-4 | ICD code matching: exact code, code range (A40-A41), and comma-separated list (E86,E87.7) |
| FR-5 | Every finding cites its DENY rule (DENY code + source row in the seed list) |
| FR-6 | Response carries a risk-level flag: `DENY_RISK` / `NO_RISK` / `NOT_APPLICABLE` |
| FR-7 | Batch check: accept N cases, return risk-level per case and a risky-cases-only filter |
| FR-8 | Automated tests must pass 100% on `inputs/sample-cases.csv` |
| FR-9 | Expose the single-case and batch checks through a minimal HTTP API |

### Could-have (blocked on data files)
- **FR-10** ICD-9 procedure code check against guideline criteria (`private-inputs/icd9-criteria.csv`)
- **FR-11** Lab/evidence check against guideline criteria (`private-inputs/lab-criteria.csv`)
- **FR-12** Guideline reference (GL) citation alongside the DENY citation

---

## 4. Non-Functional & Constraint Requirements

| ID | Constraint |
|----|------------|
| NF-1 | Python 3.12 · FastAPI · pytest · pip/requirements.txt |
| NF-2 | **No database** — DENY list loaded from CSV at startup |
| NF-3 | **No LLM calls** — fully deterministic rule engine |
| NF-4 | No pandas — use Python `csv` module; `openpyxl` when Excel support is added |
| NF-5 | Input validated at the boundary with Pydantic |
| NF-6 | DENY list and sample cases replaceable without code changes |
| NF-7 | **PDPA** — no real patient data; all test cases must be fictional |
| NF-8 | Deployment scope: local only for this release |

---

## 5. High-Level Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                         External Consumers                        │
│         Coder tool / Auditor dashboard / Test harness            │
└────────────────────────────┬─────────────────────────────────────┘
                             │ HTTP (JSON)
                             ▼
┌──────────────────────────────────────────────────────────────────┐
│                      FastAPI HTTP Layer                           │
│                                                                  │
│   POST /check          POST /check/batch                         │
│   ┌──────────────────┐ ┌──────────────────────────────────────┐  │
│   │  CaseCheckRequest│ │  BatchCheckRequest                   │  │
│   │  - fund: str     │ │  - cases: List[CaseCheckRequest]     │  │
│   │  - icd_codes:    │ │                                      │  │
│   │    List[str]     │ │  Returns: BatchCheckResponse         │  │
│   └────────┬─────────┘ │  - results: List[CaseResult]        │  │
│            │            │  - risky_only filter                │  │
│            └────────────┴───────────┬────────────────────────┘  │
│                                     │ Pydantic validation        │
└─────────────────────────────────────┼────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────┐
│                      DENY Check Service                           │
│                                                                  │
│  1. Fund Gate                                                    │
│     UC / UC (NHSO)? ──Yes──▶ proceed                            │
│                   └──No───▶ return NOT_APPLICABLE                │
│                                                                  │
│  2. ICD Code Matcher                                             │
│     For each input ICD-10-TM code:                              │
│       - exact match  (e.g. A41.9 == A41.9)                      │
│       - range match  (e.g. A41.9 in A40-A41)                   │
│       - list match   (e.g. E86 in "E86,E87.7")                 │
│                                                                  │
│  3. Citation Builder                                             │
│     Match → Finding {deny_code, source_row, matched_icd}        │
│                                                                  │
│  4. Risk Classifier                                              │
│     findings.len > 0 → DENY_RISK   else → NO_RISK               │
└──────────────────────────────────────────────────────────────────┘
                                      │
                          reads at startup
                                      ▼
┌──────────────────────────────────────────────────────────────────┐
│                       DENY List Loader                           │
│                                                                  │
│  Source: private-inputs/deny-list-seed-2569.csv                  │
│  Schema: icd_entry | deny_code | source_row                      │
│  Parses: exact codes, ranges (A40-A41), lists (E86,E87.7)       │
│  In-memory index built once at app startup                       │
│                                                                  │
│  Future: swap CSV → Excel (openpyxl) without code change        │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                       Test Harness (pytest)                      │
│                                                                  │
│  inputs/sample-cases.csv                                         │
│  GT-001 UC  A41.9;N39.0 → DENY_RISK, CR1 in findings ✓         │
│  GT-002 SSO E86         → NOT_APPLICABLE              ✓         │
│  GT-003 UC  K35.8       → NO_RISK                     ✓         │
│                                                                  │
│  Pass rule: expected deny_code *contained in* findings           │
└──────────────────────────────────────────────────────────────────┘
```

---

## 6. Component Breakdown

| Component | Responsibility | Technology |
|-----------|---------------|------------|
| **FastAPI HTTP Layer** | Route requests, validate input/output shapes | FastAPI + Pydantic |
| **DENY Check Service** | Orchestrate fund gate → code matching → citation → risk classification | Pure Python |
| **Fund Gate** | Map "UC" ↔ "UC (NHSO)"; reject non-UC with `NOT_APPLICABLE` | Python |
| **ICD Code Matcher** | Parse and match exact / range / list ICD entries | Python |
| **Citation Builder** | Attach `deny_code` + `source_row` to every match | Python |
| **Risk Classifier** | Produce `DENY_RISK` / `NO_RISK` / `NOT_APPLICABLE` flag | Python |
| **DENY List Loader** | Read CSV at startup, build in-memory index | Python `csv` |
| **Test Harness** | Drive ground-truth cases from CSV, assert pass/fail | pytest |

---

## 7. Data Flow (Single Case)

```
Client
  │
  │ POST /check  { fund: "UC", icd_codes: ["A41.9","N39.0"] }
  ▼
FastAPI  ──Pydantic validate──▶  DENY Check Service
                                        │
                              fund == UC? ──No──▶ {status: NOT_APPLICABLE}
                                        │Yes
                                        ▼
                              For each icd_code:
                                ICD Matcher → match deny-list rows
                                        │
                              Citation Builder → Finding list
                                        │
                              Risk Classifier
                                        │
                                  ┌─────┴──────┐
                             findings > 0    findings = 0
                                  │               │
                             DENY_RISK         NO_RISK
                                  │               │
                                  └─────┬──────────┘
                                        ▼
                          { status, findings: [{deny_code, source_row, matched_icd}] }
```

---

## 8. API Contract (Sketch)

### POST `/check` — Single case
```json
// Request
{
  "fund": "UC",
  "icd_codes": ["A41.9", "N39.0"]
}

// Response — DENY risk found
{
  "status": "DENY_RISK",
  "findings": [
    { "deny_code": "CR1", "source_row": 7, "matched_icd": "A41.9" }
  ]
}

// Response — no risk
{ "status": "NO_RISK", "findings": [] }

// Response — non-UC fund
{ "status": "NOT_APPLICABLE", "findings": [] }
```

### POST `/check/batch` — Multiple cases
```json
// Request
{
  "cases": [
    { "case_id": "GT-001", "fund": "UC",  "icd_codes": ["A41.9","N39.0"] },
    { "case_id": "GT-002", "fund": "SSO", "icd_codes": ["E86"] }
  ],
  "risky_only": false
}

// Response
{
  "results": [
    { "case_id": "GT-001", "status": "DENY_RISK",      "findings": [...] },
    { "case_id": "GT-002", "status": "NOT_APPLICABLE", "findings": [] }
  ]
}
```

---

## 9. Project Structure (Proposed)

```
Auditor-Agent/
├── app/
│   ├── main.py                  # FastAPI app entry point
│   ├── api/
│   │   └── routes.py            # /check and /check/batch endpoints
│   ├── models/
│   │   └── schemas.py           # Pydantic request/response models
│   ├── services/
│   │   ├── deny_check.py        # DENY Check Service orchestrator
│   │   ├── icd_matcher.py       # ICD code / range / list matching
│   │   └── fund_gate.py         # UC normalisation and gate
│   └── data/
│       └── deny_loader.py       # CSV loader, in-memory index builder
├── tests/
│   ├── test_deny_check.py       # Ground-truth driven from sample-cases.csv
│   ├── test_icd_matcher.py      # Unit tests: range, exact, list edge cases
│   └── test_api.py              # HTTP-level integration tests
├── inputs/
│   └── sample-cases.csv         # Fictional ground-truth cases
├── private-inputs/              # git-ignored
│   └── deny-list-seed-2569.csv  # Internal DENY seed list
└── requirements.txt
```

---

## 10. Build Order (Walking Skeleton First)

| Step | Backlog Item | Deliverable |
|------|-------------|-------------|
| 1 | IB-1 | CR1 sepsis end-to-end: loader → matcher → API → GT-001 test passes |
| 2 | IB-2 | All 13 seed-list rows loaded |
| 3 | IB-3 | ICD range/list matching complete and tested |
| 4 | IB-4 | Fund gate: UC passes, SSO/CSMBS/private return NOT_APPLICABLE |
| 5 | IB-5 | Citation (deny_code + source_row) on every finding |
| 6 | IB-6 | All 3 sample cases pass; 100% ground-truth metric met |
| 7 | IB-10 | Risk-level flag on single-case response |
| 8 | IB-7 | Batch check endpoint with risky-only filter |
| 9 | IB-9 *(Could)* | GL citation alongside DENY citation |
| 10 | IB-11 *(Could, blocked)* | ICD-9 procedure criteria check |
| 11 | IB-12 *(Could, blocked)* | Lab/evidence criteria check |

---

## 11. Key Risks to Watch

| Risk | Mitigation |
|------|------------|
| ICD range edge cases (A40-A41, E86,E87.7) | Explicit unit tests per entry shape in IB-3 |
| Fund label mismatch (UC vs UC (NHSO)) | Normalise in Fund Gate; covered by GT-002 test |
| Official 2026 list arrives in different format | Keep list as swappable data file; defer Excel to IB-8 |
| 3 ground-truth cases too thin | Accepted; Pod B extends to 15-25 cases |
| Time-box overrun | Drop Could items (IB-9, IB-11, IB-12) first |
