---
name: spec-writer-setup
description: "Scans a target project to extract idiomatic patterns (test framework, code style, architecture, conventions) and generates customized TDD agents that use the project's actual patterns instead of generic pseudocode. Use when the user wants to adapt the TDD agents to a specific codebase."
version: 1.0.0
user-invocable: true
argument-hint: "[scan|adapt|scan-and-adapt] [path-or-agent]"
---

# Context Adapter Skill

Scan a project to extract its idiomatic patterns, then generate customized TDD agents that speak the project's language instead of pseudocode. This bridges the gap between the generic Red/Green/Refactor agents and a real codebase's conventions.

## Core Principle

**The best TDD agent is one that looks like it was written for your project.** Generic agents say "adapt to your stack" — this skill does the adaptation automatically by reading what your project actually does.

## Setup

Before doing anything:

1. **Locate the target project.** If a path is given, use it. Otherwise, ask the user which project to scan.
2. **Read the base TDD agents.** Load `tdd-red`, `tdd-green`, and `tdd-refactor` from the `agents/` directory. These are the templates to customize.
3. **Check for prior context.** Look for an existing `context-report.md` in the target project. If found, ask the user whether to re-scan or use the existing report.

## Commands

| Command | Description |
|---------|-------------|
| `scan [path]` | Gather project context, produce a pattern report |
| `adapt [agent\|all]` | Generate customized TDD agent(s) from the last scan |
| `scan-and-adapt [path]` | Full pipeline: scan then adapt all three agents |

### Routing

- **No argument**: ask the user which command and which project path.
- **First word matches a command**: load and execute it.
- **`adapt` without a prior scan**: run `scan` first, then adapt.
- **`adapt red`** or **`adapt tdd-red`**: customize only the Red Phase agent.
- **`adapt all`**: customize all three agents.

---

## Phase 1 — Scan

The scan phase reads the target project and extracts a structured context report. This is the foundation for all customization.

### 1.1 Discovery

Scan the project root for these files, in order of priority:

**Documentation & Conventions** (read first — highest signal):
| File | What it reveals |
|------|----------------|
| `README.md` | Tech stack, setup commands, project purpose |
| `AGENTS.md`, `CLAUDE.md`, `COPILOT.md`, `.cursorrules`, `.windsurfrules`, `.clinerules` | Existing AI agent instructions and conventions |
| `CONTRIBUTING.md` | Code style, PR conventions, testing requirements |
| `docs/` directory | Architecture decisions, API docs, design docs |
| `specs/` directory | Existing specifications and their structure |

**Package Manifests** (language and dependencies):
| File | What it reveals |
|------|----------------|
| `package.json` | Node/JS/TS stack, scripts (test, lint, format), dependencies |
| `Cargo.toml` | Rust stack, dependencies, features |
| `pyproject.toml`, `setup.py`, `requirements.txt` | Python stack, dependencies |
| `go.mod` | Go stack, module path, dependencies |
| `Gemfile` | Ruby stack, dependencies |
| `pom.xml`, `build.gradle` | Java/Kotlin stack, dependencies |
| `*.csproj`, `*.sln` | .NET stack, dependencies |

**Test Configuration** (testing patterns):
| File/Directory | What it reveals |
|------|----------------|
| `jest.config.*`, `vitest.config.*` | JS/TS test framework config |
| `pytest.ini`, `conftest.py`, `tox.ini` | Python test framework config |
| `*_test.go`, `*_test.rs`, `*_spec.rb` | Language-specific test file conventions |
| `tests/`, `test/`, `__tests__/`, `spec/` | Test directory structure |
| `.github/workflows/` | CI test commands and matrix |

**Code Style** (formatting and linting):
| File | What it reveals |
|------|----------------|
| `.eslintrc.*`, `biome.json` | JS/TS linting rules |
| `rustfmt.toml`, `.rustfmt.toml` | Rust formatting config |
| `.prettierrc.*` | JS/TS formatting config |
| `ruff.toml`, `pyproject.toml [tool.ruff]` | Python linting/formatting |
| `.editorconfig` | Cross-editor style rules |

**MCP & Tooling** (external integrations):
| File | What it reveals |
|------|----------------|
| `.claude/mcp.json`, `mcp.json` | MCP server configurations |
| `.opencode/`, `opencode.json` | opencode agent/skill configs |
| `Makefile`, `justfile`, `Taskfile.yml` | Project task runners and commands |

