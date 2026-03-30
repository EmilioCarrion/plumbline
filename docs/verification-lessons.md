# Verification Lessons

What we've learned about verification from real Plumbline sessions. These lessons inform the tool's design decisions and are useful context for contributors and future development.

## The core insight

Generating is cheap. Verification is where errors hide. But verification itself has failure modes — a 13/13 pass rate means nothing if the checks were wrong. Plumbline's job is to make verification rigorous, not just present.

## Lessons

### 1. Contracts can only verify what they know to ask

A contract is a model of "done." Like all models, it's incomplete. The most dangerous failures are checks that *should exist but don't* — unknown unknowns.

**What happened:** A Madrid travel itinerary scheduled El Rastro (a Sunday-morning-only market) at 15:00. All 13 checks passed. The worst error went undetected because no check covered "visit time-sensitive attractions during their operating window."

**Design response:** Added domain research as a 5th context source (v0.3.0) and dimensional coverage hints (v0.3.0) — common failure patterns that nudge the contract generator to include checks it might otherwise miss. These don't eliminate unknown unknowns (nothing can), but they reduce the most common ones.

**Principle:** You can't systematically catch what you don't know you don't know. You can only: (a) use more information sources, (b) apply patterns from past failures, and (c) be honest about the limitation.

### 2. Verification hints cause more problems than they solve (when mandatory)

Execution hints (`<!-- verify: grep -E "pattern" file -->`) were originally required on every `[auto]` check. Three sessions revealed consistent friction:

- `grep -P` hints fail on macOS (environment coupling)
- Vague hints like "verify using map data" aren't actionable (false precision)
- Semantic checks forced into grep when analytical reading would work better (wrong tool)
- Hints become stale if file structure or environment changes

Every hint-related friction would have been avoided if the verify skill had used its own judgment.

**Design response:** Hints are now optional (v0.5.0). The contract defines WHAT to verify; the verify skill determines HOW. Hints remain valuable for command-based checks (exact commands = reproducibility) but are unnecessary for analytical or web-verified checks.

**Principle:** Separate specification from implementation. The contract is a spec. The hint is an implementation detail. Coupling them creates brittle verification.

### 3. grep matches syntax, not semantics

A check like "the itinerary doesn't recommend entering museums" verified via `grep -i "museum"` is fundamentally broken. The word "museum" can appear in context that doesn't constitute a recommendation. Conversely, a recommendation could use different words.

**Design response:** Added guidance distinguishing structural checks (where grep works) from semantic checks (where analytical evaluation works). For structural properties — use commands. For meaning — let the agent read and evaluate (v0.4.0, refined in v0.5.0).

**Principle:** Choose the verification tool that matches the nature of the check. Structural properties are syntax; semantic properties are meaning. Different tools for different jobs.

### 4. The LLM treats "should" as "could"

The contract format spec defined `status: pending` in frontmatter. The rubric format was documented. The report structure was specified. In practice, the LLM skipped fields, omitted rubrics, and generated ad-hoc reports — because the skill instructions said "read the format reference" rather than "follow it exactly."

**Design response:** Changed guidance from descriptive ("the format includes...") to prescriptive ("MUST include...") throughout (v0.5.0). Explicit enforcement for: status field in frontmatter, rubrics on manual checks, report format compliance.

**Principle:** When instructing an LLM, "should" means "sometimes will." If compliance matters, say "MUST" and be specific about what's required.

### 5. Verification tools should match the execution environment

`npx markdownlint-cli` fails without Node.js. `grep -P` fails on macOS. Ecosystem-specific tools assume an ecosystem that may not be present — especially for non-code tasks.

**Design response:** POSIX-compatible commands by default. Ecosystem tools only when the build system is detected (v0.4.0). Skipped check state for unexecutable checks (v0.4.0).

**Principle:** Auto checks must be executable in the environment where verification runs. If a check can't run, it should fail gracefully (skipped with reason), not silently disappear.

### 6. Domain-agnostic design works, but needs escape hatches

Plumbline's three dimensions (functional, craft, contextual) apply to travel itineraries, incident postmortems, and API implementations equally well. The contract format doesn't assume code. This is a genuine strength.

But the *skills* assumed code: context gathering scanned for package.json, execution hints defaulted to shell commands, examples were all software.

**Design response:** Better fallback messaging for non-code projects (v0.4.0). Non-code acknowledgment in the contract skill (v0.5.0). Optional hints that don't force command-based verification (v0.5.0).

**Principle:** Domain-agnostic formats need domain-agnostic tooling. If the format works for any task, the skills should too.

### 7. Manual check UX matters more than you think

Six manual checks = six conversation turns of "score 1-4." This is tedious for the user and burns tokens. The one-at-a-time flow exists for good reason (each check deserves attention), but forcing it on every check is too rigid.

**Design response:** Batch mode option after the first manual check (v0.4.0). User chooses between careful one-by-one and efficient batch.

**Principle:** Give users control over the verification pace. Some checks deserve careful consideration; others are quick confirmations. The tool shouldn't force one mode.

### 8. Bidirectional links prevent orphaned artifacts

The report pointed to the contract, but the contract didn't point back to the report. After verification, you had to remember which report belonged to which contract.

**Design response:** Verify now adds `report: <name>-report.md` to the contract's frontmatter (v0.5.0).

**Principle:** Related artifacts should reference each other. If A points to B, B should point back to A.

### 9. Skipped checks are real information

When a check can't execute (wrong environment, missing tool), it was silently omitted. The report showed 12/12 when reality was 11/12 + 1 skipped. This is worse than a failure — it's invisible.

**Design response:** Added `skipped` state with mandatory reason and `auto_skipped` counter in reports (v0.4.0).

**Principle:** The absence of a result is itself a result. Track it. A skipped check might be the one that would have caught the bug.

### 10. Each failure is a pattern, not an architecture problem

The temptation after each session was to design a system that "solves" the category of failure — research phases, meta-checks, adversarial review agents. Each proposed architecture was elegant and each was flawed in the same way: it deferred the problem to a component with the same limitations.

What actually worked: small, specific fixes. Add a line of guidance. Improve a fallback message. Make hints optional. Each fix addresses a concrete failure mode observed in a real session.

**Principle:** Improve the tool through accumulated experience, not architectural redesign. The right response to "El Rastro was scheduled wrong" is not "build a research subsystem" — it's "add temporal fitness to the coverage hints." Over time, the pattern library grows and the tool gets meaningfully better.

## Version history of key decisions

| Version | Key decision | Lesson # |
|---------|-------------|----------|
| v0.2.0 | Inline rubrics for manual checks, analytical auto checks | 3 |
| v0.3.0 | Domain research as 5th context source, dimensional coverage hints | 1 |
| v0.4.0 | POSIX compatibility, semantic vs structural guidance, batch mode, skipped state | 3, 5, 7, 9 |
| v0.5.0 | Optional hints, enforced specs, non-code support, bidirectional links | 2, 4, 6, 8 |
