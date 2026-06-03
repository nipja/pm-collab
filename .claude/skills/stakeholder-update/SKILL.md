---
name: stakeholder-update
description: Draft a stakeholder update tailored to an audience. Use when asked for a weekly/monthly update, status report, or exec summary.
disable-model-invocation: true
argument-hint: "[audience: exec | engineering | customer]"
---
Write a stakeholder update for audience: $ARGUMENTS

Structure (match voice-and-style.md):
- TL;DR (2-3 sentences)
- RAG status (Red / Amber / Green) with one line of why
- Progress vs goals — outcomes, not activity lists
- Risks + mitigations
- Specific asks with owners and deadlines
- Next milestones

Pull current status from the roadmap, recent commits, and Jira (via the Atlassian MCP)
if available. Tailor depth to the audience: execs get outcomes and asks; engineering gets
scope and dependencies; customers get value and timing. Do not invent progress — if a
status is unknown, say so.
