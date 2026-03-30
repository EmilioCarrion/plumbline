# Context Gathering Guide

This document describes how `plumbline:contract` collects information from each source to generate a verification contract.

## Source Priority

Gather sources in this order. Each source builds on the previous:

1. **Project docs** — establishes conventions, constraints, architecture
2. **Codebase** — reveals existing patterns, dependencies, technical context
3. **Issue/ticket** — defines what's being asked and why
4. **Domain research** — discovers external real-world constraints via web search
5. **User conversation** — fills gaps, resolves ambiguity, adds tacit knowledge

## Source: Project Docs

**What to look for:**
- `CLAUDE.md` / `AGENTS.md` — project conventions, coding standards, architectural decisions
- `docs/` directory — ADRs, specs, API documentation
- `README.md` — project overview, setup instructions, tech stack

**How to consume:**
- Use Glob to find: `CLAUDE.md`, `AGENTS.md`, `docs/**/*.md`, `README.md`
- Read each file found
- Extract: tech stack, conventions, testing approach, deployment constraints

**Fallback:** If no docs found, note this gap — the contract will rely more heavily on codebase analysis and user questions.

## Source: Codebase

**What to look for:**
- Files related to the task (use Grep to find relevant code by keywords from the task description)
- Test patterns — how are tests structured? what framework? (`**/test*/**`, `**/*.test.*`, `**/*.spec.*`)
- Project structure — monorepo? microservices? what build tool?

**How to consume:**
- Use Glob to understand project structure: `**/package.json`, `**/pyproject.toml`, `**/Cargo.toml`, `**/go.mod`
- Use Grep to find code related to the task's domain keywords
- Read relevant files to understand existing patterns

**Fallback:** If empty project (no source files), skip codebase analysis. Note in contract context that this is a greenfield project.

## Source: Issue/Ticket

**What to look for:**
- Task description, acceptance criteria, linked issues
- Comments with additional context or decisions

**How to consume:**
- Check if MCP tools are available: Jira (`mcp__claude_ai_Atlassian__getJiraIssue`), GitHub Issues, Linear
- If the user provides an issue reference (e.g., "PROJ-123", "#45"), fetch it via the appropriate MCP tool
- Extract: what's being asked, acceptance criteria, constraints mentioned, related issues

**Fallback:** If no MCP tools available or no issue referenced, ask the user to describe the task. Their description becomes the primary input.

## Source: Domain Research

**When to use:**
When the task depends on external domain knowledge not available in the project or the user's head — travel, third-party APIs, regulations, hardware constraints, pricing, scheduling against real-world hours.

Skip for pure code tasks where the codebase is the domain.

**What to look for:**
- Real-world constraints that affect the task (opening hours, seasonal availability, rate limits, legal requirements)
- Domain-specific gotchas the user may not think to mention
- Current data that could invalidate assumptions (prices, schedules, API versions)

**How to consume:**
- Use WebSearch to research domain-specific constraints relevant to the task
- Focus searches on constraints and failure modes, not general background
- Extract specific, verifiable facts that should become checks

**Fallback:** If web search is unavailable or the task is purely project-internal, skip this source. Note in the contract context that external domain constraints were not researched.

## Source: User Conversation

**When to ask questions:**
After consuming the other sources, identify gaps in the three verification dimensions:

- **Functional gaps:** "The issue says 'add authentication' but doesn't specify the auth strategy. Are you using JWT, sessions, or OAuth?"
- **Craft gaps:** "I see the codebase uses repository pattern. Should the new feature follow the same pattern?"
- **Contextual gaps:** "This endpoint will be in the hot path. Is there a latency budget?"

**How to ask:**
- One question at a time
- Prefer multiple choice when possible
- Questions must be informed by what was found in other sources — never ask generic questions
- Stop when all three dimensions have sufficient coverage (minimum 2-3 questions total)
