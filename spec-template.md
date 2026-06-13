---
title: "[Feature Name]"
version: "0.1.0"
date: "YYYY-MM-DD"
status: "draft"  # draft | in-progress | review | approved | implemented
tags: [module-name, category]
author: ""
related_specs: []  # List any related spec files
---

# [Feature Name] Specification

> **Brief description:** One or two sentences describing what this feature does
> and why it exists.

## Overview
<!-- Describe the feature in detail. What problem does it solve?
     Who are the users? What is the expected behavior?
     Provide enough context for a developer to understand the "why" -->

## Requirements

### Functional Requirements
<!-- List each requirement as a numbered item.
     Use "SHALL" or "MUST" for mandatory requirements.
     Use "SHOULD" for recommended behavior.
     Use "MAY" for optional behavior. -->

1. **FR-001:** The system SHALL [describe behavior]
2. **FR-002:** The system SHALL [describe behavior]
3. **FR-003:** The system SHOULD [describe behavior]

### Non-Functional Requirements
<!-- Performance, scalability, reliability requirements -->

- **Performance:** [e.g., "Response time < 200ms at p99"]
- **Scalability:** [e.g., "Support 1000 concurrent users"]
- **Availability:** [e.g., "99.9% uptime"]

## API / Interface Design
<!-- Define the endpoint(s) or interface(s) this spec covers.
     Adapt this section to your project: HTTP endpoints, CLI commands,
     library function signatures, message queue handlers, etc. -->

### `[METHOD] /api/v1/[resource]`

**Request:**

```json
{
  "field_name": "string (required)",
  "optional_field": "string (optional)"
}
```

**Response (201 Created):**

```json
{
  "id": "uuid",
  "field_name": "string",
  "created_at": "ISO 8601"
}
```

**Error Responses:**

| Status | Code | Condition |
|--------|------|-----------|
| 400 | `VALIDATION_ERROR` | Invalid input |
| 409 | `CONFLICT` | Resource already exists |
| 500 | `INTERNAL_ERROR` | Unexpected server error |

<!-- Add more endpoints/interfaces as needed -->

## Data Model

<!-- Describe any new or modified database tables, schemas, or data structures -->

### `table_name`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | uuid | PRIMARY KEY, DEFAULT gen_random_uuid() | Unique identifier |
| created_at | timestamptz | NOT NULL, DEFAULT now() | Creation timestamp |
| updated_at | timestamptz | NOT NULL, DEFAULT now() | Last update timestamp |

## Business Rules

<!-- Any domain logic, validation rules, state machines, etc. -->

1. **Rule:** [Description of business logic]
2. **Rule:** [Description of constraint]

## Validation Rules

<!-- Detailed input validation rules -->

| Field | Rule | Error Message |
|-------|------|---------------|
| email | Required, valid email format | "A valid email is required" |
| password | Required, min 8 chars | "Password must be at least 8 characters" |

## Edge Cases

<!-- What happens in unusual situations? -->

1. **[Scenario]:** [Expected behavior]
2. **[Scenario]:** [Expected behavior]
3. **[Scenario]:** [Expected behavior]

## Acceptance Criteria

<!-- Each criterion should be independently testable.
     Reference these in test files as AC-001, AC-002, etc. -->

- [ ] **AC-001:** [Given [context], when [action], then [expected result]
- [ ] **AC-002:** [Given [context], when [action], then [expected result]
- [ ] **AC-003:** [Given [context], when [action], then [expected result]
- [ ] **AC-004:** [Given [context], when [action], then [expected result]
- [ ] **AC-005:** [Given [context], when [action], then [expected result]

## Test Strategy

<!-- How will this feature be tested? Adapt test categories to your project. -->

### Unit Tests

<!-- What pure logic needs testing? -->

- [ ] [Description of unit test]

### Integration Tests

<!-- What endpoints, database interactions, or external integrations need testing? -->

- [ ] [Description of integration test]

### Snapshot / Contract Tests

<!-- What responses or interfaces should be snapshot-tested or contract-tested? -->

- [ ] [Description of snapshot/contract test]

## Security Considerations

<!-- Security-specific requirements for this feature -->

- [ ] [Security requirement]
- [ ] [Security requirement]

## Implementation Notes

<!-- Optional: hints for implementation, dependencies, order of work -->

- Depends on: [list dependencies]
- Estimated complexity: [low | medium | high]
- Suggested implementation order: [steps]

## Changelog

<!-- Track spec revisions -->

| Version | Date | Changes |
|---------|------|---------|
| 0.1.0 | YYYY-MM-DD | Initial draft |
