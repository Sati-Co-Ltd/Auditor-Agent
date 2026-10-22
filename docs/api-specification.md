# API Specification: 2026 NHSO DENY-list Checker

> Version: 1.0.0 · Stack: Python 3.12 / FastAPI · Base URL: `http://localhost:8000`
> Machine-readable spec: `docs/api-spec.yaml` (OpenAPI 3.1)

---

## Overview

Three endpoints. No authentication for local development.

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/check` | Check one case for NHSO DENY risks |
| `POST` | `/check/batch` | Check multiple cases in one request |
| `GET` | `/health` | Application health + DENY-list load status |

**Core rule:** NHSO DENY rules apply to **UC / UC (NHSO)** claims only.
Non-UC funds always return `NOT_APPLICABLE` without any matching.

---

## Status Codes

| HTTP | Meaning |
|------|---------|
| `200` | Request processed successfully (risk may or may not exist — check `status` field) |
| `422` | Validation error — missing or invalid fields (FastAPI / Pydantic) |
| `503` | Service degraded — DENY list failed to load at startup (`/health` only) |

---

## Common Types

### `RiskStatus` enum

| Value | Meaning |
|-------|---------|
| `DENY_RISK` | UC-funded case with ≥ 1 ICD code matching a DENY rule |
| `NO_RISK` | UC-funded case with 0 matches |
| `NOT_APPLICABLE` | Non-UC fund; NHSO rules do not apply |

### `Finding` object

One identified DENY risk. A single input ICD code can produce **multiple** findings
(one per deny_code per matched rule row, e.g. A41.9 → CR1 and CR37 from DENY-001).

| Field | Type | Description |
|-------|------|-------------|
| `deny_code` | string | DENY code triggered (e.g. `CR1`) |
| `entry_id` | string | Rule row ID in the seed list (e.g. `DENY-001`) |
| `matched_icd` | string | The input code that matched (e.g. `A41.9`) |
| `icd_entry` | string | The rule's ICD entry — single, range, or list (e.g. `A40-A41`) |
| `condition` | string | Clinical condition name (e.g. `SEPSIS`) |
| `source` | string | Full guideline citation text |
| `source_row` | integer | Row number in source guideline (for auditable citation) |

---

## Endpoint 1 — `POST /check`

Check a single case for NHSO DENY risks.

### Processing logic

```
1. Validate input (Pydantic)
2. Normalise fund:  UC → UC (NHSO)
3. Is fund UC (NHSO)?
     No  → return { status: NOT_APPLICABLE, findings: [] }
     Yes → continue
4. For each icd_code in icd_codes:
     For each rule in DENY index:
       Does icd_code match rule.icd_entry?  (exact | range | list)
         Yes → for each deny_code in rule.deny_codes:
                 add Finding { deny_code, entry_id, matched_icd, icd_entry,
                               condition, source, source_row }
5. findings.length > 0 → status = DENY_RISK
   findings.length = 0 → status = NO_RISK
6. Return CaseCheckResponse
```

### ICD matching rules

| Entry shape | Example | Matches when |
|-------------|---------|-------------|
| Single code | `E87.2` | Input code == entry exactly |
| Range | `A40-A41` | Input code falls within the alphabetic-numeric range (e.g. A41.9 is inside A40-A41) |
| Comma list | `E86,E87.7` | Input code == any code in the list |

### Request

```
POST /check
Content-Type: application/json
```

```json
{
  "case_id":   "GT-001",
  "fund":      "UC",
  "icd_codes": ["A41.9", "N39.0"]
}
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `case_id` | string | No | Caller reference. Echoed in response. |
| `fund` | string | **Yes** | Accepted: `UC`, `UC (NHSO)`, `SSO`, `CSMBS`, `Private`. Case-insensitive. |
| `icd_codes` | string[] | **Yes** | Minimum 1 item. Each code is whitespace-trimmed. |

