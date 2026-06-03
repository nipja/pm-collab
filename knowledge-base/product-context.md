# Product context

> This file loads into every Claude session in this repo. Keep it accurate and under ~150 lines.

## What we build

Infoblox IQ is the AI assistant and agentic operations layer across the Infoblox portfolio. It targets network operations (DNS/DHCP/IPAM) and security operations (SOC/threat detection) teams at mid-to-large enterprises. The core job: surface what to look at, explain what it means, and — progressively — take pre-approved remediation actions. IQ sits on top of Infoblox DDI, Threat Defense, and Asset Insights and is positioned as an "AI force multiplier," not a standalone product.

## Company / product stage

Infoblox is a Vista Equity Partners-backed enterprise networking and security company. IQ is in **Early Access Program (EAP)** as of June 2026 — ~30 active EAP customers. IQ for Threat Defense GA'd at Cisco Live (June 2, 2026). IQ for DDI is Limited Availability targeting Sept–Oct 2026 GA. AI-attributable ARR is a board-level reporting requirement. The broader PM org uses this repo across IQ DDI, IQ Security, and IQ Assets.

## Target users / personas

- **NetOps / DNS-DHCP-IPAM Admin** — manages network infrastructure; diagnoses NXDOMAIN spikes, DHCP scope exhaustion, lame delegations; drowned in alert noise and manual runbooks; wants to close tickets faster and prevent issues before they escalate.
- **SecOps / SOC Analyst** — investigates threat-correlated DNS/DHCP anomalies and correlates them with threat intel; wants to triage thousands of events down to a handful of actionable insights quickly.
- **Asset Manager / IT Operations** — tracks device lifecycle from discovery through vulnerability scan to decommission; frustrated by stale CMDBs and manual audit cycles; wants AI-driven discovery and anomaly detection.

## Strategic priorities (this quarter)

1. **IQ for DDI GA readiness** — reach GA quality bar across eval scores, guardrails, performance, OAuth, and PSEC signoff. Target: Sept–Oct 2026 GA. Metric: ≥4/5 eval scores across DNS + DHCP scenarios; zero P0 defects at launch.
2. **Pricing and commercialization** — finalize token/tier model for IQ DDI and Asset Insights at SRS 2026; model must be QPS-aware to protect margin on large enterprise accounts.
3. **Platform depth** — ship approval workflows (IBIQ-648, PI 26.3), persistent chat history (IBIQ-266, July), and MCP server RBAC to unblock write operations and external integrations.

## Key product areas

- **IQ Assistant (chat)** — conversational AI over DNS/DHCP/IPAM/Security data; read-only today (writes blocked until approval workflows land in PI 26.3); persistent chat history targeting July 2026.
- **IQ Actions — DDI** — proactive anomaly detection cards via Insighter pipeline; DNS at ~14 scenarios rated 5/5; DHCP actions EAP wave coming ~June; write execution gated on approval workflow.
- **IQ for Threat Defense** — GA as of June 2, 2026; SOC Insights repositioned under IQ umbrella; ~$2M ARR grandfathered in; ~100 eligible customers.
- **IQ for Asset Insights** — natural-language asset queries, AI-driven discovery; monetization model TBD; bundling with DDI/Security under ELA under discussion.
- **Customer-facing MCP Server** — live (read-only, API key auth, unadvertised); OAuth/OIDC GA milestone blocked on Platform (IBIQ-40).

## Out of scope / explicit non-goals

- Service provider / carrier customers (T-Mobile, Verizon) — QPS makes COGS unviable.
- NIOS on-prem IQ before T&C addendum is signed (PII telemetry gate); target Sept–Oct 2026.
- Editing Confluence directly — it is an auto-published mirror; truth lives in Git.
- Creating Jira tickets without explicit user confirmation — always show the draft list first.

## Source systems

- Roadmap: `roadmap.md`
- Tickets: Jira — TODO: confirm project key(s) (likely IBIQ for features, IQP for platform)
- Published docs: GitHub Wiki (auto-synced on merge to main) — enable wiki in repo Settings → Features → Wikis
- Deep context: `knowledge-base/IQ-Strategy.md`, `knowledge-base/IQ-Architecture.md`, `knowledge-base/IQ-COGS.md`
