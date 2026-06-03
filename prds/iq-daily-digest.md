# PRD: IQ Daily Digest

| | |
|---|---|
| **Status** | Draft |
| **Author** | Product sprint (njain2) |
| **Last updated** | 2026-06-03 |
| **Jira epic** | TODO: confirm with PM once project key confirmed |
| **Confluence** | _auto-linked after sync_ |

---

## 1. Problem

IQ surfaces anomaly action cards in real time inside the Infoblox portal — but most NetOps admins and SOC analysts don't live in the portal all day. When they're in meetings, working a incident, or offline overnight, open actions accumulate silently. There is no mechanism for IQ to reach operators in the channels where they already work.

Two groups are affected:

- **NetOps / DNS-DHCP-IPAM Admins** miss newly detected actions and may not return to the portal until a user complaint forces the issue. A For Immediate Action (FIA) card surfaced at 2am goes unworked until the morning standup.
- **Managers and team leads** (VP NetEng, NOC leads) have no network health visibility without logging into the portal themselves. They cannot answer "how many open issues do we have right now?" without pulling up a dashboard.

**Why now:** IQ portal DAU and question volume are "very low" as of April 2026 — the engagement problem is acknowledged by leadership. Scheduled reports and proactive alerting are the #1 and #2 EAP customer feedback themes. Abbe National Bank's VP of NetEng stated directly: *"I'll figure something out in IQ — and then I want to share that with my manager but I can't."* The Daily Digest is the push mechanism that drives operators back to the portal and gives managers passive visibility without requiring a login. [Source: IQ-Strategy.md, meeting-notes-abbe-national-bank.md]

---

## 2. Goals & success metrics

- **Goal:** Drive habitual IQ engagement by delivering action summaries to operators and managers in their existing workflows, without requiring a portal visit to initiate.

- **Success metric:** ≥40% of digest recipients click through to the Infoblox portal at least once per week within 60 days of launch.

- **Secondary metric:** Average time-to-first-action-view on newly detected FIA cards decreases by ≥30% for accounts with digest enabled, vs. accounts without digest (measured over 30 days post-launch).

- **Guardrail metric:** IQ portal weekly active users does not decline after digest launches. Digest should pull users into the portal, not substitute for it. If WAU drops ≥10% within 30 days of digest launch, pause and investigate.

---

## 3. Non-goals

- **Slack and Microsoft Teams delivery (V1).** Delivery channel integrations depend on the Atlas Notifications platform service and the Q1 FY27 connector roadmap. V1 is email-only.
- **LLM-generated narrative summaries.** V1 digest content is plain structured data pulled directly from existing action card fields — no AI summarization. This controls cost, latency, and hallucination risk.
- **RBAC-filtered per-recipient digests.** V1 is sent to account admins only (full visibility by definition). Per-recipient entitlement filtering is a V2 requirement.
- **Threshold-based conditional sending** (e.g., "only send if ≥3 critical actions open"). V1 sends on a fixed schedule regardless of action count.
- **Digest of resolved / closed actions.** V1 covers open and newly detected actions only.
- **Self-service recipient management for non-admins.** V1 recipient list is managed by account admins. Individual user subscription preferences are V2.

---

## 4. Proposed solution

Once per day (default: 8am recipient local time, configurable), Infoblox IQ sends a structured email digest to the configured recipient list for the account. The digest covers the 24-hour window ending at send time.

**Email content structure:**

1. **Header** — Account name, digest date, total open action count.
2. **New this period** — Actions first detected in the last 24 hours, grouped by severity (FIA first, then FYI). Each item: action title, affected component (e.g., "DNS scope: 10.10.0.0/24"), detected time, one-line AI-generated description (from the existing action card summary — not new LLM generation), and a **View in IQ** deep link.
3. **Still open (>48h)** — Actions unresolved for more than 48 hours, sorted by age descending. Same fields. This is the "aging" signal for managers.
4. **Footer** — Manage digest settings link, opt-out link.

**Configuration (account admin, in portal settings):**
- Recipients: email addresses (comma-separated; V1 does not require portal accounts for recipients)
- Send time: hour of day (default 8am)
- Cadence: daily or weekly (default daily)
- Opt-out: per-recipient unsubscribe link in every email (CAN-SPAM compliant)

**What the digest does NOT do:** It does not resolve actions, approve actions, or trigger any IQ operation. It is read-only. Every action in the digest links back to the portal for all follow-up.

---

## 5. Requirements (prioritized — RICE)

_Reach: estimated % of EAP accounts that would use this feature. Impact: 1–3 scale (3 = high). Confidence: %. Effort: person-weeks._

| # | Requirement | Reach | Impact | Confidence | Effort | RICE | Priority |
|---|-------------|-------|--------|------------|--------|------|----------|
| 1 | Daily email digest with new FIA/FYI actions and aging open actions, sent to configured recipients | 80% | 3 | 85% | 3 | 68 | Must |
| 2 | Deep link from each digest action item directly to the action detail page in portal | 80% | 3 | 90% | 1 | 216 | Must |
| 3 | Account admin can configure recipient list, send time, and daily/weekly cadence in portal settings | 80% | 2 | 80% | 2 | 64 | Must |
| 4 | Per-recipient opt-out link in every email (CAN-SPAM compliance) | 80% | 2 | 95% | 1 | 152 | Must |
| 5 | Digest is skipped (not sent) if zero actions are open and zero were detected in the period | 70% | 2 | 90% | 1 | 126 | Should |
| 6 | FIA actions visually distinguished from FYI actions in email (e.g., red vs. grey label) | 70% | 2 | 80% | 1 | 112 | Should |
| 7 | Email open and click-through tracked per recipient per digest for engagement metrics | 60% | 2 | 70% | 2 | 42 | Should |
| 8 | Slack delivery channel (V2) | 50% | 3 | 60% | 5 | 18 | Won't (V1) |
| 9 | RBAC-filtered digest per recipient (V2) | 40% | 3 | 70% | 8 | 10.5 | Won't (V1) |

