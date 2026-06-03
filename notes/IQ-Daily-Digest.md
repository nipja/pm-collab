---
title: "IQ Daily Digest — Scheduled Action Summaries"
created: 2026-06-03
last_updated: 2026-06-03
source_count: 4
status: draft
summary: "Scheduled delivery of IQ action summaries via email/Slack is on the post-July 2026 roadmap, is explicitly requested by EAP customers, and aligns with the 'operational workflows' investment pillar — but is blocked by the Atlas Notifications platform not yet being built."
type: research
tags: [iq, notifications, operational-workflows, engagement, email, slack]
---

# IQ Daily Digest — Scheduled Action Summaries

> ⚠️ CONTRADICTION (open): IQ-Strategy.md lists Atlas Notifications (the platform notification service) as "Roadmap — not shipped" in the platform services table, yet the same document lists "ServiceNow, Slack, Teams, CSP notifications" as Q1 FY27 priorities with no dependency caveat. It is unclear whether Q1 notification integrations are expected to build on Atlas Notifications or bypass it via a direct integration. Resolution unknown — needs confirmation from Platform PM before committing to a delivery path. — Sources: IQ-Strategy.md (platform services table, 2026-05-01 Jon/Himanshu sync).

Scheduled delivery of IQ action summaries to operators and managers outside the portal is a confirmed roadmap intent, validated by direct EAP customer requests, and consistent with IQ's "operational workflows" investment pillar. The primary risk is infrastructure: the platform notification service (Atlas Notifications) is not yet built, and the delivery mechanism for V1 email is unowned.

---

## What we know

### Customer demand is confirmed and specific

- EAP customers at the May 14, 2026 office hour listed **scheduled reports** as a top request, alongside proactive alerting and integrations. [Source: IQ-Strategy.md, EAP office hour 2026-05-14]
- Abbe National Bank (Sarah Cho, VP NetEng; Marcus Webb, Sr. NetOps) explicitly requested **scheduled reports for manager stakeholders** outside IQ. RBAC-aware delivery is described as a hard requirement by Marcus Webb. [Source: IQ-Strategy.md, meeting-notes-abbe-national-bank.md]
- SE Stephen Holland (Apr 2026) requested **scheduling / routines**: "run queries on intervals; notify via Teams/Slack when thresholds are met" — framing it as a replacement for Splunk-based reporting. [Source: IQ-Strategy.md, 2026-04-29]
- **Proactive alerting** is listed as the #1 EAP feedback theme: "customers want IQ to detect and notify before they ask." [Source: IQ-Strategy.md, EAP feedback themes]
- **Integrations** (Slack, Teams, ServiceNow, Jira) are the #2 EAP feedback theme. [Source: IQ-Strategy.md, EAP feedback themes]

### It is on the roadmap — but post-July 2026

- Scheduled recurring reports are explicitly confirmed as **post-July 2026** roadmap, at the EAP office hour. Quote from roadmap: "any ad-hoc query+visualization can become a recurring report delivered via email or integrated channels (Slack, Teams, ServiceNow, BMC Remedy Helix)." [Source: IQ-Strategy.md, 2026-05-14]
- Threshold-based scheduled reports (e.g., "daily list of devices with >50 DHCP requests/day") are explicitly called out for NOC and field services teams. [Source: IQ-Strategy.md, post-July 2026 roadmap]
- ServiceNow, Slack, Teams, CSP notifications are listed as **Q1 FY27 priorities**. [Source: IQ-Strategy.md, 2026-05-01]

### It aligns with stated strategic investment direction

- "Operational workflows" — scheduling reports, pinning items, Slack/email integration, recurring task automation — are the **secondary monetization and investment vector** for IQ after the data moat, per the May 18 Jon/Himanshu alignment. [Source: IQ-Strategy.md, 2026-05-18]
- IQ's "third pillar" trajectory: "recurring insights delivered to the channels where operators already work." [Source: IQ-Strategy.md, 2026-05-18]
- Jon's notifications work (defining triggers, audiences, channels) is explicitly being coordinated with the new anomaly detection hire's scope — anomaly events are the primary notification trigger. [Source: IQ-Strategy.md, 2026-05-11]

### Engagement problem makes this urgent

- IQ portal DAU and question volume are described as "very low" as of April 2026. An open-ended chat interface alone is "insufficient to drive engagement." [Source: IQ-Strategy.md, 2026-04-27]
- The digest is a push mechanism that can drive users back into the portal — addressing the engagement problem without requiring a portal visit to initiate.

### The FYI / FIA classification is already built

- IQ action cards are already classified as **FYI** (For Your Information — diagnostic) or **FIA** (For Immediate Action — time-sensitive). [Source: IQ-Strategy.md]
- This classification is a natural content filter for digest formatting: FIA items go at the top; FYI items follow. No new classification logic required.

### Monetization tier: free tier, not paid

- IQ (base) is **included with base product** — no extra license. IQ+ (paid) gates write/agentic operations. A read-only digest of action summaries with portal deep links is firmly base-tier content. [Source: IQ-Strategy.md, GTM/Pricing section, 2026-05-01]

### Infrastructure dependency: Atlas Notifications is not built

- The unified Atlas Notifications service is listed as **"Roadmap"** in the platform services table — not yet shipped. [Source: IQ-Strategy.md, platform services table]
- Without this, email delivery for V1 requires either: (a) waiting for Atlas Notifications, (b) a direct email service integration (e.g., SES/SendGrid) owned by the IQ team, or (c) leveraging CSP's existing notification hooks.

### RBAC complexity is real but deferrable

- Marcus Webb (Abbe National Bank) flagged that sharing must be RBAC-aware: a digest sent to a manager must only include actions that manager is entitled to see. [Source: IQ-Strategy.md, meeting-notes-abbe-national-bank.md]
- V1 scoped to account admins (full visibility by definition) avoids this complexity without removing value.

---

## What we don't know / open questions

- **Is Atlas Notifications on a delivery timeline?** If it ships in Q1 FY27, the digest can ride it. If it's 12+ months out, the IQ team must decide whether to build direct email delivery. — Resolve with Platform PM.
- **Who owns notification preferences UX?** Account-level digest settings (recipient list, cadence, opt-out) could belong to the IQ team or the platform settings team. Ownership is ambiguous. — Resolve with Platform PM and IQ PM.
- **What is the per-digest cost at scale?** If V1 is plain structured data (no LLM summarization), cost should be negligible. If a future version generates AI narrative summaries, cost-per-digest must be modeled before committing. — Resolve with engineering.
- **Does email delivery require legal/privacy review?** IQ's existing T&C addendum covers data access within the portal. Sending action data to external email addresses (especially manager recipients who may not be portal users) may require additional consent language. — Resolve with Olga Phillips (Legal).
- **How does the digest interact with the Slack/Teams integration timeline?** Q1 FY27 priorities list those integrations. A digest V1 in email followed quickly by V2 in Slack may be the right sequencing — but only if the underlying notification platform can support both. — Resolve with roadmap planning.

---

## Related

- [[IQ-Strategy]] — operational workflows investment pillar, post-July roadmap, EAP feedback themes, FYI/FIA classification, Atlas Notifications status
- [[IQ-COGS]] — cost implications of LLM-based vs. structured digest content
- [[MCP-Server-Strategy]] — Slack/ServiceNow delivery via MCP as an alternative to direct integration
