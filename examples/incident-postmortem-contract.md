---
task: "Write incident postmortem for payment processing outage (2026-03-28)"
sources:
  - type: docs
    paths: ["docs/postmortem-template.md", "docs/incident-response-playbook.md"]
  - type: codebase
    paths: ["src/payments/", "infra/alerts/"]
  - type: issue
    ref: "INC-892"
created: 2026-03-30
status: pending
---

# Verification Contract: Incident postmortem for payment processing outage

## Context

A postmortem document for a 47-minute payment processing outage caused by a connection pool exhaustion in the payment gateway adapter. The team uses a blameless postmortem format with required sections: summary, impact, timeline, root cause analysis, contributing factors, and action items. The postmortem template (docs/postmortem-template.md) requires all action items to have owners and due dates. This is a P1 incident — the postmortem will be reviewed by engineering leadership.

## Functional Verification

- [ ] `[auto]` All required sections are present: Summary, Impact, Timeline, Root Cause Analysis, Contributing Factors, Action Items
  <!-- verify: grep -n "^## " FILE and check that all six section headers are present -->
- [ ] `[auto]` Timeline has at least 5 entries with timestamps in ISO 8601 format
  <!-- verify: grep -cE "\d{4}-\d{2}-\d{2}T\d{2}:\d{2}" FILE — must be >= 5 -->
- [ ] `[auto]` Every action item has an owner and a due date
  <!-- verify: extract the Action Items section, verify each item contains an @mention or name AND a date. grep -A 1 "- \[ \]" to inspect each item -->
- [ ] `[auto]` Impact section includes quantified metrics (affected users, failed transactions, revenue impact)
  <!-- verify: grep -i "users\|transactions\|revenue\|\$\|%" in the Impact section — must find at least 2 quantified metrics -->
- [ ] `[auto]` The document references the incident ticket INC-892
  <!-- verify: grep "INC-892" FILE -->

## Craft Verification

- [ ] `[manual]` Root cause analysis goes beyond the immediate trigger to identify systemic causes
  <!-- rubric:
  4: Identifies the immediate trigger AND at least two systemic factors (process gaps, architectural weaknesses, missing observability) with clear causal chain between them
  3: Identifies the immediate trigger and one systemic factor, causal relationship is clear
  2: Describes what happened technically but doesn't connect it to systemic causes — reads like a bug report
  1: States the trigger only ("the connection pool ran out") with no deeper analysis
  threshold: 3
  -->
- [ ] `[manual]` Contributing factors are distinct from the root cause and identify independent conditions that allowed the incident to escalate
  <!-- rubric:
  4: Each contributing factor is independently actionable, clearly distinct from the root cause, and explains how it amplified the impact or delayed detection/recovery
  3: Contributing factors are distinct from root cause and mostly actionable, though one may overlap or be vague
  2: Contributing factors are listed but some are just restating the root cause in different words
  1: No contributing factors, or they're indistinguishable from the root cause
  threshold: 3
  -->
- [ ] `[auto]` Tone is blameless — no individual names appear in the Root Cause Analysis or Contributing Factors sections
  <!-- verify: extract Root Cause Analysis and Contributing Factors sections, grep for @mentions or capitalized proper nouns that could be person names. Must return empty -->
- [ ] `[auto]` Action items are specific and measurable, not vague ("improve monitoring" vs "add alert for connection pool utilization > 80%")
  <!-- verify: grep -i "improve\|better\|more\|enhance\|consider" in Action Items section — vague verbs should be absent or minimal -->

## Contextual Verification

- [ ] `[manual]` Action items are realistic given current team capacity and priorities — not a wish list
  <!-- rubric:
  4: Each action item has a scope that one person could complete by its due date, and due dates account for the team's current sprint commitments
  3: Action items are achievable but at least one might be ambitious given the timeline — acknowledged as a stretch
  2: Several action items are large efforts disguised as tasks ("redesign the payment adapter") with short deadlines
  1: Action items are aspirational with no realistic path to completion
  threshold: 3
  -->
- [ ] `[auto]` The postmortem references related past incidents if any exist
  <!-- verify: grep -i "INC-\|previous\|similar\|incident.*20[0-9][0-9]\|recurrence" FILE — check for references to past incidents or explicit statement that none exist -->
- [ ] `[manual]` The postmortem would give an on-call engineer who wasn't involved enough context to handle a recurrence without escalating
  <!-- rubric:
  4: Timeline + root cause + contributing factors form a complete narrative — an uninvolved engineer could diagnose a recurrence and knows exactly which runbook or action to take
  3: Enough context to understand what happened and where to look, though the engineer might need to check one additional source
  2: Key details are missing — the reader would need to ask someone who was there
  1: Only understandable to people who were in the incident response
  threshold: 3
  -->