---

## 6. Acceptance criteria

**Digest generation and delivery**

- **Given** an account has digest enabled and at least one action was detected or is still open, **when** the scheduled send time is reached, **then** the email is delivered to all configured recipients within 5 minutes of the scheduled time.
- **Given** an account has digest enabled but zero actions are open and zero were detected in the period, **when** the scheduled send time is reached, **then** no email is sent and the skip is logged.
- **Given** a recipient has clicked the opt-out link in a prior digest, **when** the next digest is generated, **then** that recipient is excluded and is not re-added unless they explicitly re-subscribe.

**Content correctness**

- **Given** a digest is sent, **when** a recipient views it, **then** the "New this period" section lists only actions first detected within the last 24 hours (relative to send time), and the "Still open >48h" section lists only actions whose detection timestamp is >48 hours before send time.
- **Given** a digest is sent, **when** a recipient clicks a "View in IQ" deep link, **then** they are taken directly to the action detail page in the portal (authenticated login required; unauthenticated users land on the portal login page with the action URL preserved as redirect).
- **Given** both FIA and FYI actions exist in the digest, **when** a recipient views the email, **then** FIA items appear before FYI items within each section.

**Configuration**

- **Given** an account admin updates the recipient list in portal settings, **when** they save, **then** the change takes effect for the next scheduled digest (not the current in-progress one).
- **Given** an account admin sets a send time, **when** the digest is delivered, **then** delivery occurs within ±15 minutes of the configured local time for the account's primary timezone.

**No unintended side effects**

- **Given** a recipient views the digest, **when** they take no action, **then** no action cards in IQ are modified, resolved, or marked as viewed on their behalf.

---

## 7. Risks & open questions

**Risks**

- **Atlas Notifications not built (High).** The unified platform notification service is listed as "Roadmap — not shipped." If V1 email requires waiting for this service, the IQ team must either own direct email delivery (SES/SendGrid) or delay the feature. This is the single largest delivery risk. Mitigation: confirm Atlas Notifications timeline with Platform PM; if >2 quarters out, scope a lightweight direct email integration as an IQ-team deliverable.
- **Legal/privacy review for external email recipients (Medium).** Sending action data to email addresses that may not belong to portal users (e.g., a manager who doesn't have a portal login) may require consent language or T&C additions beyond IQ's existing addendum. Mitigation: consult Olga Phillips (Legal) before V1 ships.
- **Digest drives email engagement, not portal engagement (Medium).** If recipients treat the digest as a substitute for the portal rather than an entry point, WAU could decline. Mitigation: monitor guardrail metric (WAU) closely for 30 days post-launch; adjust content to make portal deep links the primary CTA.
- **Email deliverability (Low-Medium).** Infoblox-originated email to enterprise inboxes may hit spam filters. Mitigation: use a reputable transactional email provider; implement SPF/DKIM/DMARC; use a dedicated sending domain.

**Open questions**

- `TODO: confirm with Platform PM` — Is Atlas Notifications on a delivery timeline? Can V1 email ride it, or does the IQ team need to own the delivery mechanism?
- `TODO: confirm with Platform PM` — Who owns notification preferences UX — IQ team or platform settings team?
- `TODO: confirm with Legal (Olga Phillips)` — Does sending action data to external email recipients (non-portal users) require additions to the IQ T&C addendum or customer consent flow?
- `TODO: confirm with Engineering` — What is the effort estimate for digest templating + scheduling + email delivery, assuming a direct SES/SendGrid integration (not Atlas Notifications)?
- `TODO: confirm with PM (Jon Abbe)` — Should the digest cadence options include "real-time FIA alerts" (immediate push on FIA detection) as a separate setting, or is that a different feature?

---

## 8. Rollout

**Phase 1 — EAP only (target: Q1 FY27)**
- Enable for existing IQ EAP accounts only, behind a feature flag.
- Default: off. Account admin must explicitly enable and configure recipients.
- Email delivery only (no Slack/Teams).
- Measure open rate, click-through rate, and portal WAU delta for digest-enabled vs. control accounts over 30 days.
- Success gate to Phase 2: ≥40% click-through rate and no WAU decline in enabled accounts.

**Phase 2 — GA (following EAP validation)**
- Enable for all IQ base accounts (included with base product, no additional license).
- Add Slack delivery channel (dependent on Q1 FY27 connector roadmap).
- Add RBAC-filtered per-recipient digests.
- Add threshold-based conditional sending (only send if ≥N FIA actions).

**Rollback plan**
- Feature flag off disables digest generation and delivery immediately.
- No data mutation occurs on rollback — digest is read-only; no portal state is affected.
- Recipients who opted out are retained in the opt-out list (do not re-send on re-enable without explicit re-consent).
