---
name: spec-writer
description: "Interactive specification authoring. Guides the user through a structured dialogue to gather requirements, then produces deterministic, reviewable .md specs. Use when the user wants to create, refine, or validate a feature specification."
metadata:
  version: "1.0.0"
  argument-hint: "[write|refine|validate] [target]"
---

# Spec Writer Skill

Author well-structured specification documents through an interactive dialogue. The agent is an **interlocutor**, not a generator — it asks, probes, challenges, and clarifies before writing anything. Every spec produced is a complete, self-contained `.md` file that can be reviewed, versioned, and traced to implementation.

## Core Principle

**Never generate a spec from a one-liner.** When the user says "write a spec for X", start a dialogue. The quality of the spec is determined by the quality of the questions asked, not by the speed of generation.

## Setup

Before doing anything:

1. **Read the project context.** Check for `AGENTS.md`, `CLAUDE.md`, `*.instructions.md`, or similar project instruction files. Understand the tech stack, conventions, and existing specs.
2. **Check for existing specs.** Look in `specs/` for any `.spec.md` files. If one exists for the requested feature, warn the user and ask whether to refine the existing one or create a new version.
3. **Check for spec template.** Load `specs/spec-template.md` as the structural reference. This is the single source of truth for spec structure.

## Commands

| Command | Description |
|---------|-------------|
| `write <feature>` | Start a dialogue to create a new spec |
| `refine <spec-path>` | Review and improve an existing spec |
| `validate [spec-path]` | Validate one or all specs against the rule set |

### Routing

- **No argument**: show available specs in `specs/`, suggest next actions.
- **First word matches a command**: load and execute it.
- **First word doesn't match, but intent maps** (e.g. "spec for auth" → `write auth`): route to the matching command.
- **Ambiguous**: ask once which command the user wants.

---

## The Dialogue Flow

When the user invokes `write`, follow this flow. **Do not skip steps. Do not generate the spec before completing the dialogue.**

### Round 1 — Understand the Shape

Ask 2-3 focused questions to understand what the feature IS:

1. **What does it do?** — One sentence. Not a paragraph. If the user gives a paragraph, help them distill it to one sentence.
2. **Who uses it?** — End user, admin, external service, another system? Different users mean different interfaces.
3. **What's the primary interface?** — REST endpoint? CLI command? Library function? WebSocket event? This determines the spec structure.

Wait for answers before proceeding.

### Round 2 — Probe the Boundaries

Ask 2-3 questions about what the feature ISN'T and where it breaks:

1. **What's explicitly out of scope?** — If not stated, ask. Unstated scope is the #1 source of spec drift.
2. **What can go wrong?** — Invalid input, missing data, concurrent access, network failures, auth expiry. Pick the 2-3 most relevant based on the feature type.
3. **Are there constraints?** — Performance requirements, regulatory limits, compatibility with existing systems, database schema limits.

Wait for answers before proceeding.

### Round 3 — Deepen the Details

Ask 1-2 questions about specifics that affect the spec structure:

1. **What's the success response?** — Exact shape, fields, status codes. If the user is vague ("returns the user"), push for specifics ("what fields? what status code?").
2. **What are the error cases?** — Not just "it should handle errors" — which specific errors, with which status codes?
3. **Is there state?** — Database writes, cache updates, queue messages, file mutations. Anything with state changes needs concurrency analysis.

If the user has been detailed in earlier rounds, this round can be shorter. If they've been vague, this round must be longer.

### Round 4 — Confirm and Structure

Before writing anything:

1. **Summarize what you heard.** 3-5 bullet points covering: feature purpose, primary interface, key requirements, constraints, error cases.
2. **Ask: "Is this correct? Anything missing?"**
3. **Only after confirmation**, proceed to generate the spec.

If the user corrects something, update the summary and re-confirm.

### Round 5 — Generate and Validate

1. **Write the spec** following the template below.
2. **Self-validate** using the validation rules.
3. **Present the spec** to the user for review.
4. **After approval**, write the file to `specs/[(auto-incremted id)]-[feature].spec.md`.

---

## Probing Rules

These rules govern HOW you ask questions, not WHAT you ask.

### Challenge Vagueness

When the user says something vague, **do not accept it and fill in your own interpretation.** Instead:

- User: "It should handle errors properly"
- Bad: *writes spec with generic error handling*
- Good: "What specific errors? Validation failure (400)? Not found (404)? Unauthorized (401)? Something else?"

### Probe for Edge Cases

Every feature has edge cases. Your job is to surface them. Based on the feature type, probe for:

- **Input validation**: missing fields, wrong types, oversized values, special characters
- **State mutations**: concurrent access, race conditions, partial failures
- **External dependencies**: timeout, unavailability, rate limiting
- **Security**: authentication, authorization, injection, data exposure
- **Performance**: large payloads, high concurrency, slow backends

Don't ask all of these — pick the 2-3 most relevant to the specific feature.

### Uncover Implicit Requirements

Users often assume things they don't say. Surface them:

- "Is this synchronous or async?"
- "Does this need pagination?"
- "Should this be idempotent?"
- "What happens if the same request is sent twice?"

### Stop When You Have Enough

The dialogue stops when you can confidently fill every section of the spec template. Signs you have enough:

- You can write the Requirements table without placeholders
- You can define at least 3 acceptance criteria in Given-When-Then
- You can fill the Verification table mapping each AC to a method
- You can fill the Interfaces section with concrete request/response shapes
- You can identify at least 2 edge cases with expected behavior

