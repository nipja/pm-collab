# Infoblox IQ: PM Onboarding Guide

**Audience:** New product manager joining the IQ team
**Current as of:** May 2026
**Maintained by:** Jon Abbe

---

## 1. What Is Infoblox IQ?

Infoblox IQ is the company's AI assistant layer across the full product portfolio: DNS/DHCP/IPAM (DDI), Threat Defense (security), and Asset Insights. It started as a diagnostic assistant for network operations and has expanded into agentic workflows with the ability to propose and execute approved changes.

The framing that matters: IQ is not a chatbot bolted onto the portal. It is a strategic platform play. The goal is a unified "AI force multiplier" for NetOps and SecOps teams. IQ is a Vista board priority; AI-attributable ARR is a board-level reporting requirement.

**One-sentence positioning (the official tagline):** "The agentic operations layer for network and security teams."

**Why Infoblox wins this category:** Any vendor can buy LLM access. Infoblox has 15+ years of DNS/DHCP/IPAM telemetry from real enterprise environments. That data moat is not replicable in 12 to 18 months. The AI is just the interface; the data is the differentiator.

---

## 2. The Two IQ Pillars

IQ itself has two core pillars:

| Pillar | What It Does | Revenue model |
|--------|-------------|---------------|
| **IQ Actions** | Proactive anomaly detection with AI-generated recommendations and human-approved remediation. The primary paid tier. | Premium (IQ+) |
| **AI Assistant** | Conversational natural-language interface for queries, investigations, and guided troubleshooting across DDI, Security, and Assets. | Included with product (limits TBD) |

The critical distinction: IQ Actions are the irreplaceable monetization vector. An external agent connecting via MCP plus an off-the-shelf LLM cannot replicate IQ Actions. IQ Actions require Infoblox-side anomaly detection models, evaluation-grade output quality, and the human-in-the-loop approval framework.

**A note on the MCP Server:** At Cisco Live, Infoblox marketed three pillars (IQ Actions, AI Assistant, MCP Server) as a single IQ story. But in practice, the **Infoblox MCP Server is a standalone platform effort**, separate from IQ. It exposes the full Infoblox API surface across all products (DDI, Threat Defense, Asset Insights, IPAM) and is owned at the platform level. IQ is just one of many data sources routed through it. The naming decision ("Infoblox MCP Server" not "IQ MCP Server") reflects this. MCP follows API T&C terms, not the IQ supplemental addendum. IQ uses MCP internally as its tool-call layer, but MCP is not an IQ feature and should not be positioned that way in roadmap or stakeholder conversations.

---

## 3. The Four IQ Surfaces

"IQ" is not one thing. Tom Hayward (IQ architect) formally defined four distinct surfaces, each with different ownership and behavior:

| Surface | Description | Owner | Write Operations |
|---------|-------------|-------|-----------------|
| **IQ Assistant (in-portal chat)** | Conversational agent calling Infoblox APIs via MCP tools | team-iq | Blocked today (DESTRUCTIVE_NOOP). Writes require approval workflow (IBIQ-648). |
| **Customer-facing MCP server** | Same MCP Gateway, exposed externally. Customers connect their own AI clients using portal API keys. | team-iq | Read-only today; write gated behind approval workflows in June/July. |
| **IQ for DDI / Insighter** | Telemetry-driven root-cause analysis via Insighter Research Agent. Recommendations only; no direct API writes. | team-iq | Recommendations only. Human approves/dismisses in portal. |
| **IQ for Threat Defense** | Separate implementation; not built on Insighter. | TD team (Shadid Chowdhury) | Human-in-the-loop per TD team design. |

**The DESTRUCTIVE_NOOP safety mechanism:** Until approval workflows ship, all write tool calls in the MCP server return API usage instructions (what would happen) rather than executing. This is the production safety mechanism. It is not a temporary hack; it is the pattern that makes agentic IQ viable in EAP.

---

## 4. What IQ Does Today (EAP Capabilities)

### IQ Actions (DDI)
IQ continuously monitors DNS and DHCP telemetry and surfaces anomaly cards when something materially impactful occurs. Each action card includes:
- Root cause analysis (which devices, scopes, or records are involved)
- AI-generated explanation of contributing factors
- Recommended remediation steps
- Downloadable logs for independent forensics

