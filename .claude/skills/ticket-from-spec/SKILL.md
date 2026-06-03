---
name: ticket-from-spec
description: Generate Jira tickets from a PRD and create them via the Atlassian MCP. Use when asked to turn a spec into tickets/issues.
disable-model-invocation: true
argument-hint: "[path to PRD]"
allowed-tools: Read, Glob, Grep
---
Generate Jira tickets from the PRD at: $ARGUMENTS

Steps:
1. Read the PRD. Derive one epic plus a set of stories/tasks from its requirements
   and acceptance criteria.
2. For each ticket draft: title, description (with strategic context from the PRD),
   acceptance criteria (Given/When/Then), and implementation notes.
3. Map to the Jira project key listed in `knowledge-base/product-context.md`.
4. **Show the full list of planned tickets and STOP. Ask for explicit confirmation
   before creating anything in Jira.** Only after the user confirms, create them via
   the Atlassian MCP and link each back to the PRD/epic.
