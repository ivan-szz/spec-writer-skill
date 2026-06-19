# Spec Writer Skill

Spec-driven development toolkit for AI coding agents. Two flavors: write specs and verify them your way, or go full TDD with red-green-refactor agents.

Works with any AI agent that supports skills or custom instructions — opencode, Cursor, Claude Code, Copilot, Windsurf, Aider, and others.

> **Note:** The `SKILL.md` and agent `.md` frontmatter is opencode-compatible. Skills use `name`, `description`, `metadata`. Agents use `description`, `mode`, `name`. For other harnesses (Cursor, Copilot, Windsurf, etc.), check your tool's docs and adjust the frontmatter — the content itself is harness-agnostic.

## What's Included

```
spec-writer-skill/
├── spec-driven/                      # Spec-first workflow (no TDD required)
│   ├── skills/spec-writer/
│   │   └── SKILL.md                  # Interactive dialogue, validation, antipattern detection
│   └── specs/
│       └── spec-template.md          # Template with Verification section
│
├── test-driven/                      # Full TDD workflow
│   ├── skills/
│   │   ├── spec-writer/
│   │   │   └── SKILL.md              # Same dialogue, with Test Strategy validation
│   │   └── spec-writer-setup/
│   │       └── SKILL.md              # Scans projects, adapts TDD agents to idiomatic patterns
│   ├── agents/
│   │   ├── tdd-red.agent.md          # Red phase — writes failing tests from specs
│   │   ├── tdd-green.agent.md        # Green phase — minimum implementation to pass tests
│   │   └── tdd-refactor.agent.md     # Refactor phase — code quality, security, idioms
│   └── specs/
│       └── spec-template.md          # Template with Test Strategy section
```

## Which Variant to Use

| | `spec-driven` | `test-driven` |
|---|---|---|
| **Spec authoring** | Interactive dialogue, probing, validation | Same |
| **Verification section** | AC → method (unit, manual, demo, etc.) | AC → test (unit, integration, e2e) |
| **Validation rules** | Structural, security, edge cases, verification traceability | All of spec-driven + test traceability |
| **TDD agents** | No | Yes — red, green, refactor |
| **Project adapter** | No | Yes — `spec-writer-setup` scans and adapts agents |

Use `spec-driven` when you want structured specs without mandating a testing methodology. Use `test-driven` when you want the full spec → test → implement pipeline.

## Spec Writer

Both variants share the same core skill: an interactive dialogue that asks, probes, and clarifies before writing anything. No one-liner → spec generation.

**Commands:**

| Command | Description |
|---------|-------------|
| `write <feature>` | Start a dialogue to create a new spec |
| `refine <spec-path>` | Review and improve an existing spec |
| `validate [spec-path]` | Validate specs against the rule set |

### Validation Rules

**spec-driven** validates:

- Structural: minimum 3 ACs, interfaces with success + error, unique requirement IDs
- Security: token lifecycle for auth specs, explicit input validation
- Edge cases: HTTP status codes, concurrent access scenarios
- Verification: every AC maps to ≥1 verification entry

**test-driven** adds:

- Test traceability: every AC maps to ≥1 test, test level specified (unit/integration/e2e)

## TDD Agents (test-driven only)

Three language-agnostic agents that implement the Red → Green → Refactor cycle:

| Agent | Role | Writes Code? | Writes Tests? |
|-------|------|:---:|:---:|
| `tdd-red` | Reads specs, writes failing tests | No | Yes |
| `tdd-green` | Implements minimum code to pass tests | Yes | No |
| `tdd-refactor` | Improves quality, security, idioms | Yes | No |

Each agent is self-contained — hand off between them as you progress through the TDD cycle.

### Adapt TDD Agents to Your Project

The `spec-writer-setup` skill scans a target project to extract its idiomatic patterns — test framework, assertion style, code structure, naming conventions, tooling commands — then adapts the generic TDD agents.

```
> scan-and-adapt /path/to/your/project

# The agent will:
# - Read your README, test configs, package manifests
# - Extract test framework, assertion style, naming conventions
# - Generate context-report.md with all findings
# - Copy the base TDD agents and apply targeted edits
```

Or run the steps separately:

```
> scan /path/to/your/project   # produces context-report.md
> adapt all                     # applies edits to all three agents
```

## Spec Format

Each spec is a markdown file with YAML frontmatter:

```yaml
---
title: "User Registration"
version: "0.1.0"
status: "draft"  # draft | in-progress | review | approved | implemented
tags: [auth, user]
---
```

**spec-driven** required sections: Purpose & Scope, Definitions, Requirements, Constraints, Security, Interfaces & Data Contracts, Acceptance Criteria, **Verification**, Edge Cases & Error Scenarios, Rationale.

**test-driven** required sections: same but with **Test Strategy** instead of Verification.

Acceptance criteria use Given-When-Then format:

```markdown
### AC-1
**Given** a valid name and email
**When** POST /api/users is called
**Then** a 201 response with the created user is returned
```

## Installation

### opencode

Copy the skill folder(s) you need into your project's `.opencode/skills/` directory (or `~/.config/opencode/skills/` for global access). OpenCode also supports `.claude/skills/` and `.agents/skills/` paths.

**spec-driven** (no TDD):
```bash
mkdir -p .opencode/skills/spec-writer
cp spec-driven/skills/spec-writer/SKILL.md .opencode/skills/spec-writer/
mkdir -p specs
cp spec-driven/specs/spec-template.md specs/
```

**test-driven** (full TDD):
```bash
# Skills
mkdir -p .opencode/skills/spec-writer .opencode/skills/spec-writer-setup
cp test-driven/skills/spec-writer/SKILL.md .opencode/skills/spec-writer/
cp test-driven/skills/spec-writer-setup/SKILL.md .opencode/skills/spec-writer-setup/

# Agents
mkdir -p .opencode/agents
cp test-driven/agents/tdd-red.agent.md .opencode/agents/
cp test-driven/agents/tdd-green.agent.md .opencode/agents/
cp test-driven/agents/tdd-refactor.agent.md .opencode/agents/

# Template
mkdir -p specs
cp test-driven/specs/spec-template.md specs/
```

### Other tools

Copy the `SKILL.md` and `spec-template.md` files into whatever location your tool expects. You may need to adjust the YAML frontmatter to match your tool's schema.

> **Agent filenames:** Agent files go in `.opencode/agents/` (project) or `~/.config/opencode/agents/` (global). The filename becomes the agent name — e.g., `tdd-red.agent.md` creates a `tdd-red` agent. OpenCode also supports `.claude/agents/` and `.agents/` paths.

## Philosophy

- **Specs before code.** Every feature starts with a written specification.
- **Ambiguity is the enemy.** If something can be interpreted two ways, the spec isn't done.
- **Verify everything.** Every acceptance criterion must map to a verification method — test, manual check, demo, whatever fits.
- **Small is better.** One spec per focused feature.
- **Specs are living documents.** Update them as you learn during implementation.
