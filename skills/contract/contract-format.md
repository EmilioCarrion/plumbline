# Verification Contract Format

This document defines the structure of a Plumbline verification contract. Both `plumbline:contract` (producer) and `plumbline:verify` (consumer) use this format.

## File Location

Contracts are stored at `<project-root>/.plumbline/contracts/<name>.md` where `<name>` is a kebab-case slug derived from the task description.

## Structure

### YAML Frontmatter

```yaml
---
task: "Short description of the task"
sources:
  - type: issue          # issue | codebase | docs | conversation
    ref: "PROJ-123"      # reference identifier (issue key, file path, etc.)
  - type: codebase
    paths: ["src/auth/"]
  - type: docs
    paths: ["CLAUDE.md"]
created: YYYY-MM-DD
status: pending          # pending | verified | failed
---
```

**Fields:**
- `task` (required): One-line description of what's being built or done
- `sources` (required): List of information sources consumed during generation. Each has a `type` and either `ref` (for issues) or `paths` (for codebase/docs)
- `created` (required): Date the contract was generated
- `status` (required): Current lifecycle state

### Markdown Body

```markdown
# Verification Contract: <task description>

## Context
Summary of what's being built and why. Includes relevant constraints,
dependencies, and decisions derived from sources. This section gives
the verifier (human or agent) enough context to understand the checks.

## Functional Verification
Checks that verify the task does what it's supposed to do.

- [ ] `[auto]` Description of an automatically verifiable check
  <!-- verify: command or instruction the agent host should execute -->
- [ ] `[manual]` Description of a check requiring human judgment

## Craft Verification
Checks that verify quality, maintainability, and design.

- [ ] `[auto]` Description
  <!-- verify: command -->
- [ ] `[manual]` Description

## Contextual Verification
Checks that verify the change works in the real system context.

- [ ] `[auto]` Description
  <!-- verify: command -->
- [ ] `[manual]` Description
```

## Check Format

Each check is a Markdown checkbox list item with a type tag:

- **`[auto]`** — Can be verified by an agent: executing a command, reading and analyzing content, comparing documents, checking structure, or any other deterministic evaluation. The execution hint is an HTML comment on the next line: `<!-- verify: <instruction> -->`. The instruction can be a shell command, an API call, a file inspection, or an analytical instruction the agent can follow (e.g., "read both files and verify equivalent section structure").
- **`[manual]`** — Requires human aesthetic or subjective judgment that an agent cannot reliably evaluate (tone, rhetorical impact, memorability). Must include an inline rubric as an HTML comment with a 1-4 scale and acceptance threshold.

### Manual Check Rubric Format

Every `[manual]` check includes an inline rubric:

```markdown
- [ ] `[manual]` Description of check
  <!-- rubric:
  4: <excellent — specific, observable description>
  3: <good — specific, observable description>
  2: <below bar — specific, observable description>
  1: <unacceptable — specific, observable description>
  threshold: 3
  -->
```

- Each level describes observable characteristics, not feelings
- Levels must be concrete and distinguishable from each other
- The threshold is the minimum score to pass (typically 3)
- During verification, the user scores the check against the rubric and optionally adds a note

## Naming Convention

Contract filenames are kebab-case slugs: `add-user-auth.md`, `fix-payment-race.md`, `migrate-database-schema.md`. The corresponding report uses the suffix `-report.md`: `add-user-auth-report.md`.

## Status Lifecycle

- `pending` — Contract generated, not yet verified
- `verified` — All checks passed (auto and manual)
- `failed` — At least one check failed during verification
- A `failed` contract returns to `pending` when the user re-runs verification after fixes