### Response `200`

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
    },
    {
      "deny_code":   "CR37",
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

| Field | Type | Notes |
|-------|------|-------|
| `case_id` | string \| null | Echoed from request. null if not provided. |
| `fund` | string | Normalised: UC → UC (NHSO). Non-UC returned as-is. |
| `status` | RiskStatus | `DENY_RISK` \| `NO_RISK` \| `NOT_APPLICABLE` |
| `findings` | Finding[] | Empty array when `NO_RISK` or `NOT_APPLICABLE`. |

### All three ground-truth scenarios

**GT-001 — UC, sepsis, DENY_RISK**
```json
// Request
{ "case_id": "GT-001", "fund": "UC", "icd_codes": ["A41.9", "N39.0"] }

// Response
{ "case_id": "GT-001", "fund": "UC (NHSO)", "status": "DENY_RISK",
  "findings": [{ "deny_code": "CR1", "entry_id": "DENY-001",
                 "matched_icd": "A41.9", "icd_entry": "A40-A41",
                 "condition": "SEPSIS", "source_row": 33, ... }] }
```

**GT-002 — SSO, NOT_APPLICABLE**
```json
// Request
{ "case_id": "GT-002", "fund": "SSO", "icd_codes": ["E86"] }

// Response
{ "case_id": "GT-002", "fund": "SSO", "status": "NOT_APPLICABLE", "findings": [] }
```

**GT-003 — UC, no matching rule, NO_RISK**
```json
// Request
{ "case_id": "GT-003", "fund": "UC", "icd_codes": ["K35.8"] }

// Response
{ "case_id": "GT-003", "fund": "UC (NHSO)", "status": "NO_RISK", "findings": [] }
```

### Validation error `422`

```json
{
  "detail": [
    {
      "loc":  ["body", "icd_codes"],
      "msg":  "ensure this value has at least 1 items",
      "type": "value_error.list.min_items"
    }
  ]
}
```

---

## Endpoint 2 — `POST /check/batch`

Check multiple cases in a single request. Each case is processed independently using
the same logic as `POST /check`. Use `risky_only: true` to filter the response to
only cases with `DENY_RISK` status.

### Request

```
POST /check/batch
Content-Type: application/json
```

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

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `cases` | CaseCheckRequest[] | **Yes** | Minimum 1 case. Each item follows the same rules as `POST /check`. |
| `risky_only` | boolean | No | Default `false`. When `true`, `results` contains only `DENY_RISK` cases. |

### Response `200`

```json
{
  "total":       3,
  "risky_count": 1,
  "results": [
    {
      "case_id": "GT-001",
      "fund":    "UC (NHSO)",
      "status":  "DENY_RISK",
      "findings": [
        { "deny_code": "CR1", "entry_id": "DENY-001", "matched_icd": "A41.9",
          "icd_entry": "A40-A41", "condition": "SEPSIS",
          "source": "Coding Guideline ICD-10-TM 2017 V2, DENY_CODE_AI_IP_Audit column (NHSO AI IP Pre-Audit training, 23 Jul 2569)",
          "source_row": 33 }
      ]
    },
    {
      "case_id": "GT-002",
      "fund":    "SSO",
      "status":  "NOT_APPLICABLE",
      "findings": []
    },
    {
      "case_id": "GT-003",
      "fund":    "UC (NHSO)",
      "status":  "NO_RISK",
      "findings": []
    }
  ]
}
```

| Field | Type | Notes |
|-------|------|-------|
| `total` | integer | Total input cases — never affected by `risky_only`. |
| `risky_count` | integer | Count of `DENY_RISK` results — never affected by `risky_only`. |
| `results` | CaseCheckResponse[] | Filtered when `risky_only: true`. Order preserves input order. |

### `risky_only: true` example

```json
// Request
{ "cases": [...3 cases...], "risky_only": true }

// Response — only GT-001 appears, but total = 3 and risky_count = 1
{
  "total": 3,
  "risky_count": 1,
  "results": [
    { "case_id": "GT-001", "status": "DENY_RISK", "findings": [...] }
  ]
}
```

---

## Endpoint 3 — `GET /health`

Returns application status and confirmation that the DENY list loaded correctly.
Call this before sending checks to confirm the service is ready.

### Request

```
GET /health
```

No request body or parameters.

### Response `200` — healthy

```json
{
  "status":           "ok",
  "deny_list_loaded": true,
  "rule_count":       13,
  "loaded_at":        "2026-10-22T06:45:00Z",
  "source_file":      "private-inputs/deny-list-seed-2569.csv"
}
```

### Response `503` — degraded

```json
{
  "status":           "degraded",
  "deny_list_loaded": false,
  "rule_count":       0,
  "loaded_at":        null,
  "source_file":      null
}
```

| Field | Type | Notes |
|-------|------|-------|
| `status` | string | `ok` or `degraded` |
| `deny_list_loaded` | boolean | `true` if the CSV loaded without error |
| `rule_count` | integer | Number of rules in memory. Expect 13 with the current seed list. |
| `loaded_at` | string (ISO 8601 UTC) \| null | Startup load timestamp |
| `source_file` | string \| null | Path of the loaded CSV |

---

## Validation Rules Summary

| Field | Rule |
|-------|------|
| `fund` | Required. Non-empty string. Case-insensitive. |
| `icd_codes` | Required. Array, minimum 1 item. Each item non-empty string. |
| `cases` (batch) | Required. Array, minimum 1 item. |
| `risky_only` | Optional boolean. Defaults to `false`. |
| `case_id` | Optional. Any string or null. |

All string fields are trimmed of leading/trailing whitespace before processing.

---

## Fund Normalisation

`UC` and `UC (NHSO)` are the same fund. The API normalises on input:

| Sent by caller | Stored / returned as |
|----------------|----------------------|
| `UC` | `UC (NHSO)` |
| `uc` | `UC (NHSO)` |
| `UC (NHSO)` | `UC (NHSO)` |
| `SSO` | `SSO` |
| `CSMBS` | `CSMBS` |
| `Private` | `Private` |

---

## Multiple Findings from One ICD Code

A single input ICD code can produce more than one finding:

**Case: A41.9 checked against DENY-001**

DENY-001 has `deny_codes = [CR1, CR37]` and `icd_entry = A40-A41`.
A41.9 falls inside the range A40-A41, so **two findings are produced**:

| Finding | deny_code | entry_id | matched_icd |
|---------|-----------|----------|-------------|
| 1 | CR1 | DENY-001 | A41.9 |
| 2 | CR37 | DENY-001 | A41.9 |

**Case: D61 checked against DENY-002 and DENY-012**

D61 is a single-code match in two different rule rows.
**Two findings are produced**:

| Finding | deny_code | entry_id | condition |
|---------|-----------|----------|-----------|
| 1 | CR39 | DENY-002 | APLASTIC ANAEMIA |
| 2 | CR39 | DENY-012 | APLASTIC ANAEMIA, PANCYTOPENIA, MYELODYSPLASTIC SYNDROME |

---

## Error Reference

| HTTP | When |
|------|------|
| `422` | `fund` missing or empty · `icd_codes` missing or empty array · `cases` missing or empty array |
| `503` | `GET /health` only: DENY list failed to load at startup |

All `422` responses follow FastAPI's standard Pydantic error format:
```json
{ "detail": [{ "loc": [...], "msg": "...", "type": "..." }] }
```

---

## Quick Reference

```bash
# Health check
curl http://localhost:8000/health

# Single case — UC with sepsis
curl -X POST http://localhost:8000/check \
  -H "Content-Type: application/json" \
  -d '{"fund":"UC","icd_codes":["A41.9","N39.0"]}'

# Single case — SSO (not applicable)
curl -X POST http://localhost:8000/check \
  -H "Content-Type: application/json" \
  -d '{"fund":"SSO","icd_codes":["E86"]}'

# Batch — all three ground-truth cases
curl -X POST http://localhost:8000/check/batch \
  -H "Content-Type: application/json" \
  -d '{
    "cases": [
      {"case_id":"GT-001","fund":"UC","icd_codes":["A41.9","N39.0"]},
      {"case_id":"GT-002","fund":"SSO","icd_codes":["E86"]},
      {"case_id":"GT-003","fund":"UC","icd_codes":["K35.8"]}
    ],
    "risky_only": false
  }'

# Batch — risky cases only
curl -X POST http://localhost:8000/check/batch \
  -H "Content-Type: application/json" \
  -d '{"cases":[{"fund":"UC","icd_codes":["A41.9"]},{"fund":"UC","icd_codes":["K35.8"]}],"risky_only":true}'
```

---

## Files

| File | Description |
|------|-------------|
| `docs/api-spec.yaml` | OpenAPI 3.1 machine-readable specification |
| `docs/api-specification.md` | This document — human-readable API reference |
| `docs/data-schema-and-dictionary.md` | Full data schema and data dictionary |
| `docs/high-level-architecture.md` | System architecture overview |
