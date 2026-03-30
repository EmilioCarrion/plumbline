---
name: contract
description: Use when starting any task, feature, or bugfix to generate a verification contract BEFORE implementation. Defines explicit criteria for "done" across functional, craft, and contextual dimensions.
---

# Plumbline: Contract

Generate a verification contract that defines what "done" means for a task, before any implementation begins.

## Core Principle

Generating is easy. Verifying is the work. This skill makes verification criteria explicit upfront, so you know what to build toward and how to prove it's correct.

## When to Use

- Before implementing any feature, bugfix, or task
- When you need to define acceptance criteria beyond what's in the issue
- When you want to make implicit quality standards explicit

## Checklist

You MUST complete these steps in order:

1. **Gather context** — read project docs, analyze codebase, fetch issue if available
2. **Ask clarifying questions** — fill gaps in functional, craft, and contextual dimensions
3. **Generate contract** — produce the verification contract with auto and manual checks
4. **User reviews contract** — present for approval, allow edits
5. **Save contract** — write to `.plumbline/contracts/<name>.md`

## Step 1: Gather Context

Read the context gathering guide before proceeding: `skills/contract/context-gathering.md`

Collect information from available sources in this order:
1. Project docs (CLAUDE.md, ADRs, README)
2. Codebase (relevant files, test patterns, project structure)
3. Issue/ticket (if MCP tools available and issue referenced)
4. Domain research (web search for external constraints, when task involves real-world dependencies)

Announce what you found:
> "I've reviewed the project context. Here's what I found: [brief summary of tech stack, conventions, relevant code areas]. Now I have a few questions to fill in the gaps."

## Step 2: Ask Clarifying Questions

Based on gaps found in step 1, ask targeted questions — one at a time. When domain research surfaced external constraints, use them to ask more specific questions (e.g., "El Rastro closes at 15:00 on Sundays — should we schedule it in the morning?"). Cover all three dimensions:

- **Functional:** What should happen? What are the edge cases? What's the happy path?
- **Craft:** What quality standards apply? What patterns should be followed? What should be avoided?
- **Contextual:** What could break in the real system? What are the dependencies? What's the performance budget?

Minimum: 2 questions. Maximum: 5. Stop when dimensions are sufficiently covered.

## Step 3: Generate Contract

Read the contract format reference: `skills/contract/contract-format.md`

For each check, decide if it's `[auto]` or `[manual]`:
- **`[auto]`** if the check can be verified by an agent using tools — this includes running commands, grepping files, running tests, but also measuring structural or positional properties of content (e.g., word counts, position of first mention, header comparison between files). The key: the verification must produce tool output as evidence, not rely on the agent's perception. If an agent can measure it and deterministically answer yes/no from the measurement, it's `[auto]`. Include the execution hint as an HTML comment — for analytical checks, the hint must describe the tool-based measurement (e.g., "use wc -w for total count, grep -n to find first mention, calculate ratio"), not just the question to answer.
- **`[manual]`** if the check requires human aesthetic or subjective judgment that an agent cannot reliably evaluate (e.g., "the tone feels right", "the closing is rhetorically powerful", "the phrasing is memorable"). The test: could two reasonable people disagree on whether it passes? If yes, it's `[manual]`.

**Bias toward `[auto]`.** The verifying agent can read, compare, count, search, and analyze structure — not just run shell commands. Reserve `[manual]` for checks where the answer is genuinely a matter of taste or perception, not analysis.

For each `[manual]` check, generate an inline rubric that makes the subjective judgment evaluable. The rubric uses a 1-4 scale with an acceptance threshold:

```markdown
- [ ] `[manual]` Description of check
  <!-- rubric:
  4: <excellent — specific description>
  3: <good — specific description>
  2: <below bar — specific description>
  1: <unacceptable — specific description>
  threshold: 3
  -->
```

Rubric guidelines:
- Each level must be concrete and distinguishable — avoid vague qualifiers like "good" vs "very good"
- The threshold is the minimum score to pass (typically 3)
- Levels should describe observable characteristics, not feelings (e.g., "reader can follow the argument without referencing external material" not "feels clear")

Guidelines for good checks:
- Each check is independently verifiable — no check depends on another passing first
- Checks are specific, not vague: "POST /auth/login returns 401 with invalid credentials" not "authentication works"
- Auto execution hints are complete commands that an agent can run without modification
- Manual checks include inline rubrics with concrete, distinguishable levels
- Aim for 5-15 checks total across all three dimensions

## Step 4: User Reviews Contract

Present the generated contract in full. Ask:
> "Here's the verification contract. Review each section — you can add, remove, or modify any check. Let me know when it looks right."

Wait for approval before saving.

## Step 5: Save Contract

- Create `.plumbline/contracts/` directory if it doesn't exist
- Write the contract to `.plumbline/contracts/<name>.md`
- The name is a kebab-case slug derived from the task description (e.g., "Add user authentication" → `add-user-authentication.md`)
- Announce: "Contract saved to `.plumbline/contracts/<name>.md`. When implementation is complete, run `/plumbline:verify` to verify against this contract."
