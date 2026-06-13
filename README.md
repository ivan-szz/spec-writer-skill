# Spec-Driven Development

A methodology for building reliable software with specifications as the source of truth.

## What Is Spec-Driven Development?

Spec-driven development (SDD) is a workflow where **every feature starts with a written specification**, not code. The spec defines what to build, how it should behave, and how it should be tested — before any implementation begins.

This is not a replacement for TDD. It is an **upstream layer** on top of TDD that answers the question: *"What tests should I write?"*

```
Traditional TDD:    Idea → Tests → Code → Refactor
Spec-Driven TDD:    Spec → Tests → Code → Refactor
```

The spec is the contract. Code implements the contract. Tests verify the contract.

## How It Works

### The Workflow

```
┌─────────┐     ┌──────────┐     ┌──────────┐     ┌─────────────┐     ┌─────────┐
│  Write   │────▶│ Validate │────▶│Test (RED)│────▶│Implement    │────▶│ Refactor │
│  Spec    │     │  Spec    │     │          │     │   (GREEN)   │     │          │
└─────────┘     └──────────┘     └──────────┘     └─────────────┘     └─────────┘
```

1. **Write Spec** — Create a `.spec.md` file describing the feature. Include API/interface design, data models, validation rules, edge cases, and acceptance criteria.
2. **Validate Spec** — Review for completeness. Every requirement should be testable. Every edge case should have expected behavior.
3. **Test (RED)** — Write failing tests that implement each acceptance criterion. Tests come first.
4. **Implement (GREEN)** — Write the minimum code to make all tests pass.
5. **Refactor** — Clean up code while keeping tests green.

### Why Specs Before Tests?

- **Tests answer "does it work?"** — Specs answer "what should it do?"
- **Specs catch design gaps** before you've written a single line of code.
- **Specs are readable by everyone** — PMs, designers, and other devs can review them.
- **Specs survive refactoring** — When code changes, the spec remains the reference.
- **Specs reduce scope creep** — If it's not in the spec, it doesn't get built (yet).

## Project Structure

```
spec-driven/
├── AGENTS.md                    # Agent instructions and project standards
├── README.md                    # This file
├── templates/
│   └── spec-template.md         # Template for creating new specs
├── examples/
│   └── auth-register.spec.md    # Example: complete, approved spec
└── specs/                       # Your feature specs live here
    ├── auth-register.spec.md
    ├── auth-login.spec.md
    └── ...
```

### Spec Files

Specs live in `specs/` and use the naming convention: `feature-name.spec.md`

Each spec has:
- **YAML frontmatter** — metadata (title, version, status, tags)
- **API / Interface Design** — endpoints, function signatures, request/response formats
- **Data Model** — database schema changes or data structure definitions
- **Business Rules** — domain logic
- **Validation Rules** — input constraints
- **Edge Cases** — unusual situations and expected behavior
- **Acceptance Criteria** — testable, checkbox items
- **Test Strategy** — what to test and how
- **Security Considerations** — security-specific requirements

### Spec Statuses

| Status | Meaning |
|--------|---------|
| `draft` | Initial version, not yet reviewed |
| `in-progress` | Being actively implemented |
| `review` | Implementation complete, under review |
| `approved` | Spec is finalized and approved |
| `implemented` | Spec is fully implemented and tested |

## Using This With TDD

### Step 1: Write (or Select) a Spec

```bash
# Create a new spec from template
cp templates/spec-template.md specs/my-feature.spec.md

# Edit it to describe your feature
```

### Step 2: Write Tests from Acceptance Criteria

Each acceptance criterion (AC-001, AC-002, ...) should map to at least one test:

```
// AC-001: Given valid input, when registering, then 201 is returned
test("register_valid_input_returns_201", () => {
    // ...
});

// AC-003: Given duplicate email, when registering, then 409 is returned
test("register_duplicate_email_returns_409", () => {
    // ...
});
```

*Note: Use your project's testing framework and conventions. The key principle is that each acceptance criterion maps to at least one test.*

### Step 3: Implement to Make Tests Pass

Write the minimum code needed. Don't over-engineer. Let the spec guide scope.

### Step 4: Refactor and Verify

Run your linter, formatter, and full test suite. Clean up while keeping all tests green.

## For AI Agents

If you are an AI agent working on this project, read `AGENTS.md` first. It contains:

- Project technology stack and conventions
- Coding standards and naming rules
- Testing requirements and frameworks
- Security requirements
- File structure expectations

### Key Rules for Agents

1. **No code without a spec.** If no spec exists, create one first.
2. **Read the spec before writing code.** Understand the full feature.
3. **Write tests that map to acceptance criteria.** Each AC → ≥1 test.
4. **Use the spec template.** Copy `templates/spec-template.md` for new specs.
5. **Mark spec status.** Update the status in frontmatter as you work.

## Creating a New Spec

```bash
# 1. Copy the template
cp templates/spec-template.md specs/your-feature.spec.md

# 2. Fill in all sections — remove placeholder comments

# 3. Set status to "draft" in frontmatter

# 4. Review with team / validate completeness

# 5. Set status to "approved" when ready

# 6. Start TDD cycle: write tests → implement → refactor
```

## Philosophy

- **Specs are living documents.** Update them as you learn during implementation.
- **Ambiguity is the enemy.** If a developer could interpret something two ways, the spec isn't specific enough.
- **Testability is the test.** If you can't write a test for an acceptance criterion, rewrite the criterion.
- **Small is better.** One spec per focused feature. Break large features into multiple specs.
- **Code implements the spec.** When in doubt, refer to the spec — not to what the code currently does.