### 1.2 Pattern Extraction

From the discovered files, extract these patterns. For each pattern, capture **what the project actually uses**, not what's generic.

#### Language & Runtime
- Primary language and version
- Runtime / compiler
- Module system (ESM, CJS, etc.)
- **Compilation model**: compiled (Rust, Go, Java, C#) vs interpreted/dynamic (Python, JS, Ruby). Compiled languages may require stub implementations for tests to even compile — see "Compilation Requirements" below.

#### Compilation Requirements

Some languages require that referenced functions/types exist before tests can compile. This affects the TDD red phase workflow — compiled languages may need stub implementations alongside tests.

When the project uses a compiled language, capture:
- The stub pattern used (e.g., `todo!()`, empty returns, `NotImplementedError`)
- Whether the project already uses stub-first TDD (look for `todo!()` or similar in source files)
- How stubs are organized (co-located with tests, in a stubs module, inline)

Dynamic/interpreted languages (Python, JS, Ruby) don't need this — tests fail at runtime when calling missing code, so the red agent writes tests only.

#### Test Framework
- Framework name and version (Jest, Vitest, pytest, Go testing, RSpec, etc.)
- Assertion library (assert, expect, chai, should, etc.)
- Test runner command (e.g., `npm test`, `cargo test`, `pytest`)
- Watch mode command if available

#### Test Structure
- Naming convention: `describe/it`, `test()`, `def test_`, `#[test]`, etc.
- File naming: `*.test.*`, `*_test.*`, `*.spec.*`, `test_*.*`
- Directory convention: co-located (`src/foo.test.ts`) vs separate (`tests/foo_test.py`)
- Setup/teardown patterns: `beforeEach`, `setUp`, `fixtures`, etc.
- Test grouping: `describe` blocks, test classes, test modules

#### Assertion Patterns
- How assertions look in the project's tests (read 2-3 actual test files)
- Common assertion idioms (e.g., `expect(x).toBe(y)` vs `assert x == y`)
- How errors/exceptions are asserted
- How async operations are tested

#### HTTP / API Testing
- Test client approach (supertest, httpx, reqwest, testcontainers, etc.)
- How requests are built
- How responses are asserted (status, body, headers)
- Mocking approach for external services

#### Data Layer Testing
- Database test strategy (in-memory, testcontainers, transactional rollback, etc.)
- Fixture/seed data approach
- ORM or query builder used
- Migration approach in tests

#### Error Handling
- Error type pattern (Result, exceptions, error enums, etc.)
- How errors are propagated
- Custom error types if any
- How errors map to HTTP status codes

#### Code Style
- Formatting tool and command
- Linting tool and command
- Import organization (sorted, grouped, etc.)
- Naming conventions (camelCase, snake_case, PascalCase for types, etc.)

#### Project Structure
- Source layout (src/, lib/, pkg/, app/, etc.)
- Module organization pattern
- Where types/models live
- Where routes/handlers live
- Where database access lives
- Where middleware lives

#### CI/CD
- CI test command
- Lint/format check in CI
- Any test coverage requirements

### 1.3 Context Report

Write the extracted patterns to `context-report.md` in the project root. This document is both a human-readable summary and the input for the adapt phase.

````markdown
# Context Report

Generated: YYYY-MM-DD
Project: <project-name>
Path: <project-path>

## Language & Runtime
- **Language**: Rust 1.78
- **Edition**: 2021
- **Module system**: crate-based
- **Compilation model**: compiled (strict type system — stubs required for tests to compile)

## Compilation Requirements
- **Stub pattern**: `todo!("function_name")` macros
- **Stub-first TDD**: yes — project already uses `todo!()` in early feature branches

## Test Framework
- **Framework**: built-in `#[test]` + `cargo test`
- **Assertion**: `assert!`, `assert_eq!`, custom `pretty_assertions`
- **Test command**: `cargo test`
- **Watch**: `cargo watch -x test`

## Test Structure
- **Naming**: `#[test]` attribute, `fn test_<feature>_<scenario>()`
- **Files**: co-located in `src/` with `#[cfg(test)] mod tests`
- **Setup**: `#[ctor]` for global setup, inline fixtures
- ...

## Assertion Patterns
```rust
// From src/auth/handler_test.rs
assert_eq!(response.status(), StatusCode::OK);
assert_eq!(response.json::<User>().name, "alice");
```

## HTTP Testing
- **Approach**: `axum::test` helper with `tower::ServiceExt`
- **Request building**: `Request::builder().method(POST).uri("/api/users").json(&body)`
- ...

## Data Layer Testing
- **Strategy**: in-memory SQLite with `sqlx::test`
- **Fixtures**: `#[sqlx::test(fixtures("users"))]`
- ...

## Error Handling
- **Pattern**: `Result<T, AppError>` with custom `AppError` enum
- **Propagation**: `?` operator throughout
- ...

## Code Style
- **Formatter**: `rustfmt` (default config)
- **Linter**: `clippy` with `-D warnings`
- **Naming**: snake_case functions, PascalCase types, SCREAMING_SNAKE_CASE constants

## Project Structure
```
src/
  main.rs           ← entry point
  lib.rs            ← library root
  config.rs         ← configuration
  error.rs          ← error types
  routes/           ← request handlers
  models/           ← data types
  db/               ← database queries
  middleware/        ← cross-cutting concerns
```

## CI/CD
- **Test**: `cargo test --all-features`
- **Lint**: `cargo clippy -- -D warnings`
- **Format**: `cargo fmt --check`

## Documented Conventions
<quoted snippets from README, CONTRIBUTING, AGENTS.md, etc.>

## MCP Servers
<any configured MCP servers and their purpose>

## Relevant Links
<any docs URLs, wiki links, or references found>
````

### 1.4 Scan Rules

- **Read actual test files.** Don't guess patterns from config alone — read 2-3 real test files to see how tests are actually written.
- **Quote, don't paraphrase.** When capturing conventions from docs, quote the original text.
- **Note conflicts.** If the project has multiple patterns (e.g., some tests use `describe/it`, others use `test()`), note both and flag the inconsistency.
- **Capture commands verbatim.** Copy test/lint/format commands exactly as they appear in package.json scripts, Makefile targets, etc.
- **Be thorough but focused.** Read enough files to be confident, but don't read every test in the project.

---

## Phase 2 — Adapt

The adapt phase takes the context report and applies **targeted edits** to the base TDD agents. It does not rewrite them — it copies the base agents to the target project, then surgically replaces only the parts that need to change: pseudocode blocks, generic commands, placeholder types. Everything else stays exactly as-is.

### 2.1 Core Principle

**Edit, don't generate.** The base agents contain well-structured workflow descriptions, rules, and guidelines that apply to any stack. The adapt phase only touches:

- Pseudocode example blocks → replaced with idiomatic project code
- Generic tool commands (`run_test_suite`, `run_linter`, `run_formatter`) → replaced with project commands
- Placeholder type definitions → replaced with project's actual types
- The "Adapt to your stack" disclaimers → replaced with a customization note

Everything else — prose, rules, checklists, handoff descriptions, workflow steps — is **copied verbatim** from the base agent.

### 2.2 Input

- The `context-report.md` generated by the scan phase (or previously saved).
- The base agent files from `agents/` (`tdd-red.agent.md`, `tdd-green.agent.md`, `tdd-refactor.agent.md`).

### 2.3 Process

For each agent being adapted:

1. **Copy the base agent** to the target project (`<target-project>/agents/tdd-<phase>.agent.md`).
2. **Identify edit targets** — find each pseudocode block, generic command, and placeholder type in the copied file.
3. **Apply edits** — replace only the identified targets with project-specific equivalents from the context report.
4. **Add the customization header** (see 2.5).
5. **Verify** — re-read the file and confirm only the intended edits were made; no surrounding prose was altered.

If the target project already has agent files, read them first. If they already contain project-specific code, ask before overwriting. If they appear to be unmodified copies of the base, proceed with the update.

### 2.4 Edit Catalog

These are the **only** edit operations allowed. Do not rewrite paragraphs, restructure sections, or rephrase guidelines.

#### Edit: Replace Pseudocode Example Blocks

Find fenced code blocks that contain pseudocode (no language tag, or tagged with a comment like `// Pseudocode pattern`). Replace the block content with idiomatic code from the target project.

**Before** (generic):
```
test get_user_returns_success:
    Arrange:
        seed_database(user_id, name="Alice")
        app = build_test_app()
    Act:
        response = app.get("/api/users/{user_id}")
    Assert:
        assert response.status == 200
        assert response.json()["name"] == "Alice"
```

**After** (project-specific, e.g., Rust + axum):
```rust
#[tokio::test]
async fn test_get_user_returns_success() {
    // Arrange
    let db = setup_test_db().await;
    seed_user(&db, "alice", "alice@example.com").await;
    let app = build_test_app(db);

    // Act
    let response = app
        .oneshot(Request::builder().uri("/api/users/1").body(Body::empty()).unwrap())
        .await
        .unwrap();

    // Assert
    assert_eq!(response.status(), StatusCode::OK);
    let user: User = serde_json::from_slice(&hyper::body::to_bytes(response.into_body()).await.unwrap()).unwrap();
    assert_eq!(user.name, "alice");
}
```

If the context report has no matching pattern for a pseudocode block, **leave the block as-is** and add a comment at the top: `<!-- TODO: replace with project-specific example -->`. Never invent code.

#### Edit: Replace Generic Tool Commands

Find occurrences of `run_test_suite`, `run_linter`, `run_formatter`, and similar generic commands. Replace with the project's actual commands from the context report. These appear in code blocks and in inline prose.

| Generic | Replace with (example) |
|---------|----------------------|
| `run_test_suite` | `cargo test --all-features` |
| `run_linter --warnings-as-errors` | `cargo clippy -- -D warnings` |
| `run_formatter` | `cargo fmt` |
| `run_dependency_audit` | `cargo audit` |

Only replace commands that the context report provides equivalents for. If no equivalent was found, leave the generic command and add `<!-- TODO: replace with project command -->`.

#### Edit: Replace Placeholder Type Definitions

Find pseudocode type definitions (e.g., `type User:`, `type AppError:`). Replace with the project's idiomatic type definition from the context report.

**Before**:
```
type AppError:
    variant NotFound
    variant BadRequest(message: String)
    variant Conflict(message: String)
    variant Database(underlying: Error)
```

**After** (Rust with thiserror):
```rust
#[derive(Debug, thiserror::Error)]
pub enum AppError {
    #[error("not found")]
    NotFound,
    #[error("bad request: {0}")]
    BadRequest(String),
    #[error(transparent)]
    Database(#[from] sqlx::Error),
}
```

#### Edit: Replace Naming Convention Examples

Find test name examples that use a different convention than the project's. Replace the casing/prefix style to match.

**Before**: `create_user_missing_name_returns_400`
**After** (if project uses `test_` prefix): `test_create_user_missing_name_returns_400`

Only change the naming style — do not change the example's semantic meaning.

#### Edit: Replace Project Structure References

Find directory paths in prose or code that don't match the project's layout. Replace with the actual paths from the context report.

**Before**: `src/routes/`, `src/models/`, `src/db/`
**After**: whatever the project actually uses (e.g., `app/api/`, `lib/models/`, `internal/handler/`)

#### Edit: Replace the "Adapt" Disclaimer

Find the blockquote that says:
> **Adapt to your stack.** This agent is language-agnostic. Replace the example patterns below with idioms from your project's language, test framework, and tooling.

Replace with:
> **Customized for \<project-name\>.** This agent was adapted to your project's patterns on YYYY-MM-DD. Verify the examples match your current conventions — re-run `spec-writer-setup scan-and-adapt` after significant changes.

#### Edit: Add Language-Specific Workflow Steps

For compiled languages where tests can't compile without matching implementations, the red phase agent needs an extra step: create stubs before (or alongside) writing tests. This edit **inserts** a workflow step into the agent — it does not replace existing steps.

**When to apply:** Only when the context report's "Compilation Requirements" section indicates a compiled language that requires stubs for tests to compile.

**Where to insert:** In the red phase agent, between "Read the Spec" and "Test Strategy". In the green phase agent, no change needed — stubs are already in place from red.

**What to insert** (adapt the stub pattern to the project's language):

Example for Rust (`todo!()` pattern):
````markdown
## 2. Create Stubs

Before writing tests, create minimal stub implementations for every function,
type, and trait the tests will reference. Use `todo!()` so the code compiles
but panics at runtime — tests will fail as expected.

```rust
pub fn create_user(db: &Db, input: CreateUser) -> Result<User, AppError> {
    todo!("implement create_user")
}
```

**Rules for stubs:**
- Match the exact function signatures the tests expect.
- Use `todo!("function_name")` with the function name as the message.
- Place stubs in the same module/file where the implementation will live.
- Do not add logic — just `todo!()`.
````

For other languages, adapt to whatever the idiomatic "not yet implemented" pattern is (empty returns, `NotImplementedError`, etc.). Derive it from the context report — look at how the project handles unimplemented code or what the language's convention is for compile-safe placeholders.

**Important:** This edit only applies to the red phase agent. The green phase agent already assumes stubs exist and builds on them. The refactor phase agent is unaffected.

### 2.5 Agent Header

Insert a header at the top of each adapted agent (after the frontmatter, before the title):

```markdown
> **Customized for <project-name>.** Adapted by `spec-writer-setup` on YYYY-MM-DD.
> Only code examples, commands, and type definitions were changed. Workflow,
> rules, and guidelines are preserved from the base agent.
> Context report: `context-report.md`
```

### 2.6 Output Files

Write adapted agents to the target project:

```
<target-project>/
├── agents/
│   ├── tdd-red.agent.md       ← adapted Red Phase
│   ├── tdd-green.agent.md     ← adapted Green Phase
│   └── tdd-refactor.agent.md  ← adapted Refactor Phase
├── context-report.md          ← the scan output
```

### 2.7 Adapt Rules

- **Edit in place, don't rewrite.** For each agent, copy the base file and apply specific edits. Do not regenerate the file from memory or paraphrase sections.
- **One edit target at a time.** Identify a specific block or command, replace it, move to the next. Don't batch multiple sections into one large replacement.
- **Never invent patterns.** If the context report doesn't capture a pattern, leave the original with a `<!-- TODO -->` marker.
- **Preserve the agent's role.** Red still only writes tests. Green still only writes implementation. Refactor still never modifies tests. The customization is about HOW, not WHAT.
- **Preserve all prose.** Workflow descriptions, rules, checklists, handoff steps, guidelines — these are the agent's brain. Do not rephrase, restructure, or summarize them.
- **Insertions are edits too.** Adding a stub-creation step to the red phase for compiled languages is an allowed edit. The inserted step must follow the same style and structure as existing steps in the agent.
- **Stubs belong in red, not green.** The stub step goes in the red phase agent. Green replaces stubs with real implementations.
- **Include the context report path** in each agent's header so future runs know where to find it.
- **Diff mentally before writing.** Before finalizing each agent, mentally compare it to the base. The only differences should be the edits listed in 2.4 and the header in 2.5.

---

## Full Pipeline — `scan-and-adapt`

When the user runs `scan-and-adapt [path]`:

1. Run the full scan phase.
2. Present the context report summary to the user.
3. Ask: "Does this look right? Anything to adjust?"
4. After confirmation, run adapt for all three agents.
5. For each agent, report what was edited: which blocks were replaced, which commands were updated, which types were changed.
6. If any edits were skipped (no matching pattern found), list the `<!-- TODO -->` markers left behind.

---

## Validation

After adapting agents, verify the edits were applied correctly:

1. **Only intended edits were made.** Compare each adapted agent to its base — the only differences should be the edits from the catalog (2.4) and the header (2.5). No prose should be rephrased or restructured.
2. **All pseudocode blocks were replaced** or marked with `<!-- TODO -->`.
3. **All commands reference real tools** from the project (not generic placeholders), or are marked with `<!-- TODO -->`.
4. **Agent roles are preserved** (Red = tests only, Green = implementation only, Refactor = no test changes).

---

## Notes

- The context report is a living document. Re-run `scan` after major project changes (new framework, new test tooling, etc.).
- The skill works best when the project has existing tests to read. For greenfield projects with no tests yet, the adaptation will be lighter (based on config and docs only) — most pseudocode blocks will get `<!-- TODO -->` markers.
- If the project uses multiple languages (e.g., TypeScript frontend + Rust backend), scan each separately and produce language-specific agent copies.
- MCP configurations found during the scan are noted in the context report but not directly used in agent customization — they may inform future extensions.
- The base agents in `agents/` are never modified. Adapted copies go to the target project. This means you can adapt the same base agents for multiple projects.
