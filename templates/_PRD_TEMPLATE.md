---
status: draft          # draft | in-review | approved | shipped | archived
feature: <kebab-slug>
author: <github-username>
created: <YYYY-MM-DD>
last_updated: <YYYY-MM-DD>
jira_epic: <IBIQ-XXX once created>
dependencies: []       # list of PRD slugs or Jira epics this depends on
---

# PRD: <Feature name>

| | |
|---|---|
| **Status** | Draft |
| **Author** | <name> |
| **Last updated** | <date> |
| **Jira epic** | <ABC-123 once created> |

## 1. Problem
What user problem are we solving, and why now? Who is affected (link personas)?
Evidence: <research, tickets, metrics>.

## 2. Goals & success metrics
- **Goal:** <user/business outcome, not a feature>.
- **Success metric(s):** <measurable, with target and timeframe>.
- **Guardrail metric(s):** <what must NOT get worse>.

## 3. Non-goals
- <Explicitly out of scope for this effort.>

## 4. Proposed solution
Brief narrative of the approach. Link mockups if any.

## 5. Requirements (prioritized — RICE)

| # | Requirement | Reach | Impact | Confidence | Effort | RICE | Priority |
|---|-------------|-------|--------|------------|--------|------|----------|
| 1 | <requirement> | | | | | | Must |
| 2 | | | | | | | Should |

## 6. Acceptance criteria (Given / When / Then)
- **Given** <context>, **when** <action>, **then** <expected result>.
- ...

## 7. Risks & open questions
- <Risk> — mitigation.
- `TODO: confirm with <role>` — <open question>.

## 8. Rollout
Phasing, flags, who gets it first, how we measure, rollback plan.
