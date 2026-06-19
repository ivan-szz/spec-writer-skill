---
title: "[Feature Name]"
version: "0.1.0"
date: "YYYY-MM-DD"
status: "draft"  # draft | in-progress | review | approved | implemented
tags: [module-name, category]
author: ""
related_specs: []  # List any related spec files
---

# [Feature Name]

## Metadata
- **ID**: SPEC-XXX (auto-incremented)
- **Status**: Draft | Ready | Implemented | Deprecated
- **Author**: <name or handle>
- **Created**: YYYY-MM-DD
- **Last Updated**: YYYY-MM-DD
- **Depends On**: <list of other spec IDs or "None">

## Purpose & Scope
<!-- What this spec covers AND what it explicitly does NOT cover. -->

## Definitions
<!-- Terminology table. Every domain term used in this spec must appear here. -->
| Term | Definition |
|------|-----------|
|      |           |

## Requirements
<!-- Every requirement gets a unique ID: REQ-XXX -->
| ID | Description | Priority | Rationale |
|----|------------|----------|-----------|
| REQ-001 | | Must / Should / Could | |

## Constraints
<!-- Technical, business, or regulatory limits: CON-XXX -->
| ID | Description | Source |
|----|------------|--------|
| CON-001 | | |

## Security
<!-- Threat surface and mitigations: SEC-XXX -->
| ID | Requirement | Mitigation |
|----|------------|------------|
| SEC-001 | | |

## Interfaces & Data Contracts
<!-- API endpoints, CLI commands, message formats, function signatures, etc. -->
<!-- For each interface, define: method/verb, path/command, input, success output, error outputs. -->

### Interface: `METHOD /path`

**Description**: <one line>

**Request/Input**:
```json
{ "field": "type — description" }
```

**Success Response** (`HTTP 2XX` or equivalent success status):
```json
{ "field": "type — description" }
```

**Error Responses**:
| Status | Error Code | Description |
|--------|-----------|-------------|
| 4XX/5XX | `ERR_CODE` | |

## Acceptance Criteria
<!-- GIVEN-WHEN-THEN format. Minimum 3 per spec. -->
<!-- AC-X: Given <precondition>, when <action>, then <expected outcome>. -->

### AC-1
**Given** …
**When** …
**Then** …

### AC-2
**Given** …
**When** …
**Then** …

### AC-3
**Given** …
**When** …
**Then** …

## Verification
<!-- How each acceptance criterion will be confirmed. -->
| AC | Method | Notes |
|----|--------|-------|
| AC-1 | | |

## Edge Cases & Error Scenarios
<!-- Every row must include an HTTP status code where applicable. -->

| ID | Scenario | Expected Behavior | HTTP Status |
|----|----------|-------------------|-------------|
| EDGE-001 | | | |

## Rationale
<!-- Why these design decisions? Link to prior art, ADRs, or trade-off analysis. -->
