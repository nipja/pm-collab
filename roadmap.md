# Roadmap

> The only doc the team maintains by hand as a deliberate source of intent.
> Everything downstream (PRDs, tickets) is generated from it.

_Last updated: 2026-06-03_

## Now (in progress)

- **Approval workflows (IBIQ-648)** — Human-in-the-loop approve/deny gate for all IQ write operations. Blocker for any Phase 2 action execution. Target: PI 26.3.
- **IQ for DDI — DHCP anomaly actions EAP wave** — Rapid discover floods, PXE boot loops, reservation failures, scope-aware behaviors. ~4–6 weeks from May 14. Owner: IQ DDI team.
- **Persistent chat history (IBIQ-266)** — Users can return to past investigations across sessions. Work started May; delivery target July 2026.
- **MCP Server — RBAC milestone (IBIQ-1193)** — Agent scope restriction; least-privilege default for external MCP clients.
- **IQ for Threat Defense GA enablement** — Auto-enable for ~100 SOC Insights / Security Token customers; legacy SOC Insights UI deprecated after 30-day transition. GA: June 2, 2026.

## Next (committed, not started)

- **QPS-based pricing tiers for IQ** — Finalize token/tier model for IQ for DDI and Asset Insights at SRS 2026. Must be QPS-aware to protect margin on large-grid enterprise accounts. PRD: `prds/qps-pricing-tiers-iq.brief.md` (brief only, sprint paused).
- **IQ for DDI Limited → GA** — Performance hardening, OAuth enterprise MCP, guardrails (500 SME-validated Q&A), PSEC signoff. Target: Sept–Oct 2026.
- **Inline widget rendering (A2UI)** — Replace text-only chat responses with metric cards, status pills, call-out blocks, and richer tables. Target: June–July prototype.
- **ServiceNow / Slack / Teams integrations** — Surface IQ actions and insights in existing operator workflows. Q1 priority.

## Later (directional, not committed)

- **Write operations via MCP** — Blocking domains, creating IP space via MCP behind RBAC + approvals + audit trail. Post-July.
- **Scheduled / recurring reports** — Any ad-hoc IQ query becomes a recurring report delivered via email or Slack/Teams. Post-July.
- **Shareable conversations and outputs** — Read-only conversation links; annotated example queries for onboarding. Post-July.
- **IQ for NIOS (on-prem)** — Requires T&C addendum (PII telemetry gate). GA target Sept–Oct 2026. NIOS MCP Server (on-prem, no cloud telemetry) target October 2026.
- **Security Actions GA** — Block indicator, isolate asset (human-approved). Target: October 2026.
- **IQ for Asset Insights GA** — Monetization model and ELA bundling TBD.
- **Cross-page memory** — IQ retains context and user corrections across sessions. October 2026.

## Recently shipped

- **IQ for Threat Defense — GA announcement** — Cisco Live, June 2, 2026. ~100 eligible customers auto-enabled. SOC Insights repositioned under IQ umbrella.
- **Slash-command prompts in IQ Chat** — `/` in chat queries MCP gateway for available prompts; first prompt: `onboarding` config readiness check. Shipped May 14, 2026.
- **Gainsight PX component-level analytics** — First component-granularity usage tracking for IQ (button clicks, widget usage, section engagement). Shipped May 14, 2026.
- **Documentation pipeline** — 8 Confluence spaces + Salesforce KB + S3, ingested daily. Answer accuracy improved ~20% → ~80%. Activated April 2026.
- **NRT pipeline refactor (Kafka)** — Investigation latency reduced from ~15 min → ~1 min. Shipped early 2026.
