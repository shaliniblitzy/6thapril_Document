# GET /runs/metering — Endpoint Specification

> **Detailed specification and verification guide for the GET /runs/metering API endpoint and its `percent_complete` field**

This document provides the detailed specification for the `GET /runs/metering` endpoint. The endpoint returns metering data for code generation project runs, including the `percent_complete` (snake_case) / `percentComplete` (camelCase) progress-tracking field that represents the completion percentage of each run.

This guide covers the URL pattern, trigger conditions, request parameters, expected response schema, validation scenarios, sample responses, and edge cases for this specific endpoint. It is part of the API verification documentation suite for validating the `percent_complete` field across all code generation project run APIs.

**Target audience:** QA engineers, front-end developers, back-end developers, and technical testers.

---

## Endpoint Details

| Property | Value |
|----------|-------|
| **HTTP Method** | `GET` |
| **URL Pattern** | `/runs/metering?projectId=xxx` |
| **Authentication** | Required (application session) |
| **Content Type** | `application/json` |

> **Note:** The URL pattern `/runs/metering?projectId=xxx` is the exact pattern to look for in the browser DevTools Network tab when verifying this endpoint.

---

## When This API Appears

This API is triggered under the following conditions:

- **Viewing run history:** This API is called when the user navigates to the project's run history page or metering dashboard. The application fetches metering data to display a list of past and current code generation runs.

- **Fetching metering data for multiple runs:** This API is called when the application loads metering data for a list of code generation runs associated with a project. This may occur during page load, data refresh, or pagination of the run list.

> **How to find this request:** Look for this request in the browser DevTools Network tab when performing the above actions. Filter by **XHR/Fetch** and search for `metering` or `runs`. For detailed DevTools instructions, see the [DevTools API Inspection Guide](../../guides/devtools-api-inspection.md).

---

## Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `projectId` | String | Yes | The unique identifier of the project whose run metering data is being requested |

The `projectId` parameter is passed as a **query string parameter** in the URL.

**Example request:**

```
GET /runs/metering?projectId=abc-123-def-456
```

---

## Expected Response Schema

The response contains metering data for **multiple runs** associated with the specified project. The `percent_complete` field appears at the **run level**, within each run object in the `runs` array of the response.

### Response Structure

```json
{
  "projectId": "xxx",
  "runs": [
    {
      "runId": "string",
      "status": "string",
      "percent_complete": "number | null",
      "startedAt": "string (ISO 8601 timestamp)",
      "completedAt": "string (ISO 8601 timestamp) | null"
    }
  ]
}
```

**Key points:**
- The response contains a `runs` array — each element represents a single code generation run
- The `percent_complete` field is present in **each** run object within the array
- When multiple runs exist, **every** run object must include the `percent_complete` field

### `percent_complete` Field

| Property | Specification |
|----------|--------------|
| **Field Name (snake_case)** | `percent_complete` |
| **Field Name (camelCase)** | `percentComplete` |
| **Data Type** | Numeric (`number`) — NOT string |
| **Valid Range** | `0.0` to `100.0` inclusive |
| **Null Allowed** | Yes — `null` when no data available |
| **Location in Response** | Inside each run object in the `runs` array |

> **Data Type Clarification:** The value `"50"` (a string) is **INVALID**, while `50` or `50.0` (a number) is **VALID**. The field must always contain a numeric type — never a string representation of a number.

> **Naming Convention Note:** Both field name variants (`percent_complete` and `percentComplete`) must be checked. The user requirements explicitly identify field name inconsistency between `percent_complete` and `percentComplete` as a potential issue to verify. If this endpoint uses a different naming convention than the other two endpoints (`GET /runs/metering/current` and `GET /project`), this is a ⚠️ **naming inconsistency** that should be flagged and investigated.

---

## Validation Scenarios

The following validation matrix defines the expected outcomes for the `percent_complete` / `percentComplete` field in this endpoint's responses:

| Scenario        | Expected               |
|-----------------|------------------------|
| Completed run   | Value between 0–100    |
| In-progress run | Likely < 100           |
| No data         | null                   |
| Field missing   | ❌ Bug                 |

### Detailed Test Scenarios

The following expanded table provides specific test scenarios for the `GET /runs/metering` endpoint:

| Scenario | Pre-condition | Action | Expected `percent_complete` | Result |
|----------|--------------|--------|----------------------------|--------|
| Completed run | Project has completed code generation runs | View run history | Numeric value `0.0`–`100.0` (typically `100.0`) | ✅ Pass if present and in range |
| In-progress run | Code generation run actively running | View run history | Numeric value `< 100.0` | ✅ Pass if present and < 100 |
| No data | Project with no metering data | View run history | `null` | ✅ Pass if field present and value is `null` |
| Field missing | Any project state | View run history | Field absent from response | ❌ **Bug** |
| Multiple runs | Project with mixed completed and in-progress runs | View run history | Each run object has valid `percent_complete` | ✅ Pass if all run objects have the field |

