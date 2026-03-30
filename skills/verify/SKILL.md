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

1. Extract the execution hint from the HTML comment (`<!-- verify: ... -->`)
2. Announce what you're checking: "Checking: <description>"
3. Execute the instruction using the appropriate tool (Bash for commands, Grep for file searches, Read for file inspection, etc.)
4. Evaluate the result:
   - **Pass:** The command output confirms the check (e.g., expected HTTP status, no grep matches for forbidden patterns, tests pass)
   - **Fail:** The command output contradicts the check. Record the evidence (command + output)
   - **Inconclusive:** The command errored or the output is ambiguous. Treat as fail, note the ambiguity

**Short-circuit rule:** If a check in Contextual Verification named something like "Existing tests still pass" fails, announce the failure and ask the user whether to continue with remaining checks or stop. The remaining results may be unreliable if foundational checks fail.

## Step 3: Collect Manual Checks

For each `[manual]` check:

1. Present the check description and any guidance text
2. Ask the user: "Does this pass? (yes/no) Add a note if you'd like."
3. Record their response

If the user wants to skip a manual check, mark it as fail with note "Skipped by user".

## Step 4: Produce Report

Read the report format reference: `skills/verify/report-format.md`

1. Compile all results into the report format
2. Calculate the verdict: `pass` if all checks passed, `fail` otherwise
3. Write to `.plumbline/contracts/<name>-report.md`
4. Update the contract's frontmatter `status` field:
   - All checks passed → `status: verified`
   - Any check failed → `status: failed`

## Step 5: Present Results

Show a summary:

**If all passed:**
> "All checks passed. Contract verified. Report saved to `.plumbline/contracts/<name>-report.md`."

**If any failed:**
> "Verification failed: <N> checks did not pass. Here's what needs attention:"
> 1. [Ordered list of failures with brief evidence]
>
> "Report saved to `.plumbline/contracts/<name>-report.md`. Fix the issues and run `/plumbline:verify` again."
