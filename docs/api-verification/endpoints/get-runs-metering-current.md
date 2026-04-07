# GET /runs/metering/current — Endpoint Specification

> **Detailed specification and verification guide for the GET /runs/metering/current API endpoint and its `percent_complete` field**

This document provides the detailed specification for the `GET /runs/metering/current` endpoint. The endpoint returns metering data for the currently active code generation run, including the `percent_complete` (snake_case) / `percentComplete` (camelCase) progress-tracking field that reflects real-time progress of the active run.

This guide covers the URL pattern, trigger conditions, request parameters, expected response schema, polling behavior documentation, validation scenarios, and sample responses for this endpoint.

This file is part of the API verification documentation suite for validating the `percent_complete` field across all code generation project run APIs. See the [Verification Overview](../overview.md) for the full scope.

**Target audience:** QA engineers, front-end developers, back-end developers, and technical testers.

---

## Endpoint Details

| Property | Value |
|----------|-------|
| **HTTP Method** | `GET` |
| **URL Pattern** | `/runs/metering/current` |
| **Authentication** | Required (application session) |
| **Content Type** | `application/json` |

> **Note:** The URL pattern `/runs/metering/current` must be used exactly as shown above.

---

## When This API Appears

This API is triggered under the following conditions within the application:

- **Run currently in progress:** This API is called when a code generation run is actively executing and the application needs to check its current status. The request appears in the Network tab whenever the system fetches the latest progress for the ongoing run.

- **Viewing live run status:** This API is called when the user navigates to a page showing the live progress of an active code generation run. The front-end application requests the current run's metering data to display real-time progress to the user.

- **Auto-refresh / polling:** This API may be called automatically at regular intervals during an active run to fetch updated progress data. Watch for multiple repeated requests to `/runs/metering/current` in the Network tab — this indicates the application is polling for progress updates.

> **How to find this request:** Look for this request in the browser DevTools Network tab when performing the above actions. Filter by **XHR/Fetch** and search for `metering` or `current`. During an active run, this endpoint may appear **multiple times** due to auto-refresh/polling behavior. See the [DevTools API Inspection Guide](../../guides/devtools-api-inspection.md) for step-by-step instructions.

### Polling Behavior

During an active code generation run, the frontend application may periodically call this endpoint to get updated progress. This behavior has the following characteristics:

- **Multiple requests in sequence:** Multiple requests to `/runs/metering/current` may appear in the Network tab in sequence during an active run
- **Increasing progress value:** The `percent_complete` value should increase over time as the run progresses — each subsequent polling response should show equal or higher progress
- **Enable Preserve Log:** Enable **Preserve Log** in DevTools before starting a run to capture all polling requests without losing them on page navigation
- **Validate each response:** Each polling response should be individually validated — the `percent_complete` value should be consistently numeric and within range (`0.0`–`100.0`) across all responses
- **Monotonic increase:** The `percent_complete` value should monotonically increase (or at least not decrease) during a single run — a decreasing value may indicate a bug

---

## Request Parameters

This endpoint typically has **no required query parameters**. It returns metering data for the currently active run based on the user's session context.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| *(None or contextual)* | — | — | This endpoint determines the current run from the application session or context. No explicit query parameters are typically required. |

> **Note:** The current run context is determined server-side from the authenticated user session. Unlike `GET /runs/metering` which requires a `projectId` parameter, this endpoint identifies the current run automatically.

---

## Expected Response Schema

The response contains metering data for the **currently active** code generation run. Unlike `GET /runs/metering` which returns an array of runs, this endpoint returns data for a **single run** (the current one). The `percent_complete` field reflects real-time progress of the active run.

### Response Structure

```json
{
  "runId": "string",
  "status": "string",
  "percent_complete": "number | null",
  "startedAt": "string (ISO 8601 timestamp)"
}
```

> **Note:** The response structure is for a **SINGLE run** (the current one). The `percent_complete` field appears at the **top level** of the response object.

### percent_complete Field

The following table defines the specification for the `percent_complete` / `percentComplete` field as it appears in this endpoint's response:

| Property | Specification |
|----------|--------------|
| **Field Name (snake_case)** | `percent_complete` |
| **Field Name (camelCase)** | `percentComplete` |
| **Data Type** | Numeric (`number`) — **NOT** string |
| **Valid Range** | `0.0` to `100.0` inclusive |
| **Null Allowed** | Yes — `null` when no data available or no current run |
| **Location in Response** | Top-level field in the response object (single run) |
| **Real-time behavior** | Value increases over time during an active run |

**Data type clarification:**

- `50` or `50.0` (number) → ✅ **VALID**
- `"50"` (string) → ❌ **INVALID** — this is a string, not a number

**Field name variants:**

Both `percent_complete` (snake_case) and `percentComplete` (camelCase) must be checked when inspecting the response. The user requirements explicitly identify field name inconsistency as a potential issue to verify. If this endpoint uses a different naming convention than the other two endpoints (`GET /runs/metering` and `GET /project`), this should be flagged as a ⚠️ **naming inconsistency**.

---

## Validation Scenarios

The following validation matrix defines the core expected outcomes for the `percent_complete` / `percentComplete` field:

| Scenario        | Expected               |
|-----------------|------------------------|
| Completed run   | Value between 0–100    |
| In-progress run | Likely < 100           |
| No data         | null                   |
| Field missing   | ❌ Bug                 |

