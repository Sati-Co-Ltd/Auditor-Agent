# Data Schema & Data Dictionary: 2026 NHSO DENY-list Checker

> Derived from: `private-inputs/deny-list-seed-2569.csv`, `inputs/sample-cases.csv`,
> `constraint-register.md`, `scope-document.md` — 22 Oct 2026

This system has **no database**. All data lives in CSV files (loaded into memory at startup)
and in Python in-memory objects (Pydantic models) that flow through the API.
This document covers all three layers: **file schemas**, **in-memory domain models**, and
**API request/response schemas**.

---

## Table of Contents

1. [File Schemas (CSV)](#1-file-schemas-csv)
2. [In-Memory Domain Models](#2-in-memory-domain-models)
3. [API Schemas (Pydantic / JSON)](#3-api-schemas-pydantic--json)
4. [Data Dictionary](#4-data-dictionary)
5. [Fund Code Reference](#5-fund-code-reference)
6. [DENY Code Reference](#6-deny-code-reference)
7. [Data Relationships](#7-data-relationships)

---

## 1. File Schemas (CSV)

### 1.1 `private-inputs/deny-list-seed-2569.csv` — DENY Rule List

The master rule source. Loaded once at application startup into an in-memory index.
Each row is one rule entry. A single ICD code or range may appear in multiple rows
(e.g. D61 appears in rows DENY-002 and DENY-012; I25 in DENY-006 and DENY-013).

| Column | Type | Required | Example | Description |
|--------|------|----------|---------|-------------|
| `entry_id` | string | ✅ | `DENY-001` | Unique identifier for this rule row. Format: `DENY-NNN` (zero-padded 3 digits) |
| `icd10tm_codes` | string | ✅ | `A40-A41` | ICD-10-TM code entry. Three shapes: single code (`E87.2`), range (`A40-A41`), comma-separated list (`E86,E87.7`) |
| `condition` | string | ✅ | `SEPSIS` | Human-readable name of the clinical condition for this rule |
| `deny_codes` | string | ✅ | `CR1;CR37` | One or more DENY codes, semicolon-separated. Each DENY code is a rule that NHSO applies during AI IP pre-audit |
| `fund` | string | ✅ | `UC (NHSO)` | Fund this rule applies to. Currently always `UC (NHSO)` for all seed rows |
| `source` | string | ✅ | `Coding Guideline ICD-10-TM 2017 V2, DENY_CODE_AI_IP_Audit column…` | Full citation of the guideline document this rule came from |
| `source_row` | integer | ✅ | `33` | Row number in the source guideline document, for auditable citation |

**Constraints:**
- `entry_id` must be unique across all rows
- `icd10tm_codes` must not be empty
- `deny_codes` must contain at least one code
- `source_row` must be a positive integer
- File encoding: UTF-8; line ending: LF
- File is git-ignored (internal data); must not be committed to the public repo

**Current content summary:**
- 13 rows total
- 10 distinct DENY codes
- All rows are `UC (NHSO)` fund
- ICD entry shapes present: range (A40-A41, F10-F19), single code (D61, E87.2, I25, J44, L03, M72.6, R57, T79.3), comma-list (E86,E87.7)

---

### 1.2 `inputs/sample-cases.csv` — Ground-Truth Test Cases

Fictional cases agreed by SATI's clinical coders. Used by pytest as ground truth.
No real patient data; no names, IDs, or real dates.

| Column | Type | Required | Example | Description |
|--------|------|----------|---------|-------------|
| `case_id` | string | ✅ | `GT-001` | Unique case identifier. Format: `GT-NNN` |
| `fund` | string | ✅ | `UC` | Fund code for this case (as entered by coder; may differ from seed-list label) |
| `coded_diagnoses` | string | ✅ | `A41.9;N39.0` | Semicolon-separated list of ICD-10-TM diagnosis codes entered by the coder |
| `expect_deny` | string | ✅ | `Y` | Whether a DENY risk is expected: `Y` = yes, `N` = no |
| `deny_code` | string | ❌ | `CR1` | Expected DENY code in findings (only when `expect_deny = Y`). Pass rule: this value must be *contained in* the findings |
| `why` | string | ❌ | `Sepsis coded without blood culture or lactate` | Human note explaining why the expected result holds. Not used by the engine |

**Constraints:**
- `case_id` must be unique
- `expect_deny` must be `Y` or `N`
- When `expect_deny = Y`, `deny_code` must be non-empty
- All data must be fictional (PDPA — constraint C-R1)

**Current rows:**

| case_id | fund | coded_diagnoses | expect_deny | deny_code | why |
|---------|------|-----------------|-------------|-----------|-----|
| GT-001 | UC | A41.9;N39.0 | Y | CR1 | Sepsis coded without blood culture or lactate |
| GT-002 | SSO | E86 | N | — | NHSO DENY rules do not apply to SSO |
| GT-003 | UC | K35.8 | N | — | No DENY rule for this code |

---

## 2. In-Memory Domain Models

These are the Python dataclasses / Pydantic models that represent data after loading.
They live only in process memory — nothing is persisted.

### 2.1 `DenyRule` — A parsed rule entry from the DENY list

```python
@dataclass
class DenyRule:
    entry_id:    str         # e.g. "DENY-001"
    icd_entry:   str         # raw string as-is from CSV e.g. "A40-A41"
    entry_type:  EntryType   # RANGE | SINGLE | LIST  (derived at load time)
    condition:   str         # e.g. "SEPSIS"
    deny_codes:  list[str]   # e.g. ["CR1", "CR37"]
    fund:        str         # normalised to "UC (NHSO)"
    source:      str         # full citation text
    source_row:  int         # guideline row number
```

| Field | Type | Description |
|-------|------|-------------|
| `entry_id` | `str` | Primary key. Carried into findings for traceability |
| `icd_entry` | `str` | The raw ICD entry string from the CSV (preserved for display) |
| `entry_type` | `EntryType` | Enum: `RANGE` (e.g. A40-A41) · `SINGLE` (e.g. E87.2) · `LIST` (e.g. E86,E87.7). Derived by the loader at parse time |
| `condition` | `str` | Clinical condition name for human display |
| `deny_codes` | `list[str]` | Parsed list from semicolon-separated CSV value |
| `fund` | `str` | Always `UC (NHSO)` in current seed data |
| `source` | `str` | Full citation string |
| `source_row` | `int` | Row number in source guideline |

---

### 2.2 `DenyIndex` — In-memory lookup index (built at startup)

```python
@dataclass
class DenyIndex:
    rules: list[DenyRule]        # all loaded rules, in file order
    loaded_at: datetime          # UTC timestamp of load
    source_file: str             # path to the CSV that was loaded
```

| Field | Type | Description |
|-------|------|-------------|
| `rules` | `list[DenyRule]` | All rules available for matching. Iterated for every case check |
| `loaded_at` | `datetime` | When the index was built (UTC). Useful for health-check endpoints |
| `source_file` | `str` | Absolute path of the CSV loaded. Supports future hot-swap |

---

### 2.3 `Finding` — One DENY risk result for a matched code

```python
@dataclass
class Finding:
    deny_code:    str    # e.g. "CR1"
    entry_id:     str    # e.g. "DENY-001"  (the rule row that triggered this)
    matched_icd:  str    # e.g. "A41.9"     (the input code that matched)
    icd_entry:    str    # e.g. "A40-A41"   (the rule's ICD entry string)
    condition:    str    # e.g. "SEPSIS"
    source:       str    # full guideline citation
    source_row:   int    # guideline row number
```

| Field | Type | Description |
|-------|------|-------------|
| `deny_code` | `str` | The DENY code triggered (e.g. CR1). One Finding per deny_code per matched rule row |
| `entry_id` | `str` | Which rule row produced this finding. Enables audit trail back to the seed list |
| `matched_icd` | `str` | The coder's input code that triggered the match |
| `icd_entry` | `str` | The rule's ICD entry (range/single/list) that the input code matched against |
| `condition` | `str` | Clinical condition name for coder-facing display |
| `source` | `str` | Full guideline citation text |
| `source_row` | `int` | Guideline row number for citation |

---

### 2.4 `RiskStatus` — Enum for case risk classification

```python
class RiskStatus(str, Enum):
    DENY_RISK      = "DENY_RISK"       # UC fund + ≥1 DENY rule matched
    NO_RISK        = "NO_RISK"         # UC fund + 0 DENY rules matched
    NOT_APPLICABLE = "NOT_APPLICABLE"  # Non-UC fund; NHSO rules do not apply
```

---

### 2.5 `CaseResult` — Full result for one checked case

```python
@dataclass
class CaseResult:
    case_id:   str | None      # echoed from request if provided
    fund:      str             # normalised fund value
    status:    RiskStatus      # DENY_RISK | NO_RISK | NOT_APPLICABLE
    findings:  list[Finding]   # empty list when status != DENY_RISK
```

---

## 3. API Schemas (Pydantic / JSON)

### 3.1 `POST /check` — Single-case check

#### Request: `CaseCheckRequest`

```json
{
  "case_id":   "GT-001",
  "fund":      "UC",
  "icd_codes": ["A41.9", "N39.0"]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `case_id` | `string` | ❌ | Optional caller-supplied identifier. Echoed back in the response |
| `fund` | `string` | ✅ | Fund code. Accepted values: `UC`, `UC (NHSO)`, `SSO`, `CSMBS`, `Private` (case-insensitive) |
| `icd_codes` | `array[string]` | ✅ | One or more ICD-10-TM diagnosis codes. Must not be empty |

**Validation rules:**
- `fund` must not be empty
- `icd_codes` must contain at least one non-empty string
- Each ICD code is trimmed of whitespace before processing

---

#### Response: `CaseCheckResponse`

```json
{
  "case_id": "GT-001",
  "fund":    "UC (NHSO)",
  "status":  "DENY_RISK",
  "findings": [
    {
      "deny_code":   "CR1",
      "entry_id":    "DENY-001",
      "matched_icd": "A41.9",
      "icd_entry":   "A40-A41",
      "condition":   "SEPSIS",
      "source":      "Coding Guideline ICD-10-TM 2017 V2, DENY_CODE_AI_IP_Audit column (NHSO AI IP Pre-Audit training, 23 Jul 2569)",
      "source_row":  33
    }
  ]
}
```

| Field | Type | Always present | Description |
|-------|------|----------------|-------------|
| `case_id` | `string \| null` | ✅ | Echoed from request |
| `fund` | `string` | ✅ | Normalised fund value (always `UC (NHSO)` or the original non-UC value) |
| `status` | `string` | ✅ | `DENY_RISK` · `NO_RISK` · `NOT_APPLICABLE` |
| `findings` | `array` | ✅ | List of Finding objects. Empty array when status is `NO_RISK` or `NOT_APPLICABLE` |
| `findings[].deny_code` | `string` | ✅ | DENY code triggered (e.g. `CR1`) |
| `findings[].entry_id` | `string` | ✅ | Rule row ID (e.g. `DENY-001`) |
| `findings[].matched_icd` | `string` | ✅ | The input ICD code that matched |
| `findings[].icd_entry` | `string` | ✅ | The rule's ICD entry (range/single/list) |
| `findings[].condition` | `string` | ✅ | Clinical condition name |
| `findings[].source` | `string` | ✅ | Full guideline citation |
| `findings[].source_row` | `integer` | ✅ | Row number in the source guideline |

---

### 3.2 `POST /check/batch` — Batch check

#### Request: `BatchCheckRequest`

```json
{
  "cases": [
    { "case_id": "GT-001", "fund": "UC",  "icd_codes": ["A41.9", "N39.0"] },
    { "case_id": "GT-002", "fund": "SSO", "icd_codes": ["E86"] },
    { "case_id": "GT-003", "fund": "UC",  "icd_codes": ["K35.8"] }
  ],
  "risky_only": false
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cases` | `array[CaseCheckRequest]` | ✅ | One or more case requests. Must not be empty |
| `risky_only` | `boolean` | ❌ | Default `false`. When `true`, response includes only cases with `status = DENY_RISK` |

---

#### Response: `BatchCheckResponse`

```json
{
  "total":      3,
  "risky_count": 1,
  "results": [
    { "case_id": "GT-001", "fund": "UC (NHSO)", "status": "DENY_RISK",      "findings": [...] },
    { "case_id": "GT-002", "fund": "SSO",       "status": "NOT_APPLICABLE", "findings": [] },
    { "case_id": "GT-003", "fund": "UC (NHSO)", "status": "NO_RISK",        "findings": [] }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `total` | `integer` | Total number of cases in the request (before risky_only filter) |
| `risky_count` | `integer` | Number of cases with `status = DENY_RISK` |
| `results` | `array[CaseCheckResponse]` | Results per case. Filtered to risky-only if requested |

---

## 4. Data Dictionary

Complete glossary of every term, field, and code used in this system.

| Term | Definition |
|------|------------|
| **DENY list** | The list of ICD-10-TM codes and their associated DENY rules, taken from the `DENY_CODE_AI_IP_Audit` column of the ICD-10-TM coding guideline. NHSO's AI IP pre-audit checks claims against this list |
| **DENY code** | A rule identifier (e.g. `CR1`, `CR37`) used by NHSO's AI IP pre-audit system. A DENY code marks a claim for additional scrutiny under stricter clinical criteria |
| **DENY risk** | The situation where a case's ICD-10-TM codes match one or more entries in the DENY list, indicating the claim may be rejected by NHSO |
| **ICD-10-TM** | International Classification of Diseases, 10th Revision, Thai Modification. The coding system used by Thai hospitals for diagnosis reporting |
| **ICD entry** | A value in the `icd10tm_codes` column of the DENY list. Can be a single code (`E87.2`), a range (`A40-A41`), or a comma-separated list (`E86,E87.7`) |
| **entry_id** | Unique identifier for a row in the DENY seed list (format: `DENY-NNN`). Used to trace findings back to their source rule row |
| **entry_type** | Classification of an ICD entry: `RANGE` (hyphen-separated start and end code), `SINGLE` (one code), or `LIST` (comma-separated codes) |
| **fund** | The healthcare reimbursement fund covering a case. Determines which rule sets apply. See Fund Code Reference |
| **UC / UC (NHSO)** | Universal Coverage scheme administered by NHSO. The only fund to which NHSO DENY rules apply in this release. `UC` and `UC (NHSO)` are treated as the same fund |
| **SSO** | Social Security Office fund. NHSO DENY rules do not apply; returns `NOT_APPLICABLE` |
| **CSMBS** | Civil Servant Medical Benefit Scheme. NHSO DENY rules do not apply; returns `NOT_APPLICABLE` |
| **coded_diagnoses** | The ICD-10-TM codes a medical-record coder has assigned to a case, semicolon-separated |
| **finding** | One identified DENY risk: a combination of a matched input ICD code, the DENY code it triggered, and the citation (entry_id, source, source_row) |
| **citation** | The auditable reference attached to a finding: `entry_id`, `source` (guideline document), and `source_row` (row number in that document) |
| **source_row** | The row number in the ICD-10-TM coding guideline document where this DENY rule appears. Required for every finding (constraint C-R3) |
| **DENY_RISK** | Status: the case is UC-funded and at least one input ICD code matched a DENY rule |
| **NO_RISK** | Status: the case is UC-funded but no input ICD codes matched any DENY rule |
| **NOT_APPLICABLE** | Status: the case's fund is not UC/UC(NHSO); NHSO DENY rules do not apply |
| **ground truth** | The set of fictional test cases in `inputs/sample-cases.csv` that define the expected behaviour of the checker, agreed by SATI's clinical coders |
| **seed list** | The initial 13-row DENY list extracted from the ICD-10-TM coding guideline for this release, stored in `private-inputs/deny-list-seed-2569.csv`. Replaced by SATI's official 2026 list when it arrives |
| **walking skeleton** | The first end-to-end build: CR1 sepsis (GT-001) through loader → matcher → API → passing test. Proves the whole path before widening to all rules |
| **risky_only** | Batch API filter flag. When `true`, the response includes only cases with `DENY_RISK` status |
| **NHSO** | National Health Security Office. The Thai government agency that administers the UC scheme and runs the AI IP pre-audit |
| **AI IP Pre-Audit** | NHSO's automated inpatient pre-audit system. Checks claims before payment; uses DENY codes to flag high-risk diagnoses for stricter review |
| **PDPA** | Personal Data Protection Act (Thailand). Prohibits use of real patient data; all test cases must be fictional |

---

## 5. Fund Code Reference

| Input value | Normalised to | Rule applies? | Status returned |
|-------------|---------------|---------------|-----------------|
| `UC` | `UC (NHSO)` | ✅ Yes | `DENY_RISK` or `NO_RISK` |
| `UC (NHSO)` | `UC (NHSO)` | ✅ Yes | `DENY_RISK` or `NO_RISK` |
| `SSO` | `SSO` | ❌ No | `NOT_APPLICABLE` |
| `CSMBS` | `CSMBS` | ❌ No | `NOT_APPLICABLE` |
| `Private` | `Private` | ❌ No | `NOT_APPLICABLE` |

Matching is **case-insensitive** and trims leading/trailing whitespace.

---

## 6. DENY Code Reference

All 10 distinct DENY codes present in the seed list (13 rows, some codes shared across rows):

| DENY Code | Appears in entry_id(s) | Condition(s) |
|-----------|------------------------|--------------|
| CR1 | DENY-001 | SEPSIS |
| CR37 | DENY-001, DENY-010 | SEPSIS · SHOCK |
| CR39 | DENY-002, DENY-012 | APLASTIC ANAEMIA · APLASTIC ANAEMIA / PANCYTOPENIA / MDS |
| CR8 | DENY-003 | VOLUME DEPLETION AND FLUID OVERLOAD |
| CR5 | DENY-004 | ACIDOSIS |
| CR44_1 | DENY-005 | MENTAL AND BEHAVIOURAL DISORDERS DUE TO PSYCHOACTIVE SUBSTANCE USE |
| CR45 | DENY-006, DENY-013 | CHRONIC ISCHAEMIC HEART DISEASE |
| CR58 | DENY-006, DENY-013 | CHRONIC ISCHAEMIC HEART DISEASE |
| CR13_1 | DENY-007 | CHRONIC OBSTRUCTIVE PULMONARY DISEASE |
| CR19 | DENY-008, DENY-009, DENY-011 | CELLULITIS · NECROTIZING FASCIITIS · WOUND INFECTION |

---

## 7. Data Relationships

```
deny-list-seed-2569.csv
  │
  │ loaded at startup
  ▼
DenyIndex
  └── rules: list[DenyRule]         ← one DenyRule per CSV row
                   │
                   │ matched against
                   ▼
         CaseCheckRequest.icd_codes
                   │
                   │ produces (one per deny_code per matched rule row)
                   ▼
               Finding
                   │
                   │ aggregated into
                   ▼
           CaseCheckResponse
               ├── status: RiskStatus
               └── findings: list[Finding]
                        │
                        │ (batch) collected into
                        ▼
               BatchCheckResponse.results


sample-cases.csv
  │
  │ read by pytest only
  ▼
TestCase
  ├── fund + icd_codes  ──▶  CaseCheckRequest (test input)
  └── expect_deny + deny_code  ──▶  assertion (deny_code in findings)
```

**Key multiplicity rules:**
- One CSV row → one `DenyRule`
- One `DenyRule` may have **multiple** `deny_codes` (e.g. DENY-001 has CR1 and CR37)
- One `Finding` represents **one** `deny_code` from **one** matched rule row
- One case check may produce **multiple** `Finding` objects (one per deny_code per matched rule row)
- One ICD input code may match multiple rule rows (e.g. D61 matches both DENY-002 and DENY-012)
