# API Verification & Testing Guide — percent_complete Field

This repository provides comprehensive API verification and testing documentation for validating the `percent_complete` / `percentComplete` field across three code generation project run APIs. It is designed for **QA engineers**, **front-end developers**, **back-end developers**, and **technical testers** who need to manually verify the correct implementation of the new progress-tracking field using browser DevTools.

The documentation covers step-by-step browser DevTools Network tab inspection procedures, per-endpoint test guides, validation matrices, edge case catalogs, and cross-API consistency checks — everything needed to confirm that the `percent_complete` field is present, correctly typed, and within the valid range across all three endpoints.

---

## Table of Contents

- [Quick Start](#quick-start)
- [APIs Under Test](#apis-under-test)
- [Documentation Index](#documentation-index)
- [Field Under Verification](#field-under-verification)
- [Validation Quick Reference](#validation-quick-reference)
- [Edge Cases Quick Reference](#edge-cases-quick-reference)
- [Contributing](#contributing)

---

## Quick Start

Get started verifying the `percent_complete` field in four steps:

1. Open the [Verification Overview](docs/api-verification/overview.md) to understand what's being tested
2. Review the [DevTools Inspection Guide](docs/guides/devtools-api-inspection.md) for browser setup
3. Follow the per-endpoint guides for detailed test procedures
4. Validate using the [Test Case Matrix](docs/test-cases/percent-complete-validation.md)

---

## APIs Under Test

The following three API endpoints are under verification for the `percent_complete` / `percentComplete` field:

| Endpoint | URL Pattern | Trigger Condition |
|----------|-------------|-------------------|
| `GET /runs/metering` | `/runs/metering?projectId=xxx` | Viewing run history, fetching metering data |
| `GET /runs/metering/current` | `/runs/metering/current` | Run in progress, live status view, auto-refresh/polling |
| `GET /project` | `/project?id=xxx` | Opening a project page |

---

## Documentation Index

### API Verification

- [Verification Overview](docs/api-verification/overview.md) — Scope summary, verification workflow, and quick links
- [Consolidated percent_complete Guide](docs/api-verification/metering-percent-complete.md) — All three APIs, trigger-action matrix, cross-API consistency
- [GET /runs/metering](docs/api-verification/endpoints/get-runs-metering.md) — Endpoint spec, trigger conditions, response schema, validation scenarios
- [GET /runs/metering/current](docs/api-verification/endpoints/get-runs-metering-current.md) — Endpoint spec, polling behavior, validation scenarios
- [GET /project](docs/api-verification/endpoints/get-project.md) — Endpoint spec, inline metering data, validation scenarios

### Guides

- [DevTools API Inspection](docs/guides/devtools-api-inspection.md) — Step-by-step browser Network tab verification workflow

### Test Cases

- [percent_complete Validation Matrix](docs/test-cases/percent-complete-validation.md) — Validation matrix, edge cases, pass/fail criteria

---

## Field Under Verification

Key facts about the `percent_complete` / `percentComplete` field:

- **Field names:** `percent_complete` (snake_case) and `percentComplete` (camelCase) — both variants must be checked
- **Data type:** Numeric (number), NOT string
- **Value range:** `0.0` to `100.0` inclusive, or `null`
- **Context:** Code generation project runs
- **Bug indicators:** Field missing entirely, value > 100, value < 0, wrong data type (string instead of number), field name inconsistency across endpoints

---

## Validation Quick Reference

| Scenario        | Expected               |
|-----------------|------------------------|
| Completed run   | Value between 0–100    |
| In-progress run | Likely < 100           |
| No data         | null                   |
| Field missing   | ❌ Bug                 |

For the full validation matrix with expanded test scenarios and pass/fail criteria, see [percent_complete Validation Matrix](docs/test-cases/percent-complete-validation.md).

---

## Edge Cases Quick Reference

- Field present but:
  - Value > 100 ❌
  - Value < 0 ❌
  - Wrong datatype (string instead of number) ❌
- Field name mismatch (percent_complete vs percentComplete)
- Present in one API but missing in others

For the complete edge case catalog with detailed test cases and severity ratings, see [percent_complete Validation Matrix](docs/test-cases/percent-complete-validation.md).

---

## Contributing

To suggest updates or improvements to this documentation:

1. Open an issue describing the proposed change or correction
2. Reference the specific documentation file and section that needs updating
3. Include any supporting evidence (e.g., API response screenshots, expected vs actual behavior)
4. Submit a pull request with the proposed changes for review

All documentation follows standard GitHub-flavored Markdown with tables, fenced code blocks, and inline Mermaid diagrams. Please maintain consistency with the existing documentation style when contributing.