**Current detection coverage:** NXDOMAIN spikes, SERVFAIL spikes, DNS latency anomalies, cache hit ratio drops, DDNS spikes, lame delegation, dead forwarders, DHCP HA failover failures, scope exhaustion. 14 scenarios currently rated 5/5 in the internal eval harness.

**The two-stage investigation model:**
1. Anomaly agent (always running): multivariate and ML-based detection compared against environment-specific baselines. Surfaces only materially impactful situations; false positive control is a core design constraint.
2. Deep research agent (triggered per action): detailed investigation via tool calls, correlating across metrics, logs, and configuration data. Produces surgical recommendations naming specific devices, scopes, and records.

### IQ Actions (Threat Defense)
TD Actions is GA at June 2, 2026. It is the most mature IQ surface and the first to reach full GA. It surfaces threat correlations, assists SOC investigation, and proposes remediation (block indicator, isolate asset) with mandatory human approval. The AI Assistant bundled with TD Actions at June 2 is early release, not full GA. TD has its own implementation owned by Shadid Chowdhury's team.

### AI Assistant
The in-portal chat interface that answers natural-language questions about the customer's network: DNS query patterns, DHCP utilization, IPAM provisioning, security alerts. It calls Infoblox APIs via MCP tools. It is read-only during EAP. The system grounds answers in customer telemetry, not generic knowledge.

---

## 5. The Roadmap Arc

IQ is on a four-phase progression from read-only diagnostics to closed-loop autonomy:

| Phase | Mode | Status |
|-------|------|--------|
| Phase 1 | Read-only diagnostic assistant | Current (EAP) |
| Phase 2 | Guarded write with human approvals | 2026 target |
| Phase 3 | Semi-autonomous (approved action classes) | Future |
| Phase 4 | Closed-loop autonomous with oversight | Vision |

**Current FY26 milestones:**

| Milestone | Target | Notes |
|-----------|--------|-------|
| TD Actions GA | June 2, 2026 (Cisco Live) | Full GA. The first IQ component to reach GA. |
| AI Assistant with TD Actions | June 2, 2026 | Early release, not full GA. Bundled with TD Actions. |
| IQ for DDI | Limited Availability from June 2 | GA at SRS 2026 (Sales Readiness Summit). |
| IQ for Asset Insights | GA at SRS 2026 | Follows DDI. |
| Universal Tokens operational | September 2026 | All net-new customers on UTK. |

**Critical nuance:** At June 2, TD Actions is GA. The AI Assistant that ships alongside TD Actions is early release. IQ for DDI is Limited Availability. Do not describe the full IQ product as GA in June, and do not describe the AI Assistant as GA.

---

## 6. Key Design Principles

These principles shape every UX, architecture, and prioritization decision. Reference them in design reviews.

**1. Trust is the central metric.**
Every decision that reduces fear of AI action accelerates adoption more than any capability addition. Human-in-the-loop is not a constraint; it is the product. The default posture for any new action type is "Needs Approval."

**2. Customers want notifications before automation.**
Across all EAP customers: people want to be alerted first (Slack, email, syslog) before any automated action is taken. False positive risk is the primary reason. Default: notify, then offer to act.

**3. The A to B to C navigation model.**
A = IQ landing page (unified posture view), B = holistic posture layer (cross-product summary), C = app drill-downs (SOC detail, DDI detail, Assets). All UX decisions should be evaluated against this flow.

**4. IQ value is trapped in individual sessions.**
This is the number one customer pain signal from EAP. Shareable links, scheduled reports, and annotated examples are higher priority than new detection capabilities. Chat sharing is on the backlog (IBIQ-919).

**5. Evaluation-driven development.**
New capabilities ship when they pass automated evals, not when engineering says they are done. The baseline is 14 scenarios at rating 5/5. Every new action type needs its own eval scenario before it ships.

**6. Do not ship half-baked features.**
Himanshu's standing principle: it costs more to re-educate customers and reset expectations than it gains in early feedback. When in doubt, delay.

---

## 7. EAP Customers and Active Accounts

As of May 2026, 25+ customers are enabled in EAP. Active external users and accounts include:

