# GET /project — Endpoint Specification

> **Detailed specification and verification guide for the GET /project API endpoint and its inline metering data including the `percent_complete` field**

This document provides the detailed specification for the `GET /project` API endpoint. The endpoint returns project data with inline metering information embedded in the response, including the `percent_complete` / `percentComplete` progress-tracking field for code generation project runs.

**Key distinction from other endpoints:** Unlike the dedicated metering endpoints (`GET /runs/metering` and `GET /runs/metering/current`), the `percent_complete` field in the `GET /project` response is **NESTED** within the project response structure — it is not a top-level field. This guide includes specific instructions on how to locate the field within the nested response.

This guide covers:

- URL pattern and HTTP method
- Trigger conditions (which user actions invoke this API)
- Request parameters
- Expected response schema with nested metering data
- **How to locate the `percent_complete` field** in the nested response (key section unique to this endpoint)
- Validation scenarios and sample JSON responses
- Edge cases and bug indicators

This document is part of the API verification documentation suite for validating the `percent_complete` field across all code generation project run APIs.

**Target audience:** QA engineers, front-end developers, back-end developers, and technical testers.

---

## Endpoint Details

| Property | Value |
|----------|-------|
| **HTTP Method** | `GET` |
| **URL Pattern** | `/project?id=xxx` |
| **Authentication** | Required (application session) |
| **Content Type** | `application/json` |

> **Note:** The URL pattern `/project?id=xxx` is the exact pattern to look for in the browser DevTools Network tab. The `xxx` is replaced by the actual project identifier.

---

## When This API Appears

This API is triggered under the following conditions:

- **Opening a project page:** This API is called when the user navigates to or opens a project page in the application. It is one of the first API calls made when a project page loads, fetching all project data including inline metering information.

- **Refreshing the project dashboard:** This API is called when the project page is refreshed (browser refresh or in-app refresh), reloading all project data including the latest inline metering information with the `percent_complete` field.

> **How to capture this request:** Look for this request in the browser DevTools Network tab when performing the above actions. Filter by **XHR/Fetch** and search for `project`. This is typically one of the first API calls made when a project page loads. See the [DevTools API Inspection Guide](../../guides/devtools-api-inspection.md) for step-by-step instructions.

---

## Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | String | Yes | The unique identifier of the project whose data is being requested |

The `id` parameter is passed as a query string parameter in the URL.

**Example request:**

```
GET /project?id=abc-123-def-456
```

---

## Expected Response Schema

The response from `GET /project` contains the **full project data object**, which includes inline metering information. The `percent_complete` field is **NESTED** within the metering section of the project response — it is **NOT** a top-level field.

The following JSON structure shows the relevant portion of the response where the `percent_complete` field appears:

```json
{
  "id": "string",
  "name": "string",
  "description": "string",
  "metering": {
    "currentRun": {
      "runId": "string",
      "status": "string",
      "percent_complete": "number | null"
    }
  }
}
```

> **Note:** The actual response contains additional project fields beyond what is shown above (e.g., creation date, owner, settings, etc.). The relevant section for verification is the `metering` object and its nested `currentRun.percent_complete` field. Focus your verification on this nested path.

---

### How to Locate the percent_complete Field

**This section is critical for this endpoint.** Unlike the dedicated metering endpoints where `percent_complete` is at or near the top level, the `GET /project` response embeds it within nested inline metering data. Follow these steps to navigate to the field:

1. **Open the API response** in the DevTools **Response** or **Preview** tab (click the request in the Network tab, then select Response or Preview)
2. **If using the Preview tab**, expand the root object by clicking the disclosure triangle (▶)
3. **Look for a `metering` key** (or similar key) that contains run/metering information within the project data
4. **Expand the `metering` object** by clicking its disclosure triangle
5. **Look for a `currentRun` key** (or similar nested object) inside the metering section
6. **Inside the nested run object**, find the `percent_complete` / `percentComplete` field — this is the field under verification
7. **Alternatively, use the search shortcut:** Press `Ctrl+F` (Windows/Linux) or `Cmd+F` (macOS) to open the in-response search, then type `percent_complete` or `percentComplete` to jump directly to the field

> **Note:** The exact nesting path may vary slightly depending on the API version — the key is to locate the metering-related section within the project response and find the `percent_complete` field within it. The search shortcut (step 7) is the fastest method.

---

### percent_complete Field Specification

| Property | Specification |
|----------|--------------|
| **Field Name (snake_case)** | `percent_complete` |
| **Field Name (camelCase)** | `percentComplete` |
| **Data Type** | Numeric (`number`) — **NOT** string |
| **Valid Range** | `0.0` to `100.0` inclusive |
| **Null Allowed** | Yes — `null` when no data is available |
| **Location in Response** | **Nested** within inline metering data (e.g., `metering.currentRun.percent_complete`) |

**Data type clarification:**

- `50` or `50.0` (number) → ✅ **VALID**
- `"50"` (string) → ❌ **INVALID** — the value must always be a numeric type, never a string representation of a number

**Field name variants:** Both `percent_complete` (snake_case) and `percentComplete` (camelCase) must be checked. The user requirements explicitly identify naming inconsistency between these variants as a potential issue to verify. Ensure the field name used in this endpoint is consistent with the naming convention used in the other two metering endpoints (`GET /runs/metering` and `GET /runs/metering/current`).

**Nested location — key difference:** The nested location within inline metering data is the **KEY DIFFERENCE** from the other two endpoints, where `percent_complete` is at or near the top level of the response. Testers must navigate through the project response structure to locate the field, rather than finding it immediately in the top-level JSON.

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

