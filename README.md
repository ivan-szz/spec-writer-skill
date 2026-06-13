# Spec Writer Skill

Spec-driven development toolkit for AI coding agents. Write structured specifications, then implement them through a disciplined TDD workflow.

Works with any AI agent that supports skills or custom instructions — opencode, Cursor, Claude Code, Copilot, Windsurf, Aider, and others.

## What's Included

```
spec-writer/
├── spec-writer/
│   └── SKILL.md           # Spec authoring skill — interactive dialogue, validation rules, antipattern detection
├── spec-writer-setup/
│   └── SKILL.md           # Context adapter — scans projects and customizes TDD agents to idiomatic patterns
├── agents/
│   ├── tdd-red.agent.md   # Red phase — writes failing tests from specs
│   ├── tdd-green.agent.md # Green phase — minimum implementation to pass tests
│   └── tdd-refactor.agent.md # Refactor phase — code quality, security, idioms
└── spec-template.md       # Canonical spec template
```

### Spec Writer (`spec-writer/SKILL.md`)

An interactive skill that guides you through authoring specs via dialogue. It doesn't generate specs from one-liners — it asks, probes, and clarifies first.

**Commands:**

| Command | Description |
|---------|-------------|
| `write <feature>` | Start a dialogue to create a new spec |
| `refine <spec-path>` | Review and improve an existing spec |
| `validate [spec-path]` | Validate specs against structural, security, and testability rules |

**Validation rules include:** minimum acceptance criteria, error contracts per interface, unique requirement IDs, security checks for auth specs, edge case coverage, test traceability.

### TDD Agents (`agents/`)

Three language-agnostic agents that implement the Red → Green → Refactor cycle:

| Agent | Role | Writes Code? | Writes Tests? |
|-------|------|:---:|:---:|
| `tdd-red` | Reads specs, writes failing tests | No | Yes |
| `tdd-green` | Implements minimum code to pass tests | Yes | No |
| `tdd-refactor` | Improves quality, security, idioms | Yes | No |

Each agent is self-contained — hand off between them as you progress through the TDD cycle.

### Spec Writer Setup (`spec-writer-setup/SKILL.md`)

Scans a target project to extract its idiomatic patterns — test framework, assertion style, code structure, naming conventions, tooling commands, stub conventions for compiled languages — then surgically adapts the generic TDD agents: replacing pseudocode examples, tool commands, and type definitions with project-specific equivalents. Everything else (workflow, rules, guidelines) is preserved as-is.

**Commands:**

| Command | Description |
|---------|-------------|
| `scan [path]` | Analyze a project and produce a `context-report.md` |
| `adapt [agent\|all]` | Apply targeted edits to TDD agent(s) using the context report |
| `scan-and-adapt [path]` | Full pipeline: scan then adapt all three agents |

The scan reads READMEs, agent files, package manifests, test configs, linter/formatter configs, existing test files, MCP configurations, and documentation. The adapt phase copies the base agents to the target project, then edits only the code examples, commands, and type definitions — prose, rules, and workflow stay untouched. For compiled languages, it can also insert a stub-creation step into the red phase agent.

## Installation

This is a collection of skill and agent definition files. Install them into whichever AI tool you use.

## Usage

### 1. Write a Spec

Ask your AI agent to write a spec for a feature. The skill will start a dialogue:

```
> write user registration

# The agent will ask:
# - What does it do? (one sentence)
# - Who uses it?
# - What's the primary interface?
# - What's out of scope?
# - What can go wrong?
# ... then produce a structured .spec.md file
```

The spec-writer skill is designed with TDD in mind, but specs are useful on their own. Even if you don't follow a strict red-green-refactor cycle. The `write`, `refine`, and `validate` commands work independently of the TDD agents.

### 2. Implement with TDD

Once a spec exists, use the TDD agents in sequence:

1. **Red** — `tdd-red` reads the spec and writes failing tests
2. **Green** — `tdd-green` implements the minimum code to pass those tests
3. **Refactor** — `tdd-refactor` cleans up without breaking tests

### 3. Validate

```
> validate specs/user-register.spec.md
```

Produces a pass/fail report checking structural integrity, security requirements, edge case coverage, and test traceability.

### 4. Adapt TDD Agents to Your Project

Scan your project to extract its patterns, then adapt the TDD agents:

```
> scan-and-adapt /path/to/your/project

# The agent will:
# - Read your README, test configs, package manifests
# - Find and parse existing test files
# - Extract test framework, assertion style, naming conventions
# - Generate context-report.md with all findings
# - Copy the base TDD agents and apply targeted edits:
#   replace pseudocode with idiomatic code, swap generic
#   commands for real ones, add stub steps for compiled languages
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

Required sections: Purpose & Scope, Definitions, Requirements, Constraints, Security, Interfaces & Data Contracts, Acceptance Criteria, Test Strategy, Edge Cases & Error Scenarios, Rationale.

Acceptance criteria use Given-When-Then format:

```markdown
### AC-1
**Given** a valid name and email
**When** POST /api/users is called
**Then** a 201 response with the created user is returned
```

## Adapting to Your Stack

The skill and agents are language-agnostic. They use pseudocode in examples — replace with your project's idioms:

- **Test framework**: Use whatever your stack provides (Jest, pytest, Go testing, RSpec, etc.)
- **Data layer**: ORM, raw SQL, or whatever your project uses
- **HTTP testing**: Your framework's test client or supertest equivalents
- **Linting/formatting**: Your project's existing tools

The `spec-writer-setup` skill automates this — run `scan-and-adapt` on your project to adapt the agents without manual editing.

## Philosophy

- **Specs before code.** Every feature starts with a written specification.
- **Ambiguity is the enemy.** If something can be interpreted two ways, the spec isn't done.
- **Testability is the test.** If you can't write a test for a requirement, rewrite the requirement.
- **Small is better.** One spec per focused feature.
- **Specs are living documents.** Update them as you learn during implementation.