| Customer | Segment | Key Use Cases |
|----------|---------|---------------|
| American Airlines | Fortune 500, airline | Two grids (23K QPS and 600 QPS). Contact: Earl. |
| ExxonMobil | Fortune 500, energy | IP scavenging, automation. Transitioning from MSP to internal teams. |
| Nationwide | Enterprise, insurance | DNS and DHCP investigation. Active at CAB. |
| Target | Enterprise, retail | DHCP/IPAM queries. |
| Salesforce | Enterprise, software | Slack-first access, MCP integration focus. Security review underway. |
| Caterpillar | On-prem NIOS, manufacturing | Log RCA, utilization, forecasting, ticket routing. |
| NYC DOE | SLED, 1M Chromebooks | DNS-based client detection, per-school baselines. |

The bi-weekly EAP office hour (started May 14, 2026) is the primary customer feedback channel. Attend these. The demo hackathon series is a secondary feedback loop for SEs.

---

## 8. Key People

**Engineering and Architecture:**
- **Tom Hayward** (IQ Architect, team-iq): architectural decisions, MCP gateway design, DESTRUCTIVE_NOOP, legal/compliance briefs.
- **Turner Anderson**: Data/analytics (CubeJS), MCP server design lead, competitive research.
- **James Zhu**: Engineering manager for IQ platform.
- **Hasmit Grover**: Engineering lead for MCP implementation and OAuth.
- **Julian Hernandez**: Frontend, slash-command prompts, chat UX.

**Product Management:**
- **Himanshu Raval** (Senior Director of Product): IQ DDI strategy, MSP portal roadmap. Your primary PM leadership contact.
- **Jon Abbe**: PM coordinator for MCP, cross-functional strategy, IQ Actions backlog. (That is you or your peer depending on your charter.)
- **Paul Lawrence**: PM for DHCP/IPAM IQ actions. Three DDI epics.
- **Anant Vadlamani**: PM for Security/Threat Defense IQ features.
- **Meenal Gupta**: PM/epic owner for customer-facing MCP server (IBIQ-1200).

**Leadership:**
- **Mukesh Gupta** (EVP/CPO): Executive sponsor for IQ and MCP. Set the "release MCP ASAP" directive. Sets AI-first product direction.
- **Mehul Patel**: Product Marketing. Owner of launch positioning, field response docs, and IQ webpage.

**Design:**
- **Chaitrali Dhole**: UX design lead for IQ.
- **Tejasvi Shiv**: Broader platform UX; also runs Gainsight PX analytics for IQ.

---

## 9. Architecture: What You Need to Know

You do not need to understand every service. You need to understand the conceptual model well enough to write accurate stories and hold technical conversations.

**The two bounded contexts:**
1. **AI Agent Context**: Handles conversational AI. IQ NLP Interface (Go) receives the user's query and routes to IQ Agent (Python/LangChain). IQ Agent calls MCP tools to retrieve data and generates responses.
2. **Insighter Context**: Handles telemetry ingestion and root-cause analysis. This is the engine behind IQ Actions (DDI). It continuously processes DNS/DHCP events from Kafka, runs anomaly detection, and generates action recommendations.

**Internal tool-call layer:**
Internally, both the AI Agent and Insighter Research Agent call Infoblox APIs through a centralized gateway. Two services handle this: MCP-CSP (Infoblox Portal REST APIs and CubeJS analytics) and MCP Insighter (ClickHouse telemetry queries). This is IQ's internal plumbing and is distinct from the standalone Infoblox MCP Server product.

**LLM stack:**
Azure OpenAI GPT via LiteLLM (model abstraction layer). LiteLLM routes all LLM calls, enables future model swaps, and handles failover between Azure regions. All traffic stays within AWS PrivateLink (no public internet).

**Data retention:**
Conversation history stored in PostgreSQL with 30-day TTL. Customer data is never used to train models. Azure enterprise tier: no Microsoft-side prompt retention.

**Key Jira boards:**
- `IBIQ` board: canonical IQ feature board. Use this.
- `IQP` board: largely uncurated legacy; focus on IBIQ.

---

## 10. Commercialization

**IQ packaging today (IQ vs. IQ+):**
- **IQ (included):** AI assistant natural language interface available to all UDDI, NIOS, Threat Defense, and Asset Insights customers at no extra charge. Usage limits to be defined.
- **IQ+ (premium, paid):** IQ Actions (DNS/DHCP/IPAM Actions and Security Actions), full agentic AI assistant capability, and Config Intelligence. Charged via tokens.