The following expanded table provides endpoint-specific test scenarios for `GET /project`:

| Scenario | Pre-condition | Action | Expected `percent_complete` | Result |
|----------|--------------|--------|----------------------------|--------|
| Completed run | Project has completed code generation runs | Open project page | Numeric `0.0`–`100.0` (typically `100.0`) in inline metering data | ✅ Pass if present and in range |
| In-progress run | Project with active code generation run | Open project page | Numeric value `< 100.0` in inline metering data | ✅ Pass if present and < 100 |
| No data / New project | New project with no runs | Open project page | `null` in inline metering data | ✅ Pass if field present and value is `null` |
| Field missing | Any project | Open project page | Field absent from inline metering data | ❌ **Bug** |
| Project with no metering section | Project without metering data structure | Open project page | Metering section or field absent | ❌ **Bug** — metering section should always be present |
| Refresh project dashboard | Project with any run state | Refresh project page | Same expected values as initial load | ✅ Pass if field present and valid |

---

## Sample Responses

The following sample JSON responses illustrate where the `percent_complete` field appears within the nested `GET /project` response structure. Each sample uses fenced code blocks with JSON syntax highlighting.

### ✅ Sample 1: Project with completed run and inline metering data

```json
{
  "id": "abc-123-def-456",
  "name": "My Project",
  "description": "Code generation project",
  "metering": {
    "currentRun": {
      "runId": "run-001",
      "status": "completed",
      "percent_complete": 100.0
    }
  }
}
```

The `percent_complete` field is `100.0` — a numeric value within the valid range, nested inside `metering.currentRun`.

### ✅ Sample 2: Project with no metering data (null value)

```json
{
  "id": "abc-123-def-456",
  "name": "My Project",
  "description": "Code generation project",
  "metering": {
    "currentRun": {
      "runId": null,
      "status": "none",
      "percent_complete": null
    }
  }
}
```

The `percent_complete` field is `null` — this is valid for a project with no active or completed runs. The field is present but has no data.

### ✅ Sample 3: Project with in-progress run

```json
{
  "id": "abc-123-def-456",
  "name": "My Project",
  "description": "Code generation project",
  "metering": {
    "currentRun": {
      "runId": "run-005",
      "status": "running",
      "percent_complete": 33.7
    }
  }
}
```

The `percent_complete` field is `33.7` — a numeric value less than `100.0`, indicating the run is approximately one-third complete.

### ❌ Sample 4: Invalid — percent_complete field missing from inline metering (bug indicator)

```json
{
  "id": "abc-123-def-456",
  "name": "My Project",
  "description": "Code generation project",
  "metering": {
    "currentRun": {
      "runId": "run-006",
      "status": "completed"
    }
  }
}
```

**Bug:** The `percent_complete` field is entirely missing from the `currentRun` object within the `metering` section. The field must be present even when nested within the project response — its absence is a defect regardless of the run status.

### ❌ Sample 5: Invalid — wrong data type (string instead of number)

```json
{
  "id": "abc-123-def-456",
  "name": "My Project",
  "description": "Code generation project",
  "metering": {
    "currentRun": {
      "runId": "run-007",
      "status": "completed",
      "percent_complete": "100"
    }
  }
}
```

**Bug:** The value `"100"` is a string, not a number. The correct value should be `100` or `100.0` (numeric type). String representations of numbers are invalid for this field.

---

## Edge Cases

The following edge cases must be verified for the `percent_complete` / `percentComplete` field:

- Field present but:
  - Value > 100 ❌
  - Value < 0 ❌
  - Wrong datatype (string instead of number) ❌
- Field name mismatch (percent_complete vs percentComplete)
- Present in one API but missing in others

### Endpoint-Specific Edge Case Notes

For this endpoint, the `percent_complete` field is **NESTED** within the project response. Additional edge cases specific to `GET /project` include:

- **Verify the `metering` section is present** in the project response. If the `metering` key is entirely absent from the response, the inline metering data structure may not be enabled or there is a structural issue with the API.

- **Verify the nested structure** (e.g., `metering.currentRun`) contains the `percent_complete` field. The field must exist within the nested run object, not at the project level.

- **Verify field name consistency** with the other two metering endpoints. The field name used in the nested `GET /project` response should match the naming convention used in `GET /runs/metering` and `GET /runs/metering/current`.

- **Check boundary values:**
  - `0.0` — ✅ Valid (lower bound inclusive)
  - `100.0` — ✅ Valid (upper bound inclusive)
  - `100.1` — ❌ Invalid (exceeds upper bound)
  - `-0.1` — ❌ Invalid (below lower bound)

- **If the `metering` section is entirely absent** from the project response, this may indicate the metering feature is not enabled for this project or there is a structural issue with the API. This should be flagged for investigation.

---

## Related Documentation

- [Back to Verification Overview](../overview.md) — Scope summary and verification workflow
- [Consolidated percent_complete Guide](../metering-percent-complete.md) — All three APIs, trigger-action matrix, cross-API consistency
- [GET /runs/metering — Endpoint Guide](get-runs-metering.md) — Companion endpoint for historical run metering
- [GET /runs/metering/current — Endpoint Guide](get-runs-metering-current.md) — Companion endpoint for current run metering
- [DevTools API Inspection Guide](../../guides/devtools-api-inspection.md) — Browser verification workflow
- [Validation Matrix & Test Cases](../../test-cases/percent-complete-validation.md) — Comprehensive test scenarios