### Detailed Test Scenarios

The following expanded table provides detailed test scenarios specific to the `GET /runs/metering/current` endpoint:

| Scenario | Pre-condition | Action | Expected `percent_complete` | Result |
|----------|--------------|--------|----------------------------|--------|
| Completed run | Last run completed, no active run | Check current run status | Numeric `0.0`–`100.0` or `null` | ✅ Pass if present and valid |
| In-progress run | Code generation run actively running | Check current run status | Numeric value `< 100.0` | ✅ Pass if present and < 100 |
| In-progress run (polling) | Code generation run actively running | Auto-refresh/polling | Value increases over time, always numeric and `< 100.0` until completion | ✅ Pass if value increases and stays in range |
| No data / No current run | No run currently in progress | Check current run status | `null` | ✅ Pass if field present and value is `null` |
| Field missing | Any run state | Check current run status | Field absent from response | ❌ **Bug** |
| Run just started | Code generation run just initiated | Check current run status immediately | Numeric `0.0` or small value, or `null` if data not yet populated | ✅ Pass if field present and value is valid |

---

## Sample Responses

The following sample JSON responses illustrate expected and invalid responses from the `GET /runs/metering/current` endpoint. Use these as reference when inspecting actual API responses.

### ✅ Sample 1: Active run in progress

A code generation run is actively running and approximately 67.3% complete:

```json
{
  "runId": "run-003",
  "status": "running",
  "percent_complete": 67.3,
  "startedAt": "2024-01-15T12:00:00Z"
}
```

### ✅ Sample 2: No current run (null value)

No code generation run is currently in progress. The `percent_complete` field is present with a `null` value:

```json
{
  "runId": null,
  "status": "idle",
  "percent_complete": null
}
```

### ✅ Sample 3: Completed run

The most recent run has completed with 100% progress:

```json
{
  "runId": "run-001",
  "status": "completed",
  "percent_complete": 100.0,
  "startedAt": "2024-01-15T10:00:00Z"
}
```

### ✅ Sample 4: Run just started

A code generation run was just initiated. The `percent_complete` field shows 0.0 progress:

```json
{
  "runId": "run-007",
  "status": "running",
  "percent_complete": 0.0,
  "startedAt": "2024-01-15T15:00:00Z"
}
```

### ❌ Sample 5: Invalid — field missing (bug indicator)

The `percent_complete` field is entirely absent from the response — **this is a bug**:

```json
{
  "runId": "run-008",
  "status": "running",
  "startedAt": "2024-01-15T16:00:00Z"
}
```

> **Bug:** Note that `percent_complete` is entirely missing from the response. The field must always be present, even if its value is `null`.

### ❌ Sample 6: Invalid — wrong data type (string instead of number)

The `percent_complete` field contains a string value instead of a number — **this is a bug**:

```json
{
  "runId": "run-009",
  "status": "running",
  "percent_complete": "67",
  "startedAt": "2024-01-15T17:00:00Z"
}
```

> **Bug:** The value `"67"` is a string, not a number. The correct value should be `67` or `67.0` (numeric type).

---

## Edge Cases

The following edge cases must be verified for the `percent_complete` / `percentComplete` field in this endpoint:

- Field present but:
  - Value > 100 ❌
  - Value < 0 ❌
  - Wrong datatype (string instead of number) ❌
- Field name mismatch (percent_complete vs percentComplete)
- Present in one API but missing in others

### Endpoint-Specific Edge Cases

In addition to the general edge cases above, the following are specific to the `GET /runs/metering/current` endpoint due to its real-time and polling nature:

- **Value jumps to null unexpectedly:** During polling, watch for the value to jump from a valid in-progress value directly to `null` or skip values unexpectedly — this could indicate a data consistency issue between the backend and the polling mechanism.

- **Decreasing progress value:** During polling, the `percent_complete` value should monotonically increase (or at least not decrease). A decreasing value (e.g., going from `67.3` to `45.0`) may indicate a bug in the progress tracking logic.

- **Boundary values:** Check these specific boundary values:
  - `0.0` — ✅ Valid lower bound (expected at run start)
  - `100.0` — ✅ Valid upper bound (expected at completion)
  - `100.1` — ❌ Invalid (exceeds upper bound)
  - `-0.1` — ❌ Invalid (below lower bound)

- **Rapid polling consistency:** When multiple polling responses are received in quick succession, each response should independently satisfy all validation criteria (numeric type, within range 0.0–100.0 or null).

---

## Related Documentation

| Document | Link | Description |
|----------|------|-------------|
| Verification Overview | [Back to Verification Overview](../overview.md) | Scope summary and verification workflow |
| Consolidated Guide | [Consolidated percent_complete Guide](../metering-percent-complete.md) | All three APIs, trigger-action matrix, cross-API consistency |
| GET /runs/metering | [GET /runs/metering — Endpoint Guide](get-runs-metering.md) | Companion endpoint for historical run metering |
| GET /project | [GET /project — Endpoint Guide](get-project.md) | Project endpoint with inline metering data |
| DevTools Guide | [DevTools API Inspection Guide](../../guides/devtools-api-inspection.md) | Browser verification workflow |
| Validation & Test Cases | [Validation Matrix & Test Cases](../../test-cases/percent-complete-validation.md) | Comprehensive test scenarios |
