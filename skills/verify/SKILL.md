---
name: verify
description: Use when implementation is complete and you need to verify work against a Plumbline verification contract. Executes auto checks, collects manual check results, and produces a verification report.
---

# Plumbline: Verify

Verify completed work against a Plumbline verification contract. Executes automated checks, collects manual check results, and produces a structured verification report.

## Core Principle

A verification contract defines what "done" means. This skill determines whether the work meets that definition by executing every check and collecting evidence.

## When to Use

- After completing implementation of a task that has a Plumbline contract
- When you want to check progress against the contract mid-implementation
- When the user runs `/plumbline:verify`

## When NOT to Use

- Before implementation (use `plumbline:contract` instead)
- If no contract exists in `.plumbline/contracts/`

## Checklist

You MUST complete these steps in order:

1. **Select contract** — find and load the contract to verify
2. **Execute auto checks** — run each `[auto]` check and collect evidence
3. **Collect manual checks** — present `[manual]` checks to the user
4. **Produce report** — write the verification report
5. **Present results** — show summary and next steps

## Step 1: Select Contract

- Use Glob to find contracts: `.plumbline/contracts/*.md` (excluding `*-report.md`)
- Filter to contracts with `status: pending` or `status: failed`
- If the user specified a contract name (e.g., `/plumbline:verify add-user-auth`), use that
- If only one active contract exists, use it automatically
- If multiple active contracts exist, list them and ask the user which one to verify

Read the contract and parse:
- Frontmatter (task, sources, status)
- All checks grouped by section (Functional, Craft, Contextual)
- For each check: type (`[auto]` or `[manual]`), description, and execution hint if present

## Step 2: Execute Auto Checks

Read the contract format reference for check structure: `skills/contract/contract-format.md`

For each `[auto]` check:

1. Check for an execution hint (`<!-- verify: ... -->`). If present, use it as guidance. If absent, determine the verification approach from the criterion itself and available tools.
2. Announce what you're checking: "Checking: <description>"
3. Translate the execution hint into concrete tool invocations and execute them. Auto checks can use any available tool: shell commands, file reading, Grep, Glob, WebSearch, or any MCP tool. Hints prefixed with `web-verify:` should use WebSearch to validate facts against external sources. Every conclusion must be backed by tool output — never infer results from reading or perception alone.
4. Evaluate the result:
   - **Pass:** The tool output confirms the check (e.g., expected HTTP status, no grep matches for forbidden patterns, tests pass)
   - **Fail:** The tool output contradicts the check. Record the evidence (tool + output)
   - **Skipped:** The check cannot be executed in this environment (e.g., required tool not installed, service unavailable). Record the reason. A skipped check does not count as a failure but is tracked separately in the report.
   - **Inconclusive:** The tool errored or the output is ambiguous. Treat as fail, note the ambiguity

### Analytical auto checks

Some auto checks verify structural or positional properties of content (e.g., "X appears after the first third", "both files have equivalent structure", "the document explains concept Y, not just mentions it"). These are still `[auto]` — but you MUST verify them with tools, not perception:

- **Position/proportion:** Use `wc -w` for total word count, `grep -n` for line position, `awk` to calculate ratios. Never estimate position by reading.
- **Counting:** Use `wc`, `grep -c`, or equivalent. Never count by reading.
- **Structural comparison:** Use `grep` to extract headers from both files, then compare. Never rely on memory of what you read.
- **Content presence vs. mention:** Use `grep -A` to extract the surrounding context of a term, then verify the context constitutes an explanation (defines or describes), not just a reference. This is the one case where reading the grep output to evaluate is acceptable — but the extraction itself must use a tool.

**The rule:** if you can measure it, measure it. Tool output is evidence. Agent perception is opinion.

**Short-circuit rule:** If a check in Contextual Verification named something like "Existing tests still pass" fails, announce the failure and ask the user whether to continue with remaining checks or stop. The remaining results may be unreliable if foundational checks fail.

## Step 3: Collect Manual Checks

Present the first `[manual]` check one-by-one:

1. Present the check description
2. If the check has an inline rubric (`<!-- rubric: ... -->`), display the rubric levels and threshold to the user
3. Ask the user to score the check: "Score this 1-4 (threshold: <N>). Add a note if you'd like."
4. Record the score and note. The check passes if the score meets or exceeds the threshold.

If the check has no rubric, fall back to: "Does this pass? (yes/no) Add a note if you'd like."

**Batch mode:** After presenting the first manual check, if there are remaining manual checks, offer batch mode:
> "There are <N> more manual checks. Want to score them in batch? I'll list them all with their rubrics, and you can respond like: `2:3, 3:4, 4:2 "note"`. Or we can continue one-by-one."

If the user chooses batch mode, present all remaining manual checks with their rubrics in a single message and parse the batch response. If one-by-one, continue the sequential flow.

If the user wants to skip a manual check, mark it as fail with note "Skipped by user".

## Step 4: Produce Report

Read and follow the report format exactly as defined in `skills/verify/report-format.md`. The report MUST include all YAML frontmatter fields and the section structure specified there.

1. Compile all results into the report format
2. Calculate the verdict: `pass` if all checks passed, `fail` otherwise
3. Write to `.plumbline/contracts/<name>-report.md`
4. Update the contract's frontmatter:
   - Set `status`: `verified` (all passed) or `failed` (any failed)
   - Add `report: <name>-report.md` linking to the generated report

## Step 5: Present Results

Show a summary:

**If all passed:**
> "All checks passed. Contract verified. Report saved to `.plumbline/contracts/<name>-report.md`."

**If any failed:**
> "Verification failed: <N> checks did not pass. Here's what needs attention:"
> 1. [Ordered list of failures with brief evidence]
>
> "Report saved to `.plumbline/contracts/<name>-report.md`. Fix the issues and run `/plumbline:verify` again."