**Token model:**
- IQ for Threat Defense: approximately 35% of Security Token allocation.
- IQ for DDI: pricing TBD at SRS 2026.
- MCP Server: included with product licenses at no extra charge (for now).
- Universal Token migration: September 2026. One-day cutover. Hybrid mode was ruled out. All net-new customers on UTK at launch.

**PAYGO:** Infoblox SaaS marketplace billing via AWS and Azure for self-serve commercial expansion. Currently generating revenue ($238K cumulative as of May 1, 7 active customers).

---

## 11. Competitive Landscape

**BlueCat** is the most credible direct competitor in DDI AI. They have announced MCP Servers (tech preview now, GA July 2026) and LiveAssist for DDI (July 2026). Infoblox assessment: BlueCat's press release has little substance and no published technical architecture for auth, RBAC, or approval workflows. Infoblox has complete answers for all of those. Monitor BlueCat's July GA closely.

The official competitive response (field guide v3a, Paul Nguyen, May 20, 2026) is your reference for any competitive situation. Key talking points:
- Lead with what is GA: IQ for Threat Defense, live today.
- Frame IQ as "earned autonomy" with approval gates, not "full autonomy."
- Drive toward a proof motion: one incident, one workflow, 2 to 3 weeks, one measurable outcome.
- Do not speculate on DDI/Asset Insights pricing.

**NSA MCP Advisory (May 20, 2026):** The NSA published a security advisory on MCP risks (prompt injection, over-permissioning, supply chain). Infoblox's existing design (DESTRUCTIVE_NOOP, intersectional RBAC, audit logging, scoped tool permissions) satisfies all major recommendations. This is a competitive asset in federal and regulated-market sales.

---

## 12. Key Processes You Need to Join

**Sprint cadence:** 2-week sprints. Sprint planning starts with a pre-sprint product alignment meeting (PM picks candidates from M1/M2/M3 backlog), then engineering planning handles estimates and dependencies before sprint day 1. Stories must be estimated before sprint commit.

**Demo and review cadence:**
- Weekly IQ demo: engineering demos in-progress features. Attend.
- Bi-weekly EAP office hour: customer-facing product updates and Q&A.
- Monthly hackathon: SEs using IQ on live customer data; provides raw feedback.
- IQ Roadmap Workshop (monthly-ish): planning and priority alignment across PM and engineering leads.

**Jira workflow:**
- IBIQ board is the source of truth.
- Use the `roadmap` field for delivery month tagging (M1/M2/M3).
- Once a concrete GA release is named, stories get a GA release tag. Do not create a GA tag until leadership names the release.
- Mid-sprint additions from other squads require an explicit trade-off (something comes out when something goes in).

**Eval framework:**
Before claiming a capability is done, it needs eval scenarios passing at rating 4 or 5. The eval harness runs daily. Engineering publishes scores to a dashboard. PM is responsible for making sure eval scenarios exist for every new capability before it ships.

---

## 14. What to Read First

These wiki articles are your foundation. Read them in this order:

1. **[IQ-Strategy.md](../wiki/IQ-Strategy.md)** - The master article: vision, DDI/Security/Assets use cases, roadmap, EAP status, UX patterns, launch milestones.
2. **[CORE-INSIGHTS.md](../wiki/CORE-INSIGHTS.md)** - 18 durable insights distilled from all sources. Read this early; it frames how to think about every decision.
3. **[IQ-Architecture.md](../wiki/IQ-Architecture.md)** - Technical architecture. You need this for writing accurate stories and understanding dependencies.
4. **[MCP-Server-Strategy.md](../wiki/MCP-Server-Strategy.md)** - Why MCP exists, what it is and is not, auth/RBAC design, Cisco Live scope vs. enterprise GA scope.
5. **[UX-Design-Direction.md](../wiki/UX-Design-Direction.md)** - Design philosophy, split-pane pattern, A-to-B-to-C navigation model, prototyping process.
6. **[Tokens-and-Commercialization.md](../wiki/Tokens-and-Commercialization.md)** - Token model, universal token migration, IQ vs. IQ+ packaging.
7. **[competitive-intelligence.md](../wiki/competitive-intelligence.md)** - BlueCat positioning, what to say and not say in competitive situations.

