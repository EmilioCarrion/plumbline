# Verification Report Format

This document defines the structure of a Plumbline verification report, produced by `plumbline:verify`.

## File Location

Reports are stored alongside their contract at `<project-root>/.plumbline/contracts/<name>-report.md`.

## Structure

### YAML Frontmatter

```yaml
---
contract: <name>.md
verified: YYYY-MM-DD
verdict: pass | fail
auto_passed: N
auto_total: M
manual_passed: N
manual_total: M
---
```

**Fields:**
- `contract` (required): Filename of the contract this report verifies
- `verified` (required): Date verification was run
- `verdict` (required): `pass` if all checks passed, `fail` if any check failed
- `auto_passed` / `auto_total` (required): Count of auto checks passed vs total
- `manual_passed` / `manual_total` (required): Count of manual checks passed vs total

### Markdown Body

```markdown
# Verification Report: <task description>

## Summary
<N> auto checks passed out of <M>. <N> manual checks passed out of <M>.

## Passed Checks
- [x] `[auto]` Description of passing check
- [x] `[manual]` Description of passing check

## Failed Checks

### [auto] Description of failing check
**Evidence:** What happened when the check was executed.
**Command:** The command that was run (for auto checks).

### [manual] Description of failing check
**Score:** <N>/4 (threshold: <T>)
**Note:** The user's explanation of why it failed.

## Next Steps
Ordered list of what needs to be fixed, prioritized by:
1. Functional failures (broken behavior)
2. Contextual failures (system integration issues)
3. Craft failures (quality issues)
```

## Verdict Logic

- `pass` — Every check (auto and manual) passed
- `fail` — At least one check failed or was inconclusive

There is no partial pass. The contract either passes or it doesn't.