> **Important for multiple runs:** When the response contains an array of run objects, verify `percent_complete` in **every** run object — not just the first one. Each run in the array must independently satisfy the validation criteria.

---

## Sample Responses

The following sample JSON responses illustrate valid and invalid scenarios for the `GET /runs/metering` endpoint. Use these as reference when inspecting actual API responses in the browser DevTools Network tab.

### Sample 1: ✅ Valid — Completed run

A completed code generation run with `percent_complete` at `100.0`:

```json
{
  "projectId": "abc-123-def-456",
  "runs": [
    {
      "runId": "run-001",
      "status": "completed",
      "percent_complete": 100.0,
      "startedAt": "2024-01-15T10:00:00Z",
      "completedAt": "2024-01-15T10:15:00Z"
    }
  ]
}
```

### Sample 2: ✅ Valid — Null value (no data)

A pending run where progress data is not yet available:

```json
{
  "projectId": "abc-123-def-456",
  "runs": [
    {
      "runId": "run-002",
      "status": "pending",
      "percent_complete": null,
      "startedAt": "2024-01-15T11:00:00Z",
      "completedAt": null
    }
  ]
}
```

### Sample 3: ✅ Valid — In-progress run

An actively running code generation run at 45.5% completion:

```json
{
  "projectId": "abc-123-def-456",
  "runs": [
    {
      "runId": "run-003",
      "status": "running",
      "percent_complete": 45.5,
      "startedAt": "2024-01-15T12:00:00Z",
      "completedAt": null
    }
  ]
}
```

### Sample 4: ❌ Invalid — Field missing (bug indicator)

The `percent_complete` field is entirely absent from the run object — **this is a bug:**

```json
{
  "projectId": "abc-123-def-456",
  "runs": [
    {
      "runId": "run-004",
      "status": "completed",
      "startedAt": "2024-01-15T13:00:00Z",
      "completedAt": "2024-01-15T13:10:00Z"
    }
  ]
}
```

> **Bug:** Note that `percent_complete` is entirely missing from the run object. The field must always be present in every run object in the response — its absence indicates a defect in the API implementation.

### Sample 5: ❌ Invalid — Wrong data type (string instead of number)

The `percent_complete` value is a string `"100"` instead of a number — **this is a bug:**

```json
{
  "projectId": "abc-123-def-456",
  "runs": [
    {
      "runId": "run-005",
      "status": "completed",
      "percent_complete": "100",
      "startedAt": "2024-01-15T14:00:00Z",
      "completedAt": "2024-01-15T14:12:00Z"
    }
  ]
}
```

> **Bug:** The value `"100"` is a string, not a number — this is a bug. The correct value should be `100` or `100.0` (numeric type). String representations of numbers are never valid for this field.

---

## Edge Cases

The following edge cases must be verified for the `percent_complete` / `percentComplete` field in the `GET /runs/metering` response:

- Field present but:
  - Value > 100 ❌
  - Value < 0 ❌
  - Wrong datatype (string instead of number) ❌
- Field name mismatch (percent_complete vs percentComplete)
- Present in one API but missing in others

### Endpoint-Specific Edge Case Notes

- **Multiple runs verification:** For this endpoint, verify `percent_complete` in **EACH** run object when the response contains multiple runs — the field must be present and valid in every run object, not just the first one.

- **Boundary value testing:** Check for the following boundary values:
  - `0.0` — Valid lower bound ✅
  - `100.0` — Valid upper bound ✅
  - `100.1` — Invalid, exceeds upper bound ❌
  - `-0.1` — Invalid, below lower bound ❌

- **Mixed run states:** When the response contains runs in different states (completed, running, pending), verify that each run's `percent_complete` value is appropriate for its status:
  - Completed runs should have a value between `0.0` and `100.0` (typically `100.0`)
  - In-progress runs should have a value less than `100.0`
  - Pending runs may have `null`

- **Cross-API consistency:** After verifying this endpoint, compare the `percent_complete` field behavior with the other two endpoints (`GET /runs/metering/current` and `GET /project`) to ensure consistent presence, naming, data type, and value range. See the [Validation Matrix & Test Cases](../../test-cases/percent-complete-validation.md) for the full cross-API consistency check procedure.

---

## Related Documentation

- [Back to Verification Overview](../overview.md) — Scope summary and verification workflow
- [Consolidated percent_complete Guide](../metering-percent-complete.md) — All three APIs, trigger-action matrix, cross-API consistency
- [GET /runs/metering/current — Endpoint Guide](get-runs-metering-current.md) — Companion endpoint for current run metering
- [GET /project — Endpoint Guide](get-project.md) — Project endpoint with inline metering data
- [DevTools API Inspection Guide](../../guides/devtools-api-inspection.md) — Browser verification workflow
- [Validation Matrix & Test Cases](../../test-cases/percent-complete-validation.md) — Comprehensive test scenarios