**Customer context (read these for any EAP customer call):**
- `wiki/entities/american-airlines.md`
- `wiki/entities/exxonmobil.md`
- `wiki/entities/caterpillar.md`

---

## 15. The Most Common Mistakes to Avoid

1. **Conflating what is GA vs. early release at June 2.** TD Actions is GA at June 2. The AI Assistant bundled with TD Actions is early release. IQ for DDI is Limited Availability. These are three different states. Say each one precisely.

2. **Conflating the four IQ surfaces.** IQ Assistant (chat), Customer MCP, Insighter/DDI, and IQ for TD are different implementations with different owners and different write models. Be specific.

3. **Shipping without evals.** Every new capability needs an eval scenario passing at rating 4+ before it ships. Do not accept "feature complete" as the definition of done.

4. **Treating MCP as a product.** MCP is a distribution channel. The product is IQ Actions. The MCP server makes the data and action APIs available to external agents; it does not substitute for the differentiated Infoblox-side intelligence.

5. **Using legacy product names.** "BloxOne," "BloxOne Threat Defense," and "Universal DDI" are being cleaned up from all system prompts and documentation. Use "Infoblox IQ for DDI," "Infoblox IQ for Threat Defense," "Infoblox IQ for Asset Insights," and "AI Actions" / "AI Assistant" as the descriptive labels.

6. **Letting cross-squad additions into sprints without trade-offs.** When Security, Asset Insights, or NIOS teams add stories mid-sprint, something must come out. This is a standing process decision. Enforce it.

---

## 16. Quick Reference: Key Jira Epics

| IBIQ | Description | Status |
|------|-------------|--------|
| IBIQ-1200 | Customer-Facing MCP Server | Reviewing |
| IBIQ-648 | Approval Workflow (human-in-the-loop writes) | PI 26.3 |
| IBIQ-1162 | Full-screen IQ Chat Landing Page | In Progress |
| IBIQ-1193 | MCP Gateway RBAC via ext_authz + OPA | In Flight |
| IBIQ-40 | Identity as OAuth 2.1/OIDC provider for MCP | P1 Blocked (Platform dependency) |
| IBIQ-266 | Chat History | Reviewing |
| IBIQ-926 | Agent write capability (add/remove custom list domain) | Reviewing |
| IBIQ-1396 | Rich Response Formatting and Inline Visualizations | Reviewing |
| IBIQ-161 | Out-of-Scope Guardrails | In Progress |
| IBIQ-889 | DNS Cache Hit Ratio Drop detection | In Progress |

---

## 17. Glossary

| Term | Meaning |
|------|---------|
| EAP | Early Adopter Program. Current state for most IQ features. |
| LA | Limited Availability. Controlled access post-EAP; not full public GA. |
| GA | General Availability. Full public release. |
| Insighter | The analytics engine underlying IQ Actions (DDI). |
| DESTRUCTIVE_NOOP | Safety flag in MCP-CSP that intercepts all write tool calls and returns instructions instead of executing. |
| CubeJS | Analytics query layer over ClickHouse. 50+ cubes covering DNS/DHCP/IPAM metrics. |
| LiteLLM | Model abstraction layer routing all LLM calls. Enables future model swaps without rewiring. |
| UTK | Universal Token. Single metering unit replacing all per-product tokens. Operational September 2026. |
| MCP | Model Context Protocol. The open standard Infoblox MCP Server implements. |
| RBAC | Role-Based Access Control. Currently undergoing a major platform-wide overhaul. |
| OPA | Open Policy Agent. Target authorization enforcement layer (post-IBIQ-1193). |
| A2A | Agent-to-Agent. Infoblox-owned agents communicating with external agents as peers. Different from MCP (where external parties build on top of Infoblox). |
| SRS | Sales Readiness Summit. Internal Infoblox sales conference. DDI and Asset Insights GA target. August 2026. |
| IQ+ | Premium tier: IQ Actions + full agentic AI assistant. Paid via tokens. |
| FYI / FIA | "For Your Information" (diagnostic) / "For Immediate Action" (time-sensitive). IQ action classification system. |

---

*This document was generated from the IQ knowledge base as of 2026-05-27. For the most current state, query the wiki or check the IQ Product Planning spreadsheet in OneDrive.*
