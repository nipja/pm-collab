# Problem Brief — IQ Daily Digest

**Status:** Awaiting user input on forcing questions
**Created:** 2026-06-03

---

## Sharpened Problem Statement

IQ surfaces anomaly action cards in real time inside the portal — but most network and security operators don't live in the portal all day. When they're away (overnight, in meetings, heads-down on other work), open actions accumulate silently. There is no mechanism today for IQ to reach operators in the channels where they already work (email, Slack, Teams). The result: high-priority actions go unnoticed for hours, and managers with budget/oversight responsibility have no visibility into network health unless they log in themselves.

Concrete example from the KB: Abbe National Bank's Sarah Cho (VP NetEng) discovers insights in IQ but can't share them with her manager or teammates — "I want to share that with Derek or my manager but I can't." [Source: meeting-notes-abbe-national-bank.md]. The Daily Digest is the push equivalent of that pull problem.

---

## Forcing Questions

Before writing a spec, these need answers:

1. **Who is the primary recipient — the operator or the manager?**
   The NetOps admin who works actions directly, or the manager/VP who needs a health summary without logging in? These require very different content and tone. (Abbe National Bank's Sarah Cho = both; Marcus Webb = operator only.)

2. **Is this a digest of *open* actions, *new* actions, or *resolved* actions?**
   "24-hour summary" could mean: (a) everything still open, (b) only what was newly detected since last digest, (c) what was resolved and by whom. Each has different value and different complexity.

3. **What is the narrowest version that delivers value?**
   A plain-text email with: count of open actions by severity + top 3 unresolved by age + one-click deep link back to the portal. No AI summarization required. Would that alone move the needle on engagement?

4. **What would make this not worth building?**
   If the Slack/Teams/email notification platform (Atlas Notifications — currently "Roadmap" status per IQ-Strategy.md) isn't ready, this feature has no delivery mechanism and becomes a platform dependency, not a product feature. Has Atlas Notifications shipped?

5. **What does the KB already tell us?**
   - Scheduled recurring reports are explicitly on the post-July 2026 roadmap. [Source: IQ-Strategy.md]
   - ServiceNow, Slack, Teams, and CSP notifications are Q1 priorities. [Source: IQ-Strategy.md, May 1]
   - "Operational workflows" are the stated secondary investment after the data moat. [Source: IQ-Strategy.md, May 18]
   - The unified Notifications platform is listed as "Roadmap" — not yet built. [Source: IQ-Strategy.md, Platform services table]
   - Abbe National Bank explicitly requested scheduled reports for manager stakeholders. [Source: meeting-notes-abbe-national-bank.md]

6. **Success metric and guardrail?**
   Hypothesis: *primary metric* = % of EAP customers who open ≥1 digest/week; *guardrail* = digest does not replace portal visits (IQ portal DAU must not decline after digest launches — we want digest to drive portal engagement, not substitute it).

---

## Hidden Assumptions

- **Assumption:** Operators want a daily cadence. Some teams may prefer real-time alerts for FIA (For Immediate Action) cards and a weekly digest for FYI cards — not one size fits all.
- **Assumption:** Email and Slack are the right channels. Salesforce explicitly said "Slack-first." Western Bank Alliance asked for Teams. These may need to be treated as separate delivery integrations.
- **Assumption:** The digest content is a formatted summary of existing action card data. If it requires LLM summarization per recipient, cost per digest could be non-trivial at scale.
- **Assumption:** RBAC is not a blocker for V1. Marcus Webb (Abbe National Bank) flagged that sharing must be RBAC-aware — a digest sent to a manager must only include actions that manager is entitled to see. This is non-trivial and may scope-out V1.

---

## Known Facts (sourced)

1. Scheduled recurring reports are post-July 2026 roadmap, confirmed at EAP office hour. [Source: IQ-Strategy.md, 2026-05-14]
2. Slack, Teams, ServiceNow, CSP notifications are Q1 FY27 priorities. [Source: IQ-Strategy.md, 2026-05-01]
3. Atlas Notifications (unified platform notification service) is listed as "Roadmap" — not shipped. [Source: IQ-Strategy.md, Platform services table]
4. "Operational workflows" (scheduling, Slack/email integration, recurring tasks) are explicitly the secondary monetization/investment vector for IQ. [Source: IQ-Strategy.md, 2026-05-18 Jon/Himanshu]
5. Abbe National Bank requested scheduled reports for manager stakeholders; RBAC-aware sharing is a hard requirement from their NetOps lead. [Source: meeting-notes-abbe-national-bank.md]
6. EAP customers' top requests include scheduled reports and proactive alerting. [Source: IQ-Strategy.md, EAP office hour 2026-05-14]
7. FYI / FIA classification already exists for action cards (For Your Information vs. For Immediate Action). [Source: IQ-Strategy.md]
8. IQ portal DAU / question volume is described as "very low" as of April 2026 — engagement is a known problem. [Source: IQ-Strategy.md, 2026-04-27]

---

## Scope (proposed)

### In scope (V1)
- Daily (or configurable cadence) digest delivered via email
- Content: count of open actions by severity, top N unresolved actions by age, one-click deep link per action
- Configurable recipient list per account (admin sets who receives)
- Opt-out / pause per recipient

### Out of scope (V1)
- Slack / Teams delivery (depends on integration platform; defer to V2)
- LLM-generated narrative summary (plain structured data only for V1 — cost and latency risk)
- RBAC-filtered digests per recipient (required long-term; too complex for V1)
- Threshold-based triggers (e.g., "only send if ≥3 critical actions open") — V2
- Resolved/closed action history in digest — V2

---

## Open Questions

1. Is Atlas Notifications (the platform notification service) available, or does email delivery require a new infrastructure investment?
2. What is the engineering estimate for digest templating + scheduling + email delivery (without Slack)?
3. Does the monetization model gate this behind a paid tier, or is it a free engagement driver?
4. Who owns notification preferences UX — IQ team or platform team?

---

## Success Metric

**Primary:** ≥40% of digest recipients click through to the portal at least once per week within 60 days of launch.

**Guardrail:** IQ portal weekly active users does not decline after digest launches (digest should drive portal engagement, not replace it).

---

## Recommended next step

Proceed to `/synthesize-research` to pull KB material on notifications infrastructure, operational workflows roadmap, and EAP customer signals into a single sourced note.