If you're still guessing on a section, ask one more question about that section specifically.

---

## Spec Template

The canonical template lives at `specs/spec-template.md`. Load it at the start of every `write` or `refine` command. Every generated spec MUST contain every section defined there, in that order. Empty sections are not permitted — write `<!-- intentional: N/A with justification -->` with a one-line rationale if a section genuinely does not apply.

---

## Validation Rules

The `validate` command evaluates every spec against these deterministic rules. A spec **passes** only when ALL applicable rules report PASS.

### Structural Rules

| Rule ID | Rule | Check |
|---------|------|-------|
| **REQ-001** | Minimum acceptance criteria | Every spec must contain **at least 3** `### AC-N` subsections inside Acceptance Criteria. |
| **REQ-002** | Interfaces define success and error | Every `### Interface` block must contain both a **Success Response** and an **Error Responses** table. |
| **REQ-003** | Unique requirement IDs | Every row in the Requirements table must have a unique `REQ-XXX` ID. No duplicates. No missing IDs. |

### Security Rules

| Rule ID | Rule | Check |
|---------|------|-------|
| **SEC-001** | Token/session lifecycle for auth specs | If the spec's domain is `auth`, `session`, `token`, or the word "authentication" appears in Purpose & Scope, the Security table must contain at least one row addressing **token/session creation, refresh, and revocation**. |
| **SEC-002** | Explicit input validation | Every interface's Request/Input block must either (a) list validation constraints per field (e.g., `min_length`, `max_length`, `pattern`, `required`), or (b) the Constraints table must contain a `CON-XXX` row that references the interface and defines the validation rules. |

### Edge Case Rules

| Rule ID | Rule | Check |
|---------|------|-------|
| **EDGE-001** | HTTP status codes in error scenarios | Every row in the Edge Cases & Error Scenarios table must have a non-empty **HTTP Status** column. |
| **EDGE-002** | Concurrent access addressed | If the spec touches a shared resource (database write, cache mutation, queue append, file write), the Edge Cases table must contain at least one row describing a **concurrent access scenario** (e.g., race condition, optimistic lock failure). Shared resources are detected by keywords: `INSERT`, `UPDATE`, `DELETE`, `cache`, `queue`, `counter`, `lock`, `mutex`, `concurrent`. |

### Verification Rules

| Rule ID | Rule | Check |
|---------|------|-------|
| **VER-001** | Every AC has ≥1 verification entry | For each `AC-N` in Acceptance Criteria, the Verification table must contain at least one row with `AC` = `AC-N`. |

### Output Format

```
=== Spec Validation Report ===
File: specs/auth-register.md

[PASS] REQ-001 — 4 acceptance criteria found (minimum: 3)
[PASS] REQ-002 — All 3 interfaces define success + error responses
[PASS] REQ-003 — 8 unique requirement IDs, 0 duplicates
[PASS] SEC-001 — Token lifecycle addressed in SEC-003
[PASS] SEC-002 — Input validation defined for all interfaces
[PASS] EDGE-001 — All 5 error scenarios include HTTP status codes
[PASS] EDGE-002 — Concurrent access scenario present for database writes
[PASS] VER-001 — All 4 acceptance criteria map to ≥1 verification entry

Result: 8/8 PASS — spec is valid
```

---

## Antipattern Registry

These patterns are **forbidden** in generated specs. If any are detected during validation, they produce a hard FAIL with a fix suggestion.

| ID | Antipattern | Detection | Fix |
|----|------------|-----------|-----|
| **ANT-001** | Vague acceptance criteria | AC text contains "should work correctly", "handles it", "is secure" without concrete measurable outcomes. | Rewrite with Given-When-Then and a concrete assertion. |
| **ANT-002** | Missing error contract | Interface defined without any error response entries. | Add at minimum: 400 (validation), 401 (auth), 404 (not found), 500 (internal). |
| **ANT-003** | Unbounded inputs | Request field lacks validation constraint and no CON-XXX references it. | Add `max_length` / `max_items` / pattern constraint to the field or a CON row. |
| **ANT-004** | Orphan requirements | REQ row exists but no AC or implementation note references it. | Link to at least one AC or mark as deferred with rationale. |
| **ANT-005** | Silent deletion | Spec includes a delete endpoint but no edge case covers what happens to dependent resources. | Add EDGE row describing cascade, soft-delete, or reference integrity behavior. |
| **ANT-006** | Magic numbers | Spec uses literal numeric values (e.g., `3600`, `5`) without explanation. | Wrap in a named constant in the Rationale or Constraints section. |

---

## Refine Flow

When the user invokes `refine <spec-path>`:

1. **Read the existing spec.** Understand what it says.
2. **Self-validate** using the validation rules. Report any FAIL items.
3. **Identify gaps.** Missing edge cases? Vague acceptance criteria? Unclear interfaces?
4. **Start a focused dialogue** — ask only about the gaps, not the whole spec.
5. **Apply fixes** after user confirmation.
6. **Re-validate** and report results.

---

## Notes

- Specs are living documents. Re-running `write` on an existing feature overwrites after confirmation.
- `validate` never modifies files — it only reads and reports.
- The spec-writer respects the project's directory structure. Specs for sub-projects or modules go under `specs/{module-name}/` if the project uses a multi-module layout.
- The dialogue is not a form to fill. Adapt questions based on answers. If the user gives detailed answers early, skip ahead. If they're vague, dig deeper.
