---

## title: "MCP Server Strategy"

created: 2026-04-09
last_updated: 2026-05-21
source_count: 46
status: reviewed
summary: "Strategy for an official Infoblox MCP server exposing the full CSP product suite to external AI agents; primary distribution mechanism for IQ beyond the Infoblox UI."
type: strategy
tags: [mcp, iq, platform, gtm]

# MCP Server Strategy

Infoblox is building an official MCP (Model Context Protocol) server to make the full CSP product suite — UDDI/DDI, Threat Defense, Asset Insights, and Cubes — accessible to external AI agents and automation pipelines without requiring users to log into the portal. As of April 2026 this is a top executive priority, triggered by a directive from CPO Mukesh Gupta, with a CAB signal confirming customer demand is higher for agent-first access than for portal investment. Two internal implementations already exist; the work is to consolidate and productize them.

---

## Why This Exists

Today, agents cannot autonomously drive Infoblox. A customer workflow requiring a DNS record change or IP allocation goes: agent request → IT ticket → human admin → manual portal update → hours of delay. With an official MCP server it becomes: agent request → MCP tool call → CSP API → result returned in seconds.

**The competitive window is real.** BlueCat, EfficientIP, and SolarWinds IPAM have no MCP presence. Cloudflare, Stripe, GitHub, Shopify, and Zapier have already shipped MCP servers and are embedded in agent-driven pipelines. The software distribution channel is shifting from web → mobile app stores → LLM answers → **agent distribution (CLI/MCP/APIs)**, and Infoblox needs to be present in that channel.

**CAB validation (19 Fortune 100 attendees — Nordstrom, ServiceNow, American Airlines, Salesforce):** API-first access and MCP server ranked as a *higher* priority than portal investment. Use cases cited: Slack bots, ServiceNow agents, Splunk pipelines, CLI automation.

Market signals: 97M monthly MCP SDK downloads, 10K+ active MCP servers in production, Gartner forecasting 40% of enterprise apps will have AI agents by end of 2026.

---

## What It Is (and Is Not)

**What it is:**

- A lightweight semantic façade over existing CSP SaaS APIs, exposing read and write operations via MCP tools
- An alternate portal entry point — if a user can do it in the CSP portal, an agent should eventually be able to do it via MCP
- Provider-hosted by Infoblox at a stable URL; customers connect without any infrastructure setup

**What it is not:**

- A replacement for the CSP portal (account management, billing, user admin stay in portal)
- An LLM service — no provider-side LLM calls; external agents (Claude, ChatGPT, Glean, VS Code, Cloud Code) supply all intelligence
- A substitute for IQ Actions — the IQ Actions (DDI IQ Actions, SOC IQ Actions) are the irreplaceable differentiator and primary monetization vector; a plain MCP server plus an external agent cannot replicate them

---

## Current State (as of April 2026)

Three MCP implementations exist; none is an official Infoblox product:


| Implementation         | Status                   | Notes                                                                                                             |
| ---------------------- | ------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| **Igor's TME version** | Production with partners | Shared with HCL and Juniper Mist; no formal security review; public mirror by Sif Baksh on GitHub                 |
| **Cloud code version** | Internal prototype       | Built by engineering; differences vs. Igor's version uncharacterized as of Apr 24                                 |
| **Internal IQ MCP**    | Prototype                | Used by the CSP IQ agent internally; curated compact tool design; read-oriented with write operations in progress |


Reconciling the first two before any external release is a hard prerequisite. The risk is diverging behavior, security posture, and tool design producing confusing agent experiences.

---

## MVP Scope

### Phase 1 — In Scope

- **Read access** across all product families: UDDI/DDI, Threat Defense, Asset Insights, Cubes
- **Write operations** for high-frequency, low-impact use cases (DNS record creation, IP allocation, security policy updates) with explicit agent confirmation before execution
- **OIDC/OAuth authentication** — customers connect using their existing CSP identity; no API keys required; RBAC enforced automatically
- **Complete datasets** — full results, not 100-item capped responses (fixes the main reliability gap in current implementations)
- **Versioned tool design** — prevents breaking changes as the API evolves
- **Permissioning at product-family level** (UDDI vs Threat Defense vs Asset Insights) + read/write method-level controls
- **MCP off by default** — admin enables; write access requires a second, separate opt-in
- **Full audit log** — every action recorded (user, action, outcome, timestamp)
- **Structured error responses** — machine-readable so agents can respond without human intervention

### Phase 1 — Out of Scope

- Account management, billing, user administration
- Multi-environment context switching (sandbox/dev/prod) via a single MCP endpoint
- Granular per-operation or per-role access controls
- Multiple named MCP configurations per tenant
- NIOS-embedded on-prem MCP server form factor
- Provider-side LLM calls or internal IQ prompts

### Two Permission Layers (April 13 Design Session)

Two distinct permission layers were clarified in the April 13 MVP design session:

1. **User session permissions** — The user's effective CSP permissions travel with the OIDC session. MCP tools are scoped to what the authenticated user can actually do in the portal.
2. **Admin-level MCP resource masking** — Admins can create "MCP resource" endpoints with stricter controls than the user's RBAC allows (e.g., read-only override even for users who have write access). This is the primary blast-radius control for autonomous agents.

The simpler "user-permissions-only" model and the full "resource-based" admin masking model are the two product options — Phase 1 defaults to the simpler path with masks as a Phase 2 addition.

### Phase 2 (Deferred)

- Operation-level access controls (which specific API operations are permitted)
- Role and group-based access
- Usage visibility dashboards
- Configurable approval rule sets
- Multiple MCP configurations with different access levels per tenant

---

## Architecture

### Authentication

- **OIDC/OAuth standard flow** — CSP acts as OIDC provider; tokens reflect CSP identity and RBAC
- Callback URLs must unambiguously route to the correct tenant (`/oidc/callback/{customer-id}/...`) to prevent cross-tenant token leakage
- Per-tenant callback URL registration and validation in the CSP portal
- **Tenant-level masks / global policies** — admins can express "no writes via MCP" even if their users hold write permissions in CSP; this is the primary blast-radius control

### Tool Design

The team favors a **curated compact tool set** (matching the internal IQ approach) over a high-granularity many-tool design (Zscaler-style ~500 tools). The thesis is that curated tools lead to better agent reasoning, fewer failures, and less brittle prompting. A concrete task-set evaluation against both designs is pending (Turner).

Tools are the primary abstraction in Phase 1. Resources (mapping key UDDI entities) and Prompts (internal to IQ agent experience) are not exposed in the external server.

### Security

- Sensitive write operations require explicit agent confirmation before execution
- Dynamic tool pruning optional — tools can call effective-permissions APIs to hide operations the user cannot perform
- Full audit logging mandatory
- Anomalous usage patterns must be detectable via logging/monitoring

---

## Key Decisions


| Decision                                                      | Date         | Detail                                                                                    |
| ------------------------------------------------------------- | ------------ | ----------------------------------------------------------------------------------------- |
| Release official MCP server ASAP                              | Apr 7, 2026  | Mukesh Gupta directive: "do whatever is needed to make this happen"                       |
| Unified for all products (UDDI + NIOS)                        | Apr 7, 2026  | Single server, not per-product siloes                                                     |
| MCP included free with product purchases                      | Apr 12, 2026 | May charge separately later if platform load becomes margin issue                         |
| IQ Actions = irreplaceable differentiator, not IQ Assistant   | Apr 9, 2026  | Primary monetization; plain MCP + external agent cannot replicate IQ Actions              |
| Dashboards → AI-powered reporting, expose raw data for agents | Apr 9, 2026  | Mukesh: dashboards are becoming useless; raw data optimized for agents is the replacement |
| North Star: MCP as alternate portal entry point               | Apr 20, 2026 | Multitenancy and environment-switching deferred to later phases                           |
| Consolidate Igor's and cloud code versions before release     | Apr 24, 2026 | Use test cases to identify coverage gaps; reconcile before any productization             |
| Write operations intentionally limited in Phase 1             | Apr 28, 2026 | High-frequency, low-impact only; anything not in Phase 1 spec docs is out of scope        |
| MCP off by default; write requires separate admin opt-in      | Apr 28, 2026 | Autonomous agents carry different risk than manual UI; separate consent is required       |


---

## Stakeholders


| Name            | Role                                                                                     |
| --------------- | ---------------------------------------------------------------------------------------- |
| Mukesh Gupta    | EVP/CPO — executive sponsor; issued Apr 7 directive                                      |
| Jon Abbe        | PM coordinator — cross-functional working group, strategy, pricing framing               |
| Meenal Gupta    | Epic owner — IBIQ-1200 (Customer-Facing MCP Server)                                      |
| Hasmit Grover   | Engineering lead — OAuth status, implementation reconciliation, productization readiness |
| Turner Anderson | MCP design lead — competitor research, curated vs many-tool evaluation                   |
| Himanshu Raval  | Business case alignment, AWS Bedrock AgentCore assessment                                |
| Prasad          | Platform — OAuth story owner; blocking dependency for OIDC integration                   |
| Igor (TMA)      | External contributor — built the production-used TME version; multitenancy expertise     |
| Sif Baksh       | Ex-Infoblox — public GitHub MCP server (github.com/seefor/infoblox-uddi-ipam-mcp-server) |
| Manish Patel    | Business Development — AWS Bedrock AgentCore ISV coordination                            |


**Ownership model:** MCP is a platform-level investment, not per-product. Platform team owns integration and infrastructure; PM owns use case decisions (what to expose, how tools are described for agent discoverability, distribution strategy).

---

## Timeline & Blockers


| Milestone                                                                   | Target                 | Owner                      |
| --------------------------------------------------------------------------- | ---------------------- | -------------------------- |
| Inventory and demo both MCP implementations                                 | ASAP                   | Jon + Tom                  |
| Competitive research (Glean, Jira, ServiceNow, Zscaler, Palo Alto admin UX) | Next week from Apr 13  | Turner + Jon               |
| Draft PRD covering MVP scope, OIDC, permissioning, versioning, monetization | TBD                    | Product lead               |
| "Cisco-ready MCP" (API key, read-only) for Cisco Live                       | End of May 2026        | Turner/Cisco workstream    |
| Reconcile Igor's vs cloud code MCP implementations                          | ASAP                   | Hasmit                     |
| "OAuth-ready enterprise MCP" (client reg, server-side approvals)            | June–July 2026         | Turner/Tom/Platform        |
| IQ for DDI GA (incl. enterprise MCP)                                        | September–October 2026 | Contingent on PSEC signoff |


**Critical path blockers:**

1. **Platform OAuth readiness** (Prasad's story) — gates OIDC integration; also required for Cisco product launch end of May
2. **MCP implementation reconciliation** — Igor's vs cloud code; differences unknown; must be resolved before release
3. **Agent visibility** — agent must see user screen context as MCP capabilities are modularized; design unresolved
4. **Approval/confirmation workflow design** — write operations require explicit confirmation; UX pattern not yet defined (see Approval Workflows section below)

---

## "Cisco-Ready MCP" vs "OAuth-Ready Enterprise MCP" (May 1, 2026)

The May 1 PM leads sync established a critical product split for the MCP launch:

**"Cisco-ready MCP" (end-of-May, Cisco Live):**

- API-key authentication — Cisco is not yet ready for OAuth; Infoblox meets this request
- Read-only, limited tool scope
- For Cisco AI Canvas demo showing cross-vendor interoperability (Cisco MCPs ↔ Infoblox MCP)
- Explicitly below Infoblox's long-term enterprise security bar; labeled as special-case partner configuration
- No broad customer enablement at this release; Cisco-only and selected internal accounts

**"OAuth-ready enterprise MCP" (follow-on, June/July):**

- OAuth with proper client registration, scopes, and refresh tokens
- Per-client tool visibility: each client (Cisco UI, CSP portal, internal consoles, custom agents) registers with MCP server; RBAC determines which tools are visible and read vs. write per client
- Server-side workflow approvals (see section below)
- This is the real enterprise-grade offering that becomes the standard for all commercial customers

**Client registration model:**

- Single MCP endpoint; capabilities tailored per registered client via RBAC
- Cisco, CSP portal, internal UIs, and future partner channels all see different tool sets through the same server
- Near-term: reuse existing RBAC admin pages to enable/disable MCP tools and set read/write per role — no separate MCP admin console needed for June/July window

**Competitive context:** This release is driven by BlueCat's MCP + AI assistant announcement and Cisco Live visibility requirements. The strategic framing: end-of-May announcement is marketing noise to leverage; the differentiated product ships over the following 4–6 months.

**⚠️ Updated BlueCat competitive intelligence (May 1, 2026):** BlueCat announced on April 30/May 1 that its MCP Servers will reach **GA via public registry in July 2026** (currently in tech preview) and will be included with base product licenses. Their LiveAssist AI virtual engineer also expands to **DDI starting July 2026**. This narrows the window — Infoblox's external MCP Server launching in Limited Availability alongside IQ+ Security GA in July is now directly racing BlueCat's DDI+MCP GA. Infoblox's enterprise-grade Internal MCP server has been operating IQ for months, providing significant head start in tooling quality, but the market-visible launch timing is converging. Internal assessment: BlueCat's press release has "much hoopla and fuzziness with not much substance" and lacks real customer endorsements; however, Greg Delaney (Sr. Product Marketing) flagged this needs a proper field competitive response. See [[IQ-Strategy]] Competitive Intelligence section.

[Source: 2026-05-01_Turner_Himanshu_Jon_weekly_IQ_Sync.md, 2026-05-01_Reconnect_on_legal_TCs_for_Infoblox_IQ.md, wiki/summaries/re-_bluecat_announcement.md]

## IQ Actions API Access via Published APIs and MCP Server (IBIQ-1417/1416)

Jon Abbe created Epic IBIQ-1417 and Story IBIQ-1416 on May 11, 2026 to expose IQ Action APIs through Infoblox's published (external) API surface and the external MCP server, gated by entitlement-based access control.

**What this means:** External agents and customers with the appropriate IQ entitlement will be able to invoke IQ Actions via the same MCP channels used for UDDI/DDI read operations. This is the first step toward making IQ's action-triggering capabilities accessible to external workflows (ServiceNow agents, Slack bots, third-party SIEM pipelines) rather than requiring them to go through the IQ portal.

**Entitlement gating:** Only customers with an IQ license will be able to access these endpoints — protecting the premium Actions monetization tier while opening the interface to entitled agents.

**Assigned:** Meenal Gupta (story). Targeted at 26.4 scope.

This closes the gap between the MCP server (currently read/query-focused) and the IQ Actions framework (currently only accessible through the IQ portal). See also [[IQ-Strategy]] IQ Actions Framework section.

---

## Approval Workflows (April 30, 2026 Design)

From the Apr 30 Arch/PM sync, the approval workflow design was clarified:

**Three behavior modes:**

1. **Read-only** — no write operations permitted; safe for exploratory/diagnostic agents.
2. **Approval-required** — MCP server presents an approval form describing the proposed change; user must explicitly click Approve before execution.
3. **Auto-approve** — for trusted automation / headless agent flows where policy permits.

**Client capability detection:**

- If client declares "human is present," MCP sends approval forms for each write operation.
- Purely automated agents (no human in the loop) can request auto-approval where policy allows.
- Example: "add bad-domain.com to block list" → approval form shows domain, block list ID, scope, and effect summary for user confirmation.

**Implementation phasing:**

- Phase 1: Client-side approval (rendered in CSP/IQ UI).
- Phase 2: Server-side, policy-driven approvals (enforced at policy service level; works for non-interactive paths too).

**May 1 update — server-side enforcement requirement:**
The May 1 PM sync upgraded server-side approval enforcement from Phase 2 to a near-term requirement for the "OAuth-ready enterprise MCP." Rationale: Infoblox sells highly sensitive enterprise infrastructure; security posture must be consistent regardless of which client or custom agent invokes the action. The desired flow:

- Rich clients (cloud desktop, CSP, first-party UIs): approval form renders inline in chat
- Simple/custom clients: IQ returns a secure approval URL (`csp.infoblox.com/approval/...`) for user to approve/deny out-of-band

This satisfies MCP spec (which leaves "good UX for authorization" to implementers) while ensuring server-side enforcement is non-negotiable. [Source: 2026-05-01_Turner_Himanshu_Jon_weekly_IQ_Sync.md]

[Source: 2026-04-30_IB_IQ_ArchPM_sync_up.md]

---

## Agent Identity and Authorization (Design — April 30, 2026)

**Current state:** Only user identity exists. Agents act entirely as the authenticated user — agent's effective rights equal the user's full RBAC. No separate agent principal.

**Planned model:** Introduce an **agent principal** — a synthetic agent account that records "agent X acted on behalf of user Y" in audit logs and authorization decisions.

**Permission intersection:** Effective rights for an agent-initiated action = (user permissions) ∩ (agent permissions). This allows admins to constrain agents even when the user holds broad privileges.

**Admin controls needed:** Per-agent capability configuration before enabling MCP in production (e.g., "this agent may modify block lists but not DHCP scopes or core IPAM objects").

**Release gate tension:** Team has not declared agent identity a strict gate for the initial MCP release. Shipping with user-only identity is acceptable short-term but not as a long-term posture — admins have very limited control over agent behavior beyond standard user RBAC.

[Source: 2026-04-30_IB_IQ_ArchPM_sync_up.md]

---

## Audit Logging (April 30, 2026)

**Current behavior:** Backend APIs (DDI and others) already create audit log entries for every mutation (create/update/delete). MCP invokes these APIs and never bypasses them, so MCP-triggered changes are audited identically to UI or direct-API changes.

**Current gap:** CSP audit logs (Logs → Audit tab) are confusing to navigate:

- Non-obvious resource type taxonomy and filter names
- Customers can't easily search by domain name (e.g., "bad-domain.com" to find a block list change)
- No distinct agent principal visible today; changes appear attributed only to the user account

**Planned improvement:** Once agent identity exists, audit records will show both end user and synthetic agent account. UX improvements (better search, labeling, resource naming filters) are needed so operators who live in Datadog/ServiceNow can still trace IQ/agent-originated changes in CSP.

**Descoped from current MCP story:** Additional audit logging work pulled from MCP scope; existing backend logging is sufficient; focus shifts to identity and approvals.

[Source: 2026-04-30_IB_IQ_ArchPM_sync_up.md]

---

## Platform/Process Issues (April 30, 2026)

- **Duplicate Jira epics:** Stories being cloned TP → Jira → IQP, creating diverging versions of the same work (e.g., DHCP scope optimization exists in both main Jira and IQP project).
- **Rushed TP migration:** Drop of TargetProcess licensing forced a rushed Jira migration with minimal automation; misconfigured tickets and parallel backlogs resulted.
- **Contributor overload:** Many external teams want to contribute to MCP, but without stable contribution model and clear ownership, coordination overhead slows the core team (Mythical Man Month dynamic).
- **Foundation needed first:** Contributor docs, Jira cleanup, and ownership boundaries required before broad MCP development opens to cross-team contributors.

[Source: 2026-04-30_IB_IQ_ArchPM_sync_up.md]

---

## Revenue & Pricing Model


| Component                  | Pricing                                                    | Rationale                                                                                |
| -------------------------- | ---------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| MCP tools (API proxies)    | **Free** — included with product licenses                  | Customers already pay for CSP; usage covered by management tokens                        |
| IQ Actions (DDI + SOC)     | **Premium tier** — primary monetization                    | Cannot be replicated by external agent + basic MCP; Infoblox's irreplaceable value       |
| IQ Assistant UI            | Low priority for non-advanced users                        | Not primary monetization target                                                          |
| Future: IQ-as-MCP-tool     | Separate charge — agent token usage + premium capabilities | Long-term; exposes IQ agent itself as a callable MCP service                             |
| Platform load relief valve | Usage-based charges or limits if needed                    | Included for now; Mukesh: "may charge separately later if it creates SaaS platform load" |


---

## Agent Distribution Platform — The Bigger Picture

The MCP server MVP is phase one of a larger **FY27 Agent Distribution Platform** investment. The thesis: Infoblox must become accessible to AI agents as infrastructure, not just as a product customers log into.

**The investment case:** If done at platform level, every product becomes agent-accessible automatically. This requires:

1. Complete API coverage (not just portal-mirroring)
2. Machine-readable authentication (OIDC/OAuth without browser)
3. CLI with JSON output
4. OpenAPI 3.0 documentation
5. AGENTS.md or equivalent platform description for agent discoverability

**Long-term architecture:**

- Unified MCP server for all CSP APIs (provider-hosted, multi-tenant)
- Separate NIOS-embedded MCP for on-prem (NIOS or customer IdP as OIDC provider; keeps traffic local)
- IQ agent exposed as a callable "agent service" — monetized separately on top of basic MCP

---

## AWS Bedrock AgentCore

AWS offered an ISV Solution Acceleration engagement (via Loka as implementation vendor) with four SOW options: Tracing, Digital Employee, Anomaly, and Agent for DNS Outages. As of Apr 13, 2026:

- Hasmit Grover is skeptical — last quarter's Loka engagement delivered little value; considering skipping entirely
- Himanshu Raval is interested in validating architecture leveraging AgentCore services, but Loka confirmed scope is limited to Jupyter notebooks for PoC (not the broader architecture diagram)
- Decision pending: internal alignment needed before accepting or declining

This does not block MCP server MVP but represents a parallel infrastructure bet on AWS's agent strategy.

### Integration Monetization Models — Under Consideration

As third-party integrations (Cisco, ServiceNow) enter coordination, a question has surfaced: should Infoblox charge for access to pre-built connectors, or capture value further up the stack? Four models are in consideration (raised by engineering, Apr 2026):


| Model                                            | What You Charge For                                                                                                     | Strength                                                                                   | Risk                                                                                                                 |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| **1. Paywall the Integration**                   | Access to pre-built connectors (Cisco, ServiceNow, etc.) requires tokens                                                | Directly monetizes Infoblox engineering effort; simple to explain                          | Slows ecosystem adoption; Cisco/ServiceNow may resist being behind a paywall; engineers find workarounds             |
| **2. Free Integrations, Charge for Consumption** | All connectors free; token burn on every agent action (query, update, remediation)                                      | Maximizes adoption; aligns cost to value delivered; maps to IQ's existing token model      | Customers may feel nickel-and-dimed on automated workflows at scale; harder to forecast spend                        |
| **3. Free Plumbing, Charge for Intelligence**    | MCP + connectors free; charge for IQ's AI layer — analysis, recommendations, agentic decisions                          | Positions Infoblox as intelligence provider, not middleware vendor; hardest to commoditize | Harder to meter; requires clean separation between raw connector output and IQ-enriched output                       |
| **4. Partner-Funded (Ecosystem Play)**           | Cisco/ServiceNow pay Infoblox a platform fee or rev-share for certified integration status; customers pay nothing extra | Removes end-customer friction entirely; creates partner incentive alignment                | Requires Infoblox to have sufficient distribution leverage to charge partners; may not be realistic at current scale |


**Current lean:** Models 2 or 3 fit IQ's positioning better than Model 1. Paywalling connector *access* creates adoption drag during ecosystem formation. Charging for *consumption* (Model 2) or *intelligence* (Model 3) scales better. Model 4 becomes viable once Cisco coordination matures and Infoblox can credibly claim to be the agent distribution channel Cisco wants access to.

> ⚠️ This is unresolved — flagged in Open Risks below. Alignment needed between PM (Jon), Himanshu, and Mukesh before PRD is finalized.

---

## Three-Tier Response Architecture (April 30, 2026)

An internal email thread (FW: IQ Feedback from Brent @ Viasat) surfaced a key architectural insight about how MCP should be positioned within IQ's response pipeline — not as a generic additional source, but as a **targeted validation layer for navigation queries**.

The current problem: IQ routes navigation, explanation, and troubleshooting queries through too similar a response path, even though they should be grounded differently. This is why IQ confidently returns stale UI paths — navigation answers are being grounded in static docs that lag the portal.

**Proposed three-tier grounding model:**


| Question Type       | Ideal Grounding                                                         |
| ------------------- | ----------------------------------------------------------------------- |
| **Navigation**      | Validated against live portal via MCP — self-updating, no docs required |
| **Explanation**     | Curated docs — authoritative, stable content remains primary            |
| **Troubleshooting** | Live product state and IQ Actions — real-time data                      |


**Design principle:** MCP as a *validation layer* for navigation, not just another retrieval source. This can be introduced incrementally without replacing the existing docs-based path for explanation queries.

**Two complementary tracks (not mutually exclusive):**

1. Docs team keeps explanation and configuration content accurate
2. MCP server handles live portal introspection so navigation guidance is always current as the UI evolves

This framing also surfaces a retrieval freshness control need: IQ should differentiate which sources it grounds different query types in, rather than treating all retrieved context uniformly.

[Source: 2026-04-30_fw_iq_feedback_brent_viasat.md — internal discussion between Jon Abbe, James Zhu, Himanshu Raval]

---

## Six Cross-Cutting Design Dimensions (April 27, 2026)

The April 27 Jon/Himanshu catchup (separate from the April 13 MVP session) enumerated six design dimensions that must be resolved before engineering begins:


| Dimension                           | Key Decision                                                                                                                                                  |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Authentication & access control** | Auth mechanism (OIDC/tokens/feature flags tied to entitlements); alignment with RBAC so admins simply "enable MCP access"                                     |
| **Scope & permissions**             | Read-only vs read/write tool mapping; admin-level MCP configuration page mirroring portal RBAC                                                                |
| **Data residency & hosting**        | Infoblox-hosted preferred for cloud; NIOS/non-cloud may need customer-hosted or CSP-downloadable variant                                                      |
| **Versioning & updates**            | How MCP is versioned, upgraded, distributed (central endpoint vs downloadable); non-breaking upgrades for paying vs free customers                            |
| **Audit, security & logging**       | Mandatory audit trail for every AI/tool invocation; integration with existing IQ and SOC logging processes                                                    |
| **Discovery & distribution**        | Where customers find and activate MCP (CSP portal, Infoblox portal, other channels); free vs paid customer entry points; default: MCP off until admin enables |


**Entitlement gating model:** Conceptually a single MCP endpoint; what each customer sees is controlled by entitlement and feature flags. SOC Insights, IQ Actions, and specific asset capabilities are exposed only to entitled customers. [Source: 2026-04-27_IQ_Platform_AI_Infra_Deliverables_Discussion_JonHimanshu_catchup.md]

---

## Cisco Project Luna — Legal Concern (May 7, 2026)

Ken Lancaster (BD/Legal) flagged a legal concern with the Cisco Project Luna Tech Partner Agreement, which contains a broad IP license grant from Infoblox to Cisco:

> "Partner grants to Cisco a non-exclusive, worldwide, transferable, sublicensable, fully paid-up, royalty-free license to host, link to, reproduce, modify, display, test, distribute, and make available the Integration; and to perform, use and access the Integration for administration and demonstration purposes."

**Legal's questions:** How will Infoblox offer MCP/LLM access to customers and partners once a Production MCP is available post-GA? How will Infoblox limit what another vendor's application can and cannot do within its product? Stacey from Legal invited to the May 9 call with Cisco.

**Himanshu's response (May 7):** The Cisco-bound MCP Server version is not monetized by Infoblox; the agreement covers a simpler auth implementation (API key, not full OAuth); Infoblox has no unique IP in the MCP Server (confirmed question to Turner Anderson and Tom Hayward). MCP servers built by ex-employees and TMEs are already in the wild (including one given to Juniper Mist). This broad legal language may be common to other integration agreements; recommend legal evaluate.

**Post-Cisco plan:** Release the Infoblox MCP Server via the SaaS portal to customers after the Cisco engagement.

**Multiple Cisco initiatives in parallel (May 8 sync):** Thousand Eyes, Meraki, Luna (Cisco AI Canvas MCP demo for Cisco Live May 31), and IQ Launch. Rishabh Parmar organized alignment meeting.

[Source: 2026-05-07_Weekly_IQ_Platform_Sync_up.md, Re-_Sync_on_Cisco_initiatives.md]

---

## Naming Decision — "Infoblox MCP Server" (May 9, 2026)

A naming discussion confirmed the official product name should be **"Infoblox MCP Server"** — not "IQ MCP Server."

**Rationale:** IQ is only one dataset (a premium layer) routed through the MCP server. The MCP server exposes the full Infoblox API surface (DDI, Threat Defense, Asset Insights, IPAM). Naming it "IQ MCP Server" creates a false scope impression and undermines discoverability.

**Precedent:** Vendor-first naming is the standard pattern for discoverability — analogous to "Cloudflare MCP Server," "Atlassian Jira," etc. LLMs and agent catalogs search by vendor/product name; "IQ" is not the vendor.

**Entitlement model:** IQ Actions data is accessible only to entitled customers via RBAC. Customers who haven't subscribed to IQ won't receive IQ Actions data via the MCP server — the server is vendor-branded but the data remains entitlement-gated.

**Security note:** Primary enforcement and visibility should be at the endpoint, not solely at DNS or network choke points. Endpoint monitoring captures user interactions with AI tools and data use more completely.

**Action:** Brad and Mukesh to be informed of naming rationale. PR/messaging team to update materials accordingly.

[Source: 2026-05-09_Infoblox_MCP_Server_Naming.md]

---

## M1 Phase 1 Scope — Confirmed (May 8, 2026)

The May 8 bi-weekly product prioritization meeting confirmed the customer-facing MCP server Phase 1 scope for M1 (May):

- **Phase 1 (May):** API key-based, read-only access sufficient for initial external scenarios and Cisco Live demos. Not a full enterprise-grade release.
- **Phase 2 (June):** AAUTH integration and Tom's architectural requirements, making the service hardened and suitable for broader production roll-out.

The group agreed that seeing later AAUTH and architectural requirements in the same epic should not cause confusion — the two phases are clearly distinct.

[Source: 2026-05-08_IQ_Bi_weekly_Product_prioritization_for_next_Sprint.md]

---

## Sprint 9 Scope Update — Cisco Live / Full June Release (May 12, 2026)

The May 12 sprint grooming session introduced a more explicit two-track structure for MCP:

### Cisco Live Minimal Scope (Late May / Early June)

- Use the existing MCP server (already backing the IQ agent) as a demo-ready, externally callable endpoint
- API key-based access only; external agents connect and invoke network/security APIs
- Accept a "jerry-rigged" configuration: simplified RBAC, explicit Cisco Live account configuration, temporary access rules adequate for the event
- Document how to obtain and manage API keys from the portal, how agents connect, which APIs are exposed for Cisco Live, and what is demo-only vs. long-term
- **Alex** owns the documentation story
- Turner already owns a story ensuring the MCP gateway exposes only the intended public tools; this proceeds in Sprint 9 regardless of MCP epic status

**Note on dates:** Internal expectations that all Cisco Live work must be done by May 15 were acknowledged as unrealistic. Cisco Live runs May 31–June 4; realistic delivery reanchoring is underway.

### Full June Release (General Availability)

- Proper RBAC and user-access toggles (MCP on/off toggle in user access UI)
- Enforce business rules: MCP off by default; first enablement = read-only only; write operations require a second explicit opt-in
- UX stories for the first-time enablement flow (currently missing from the epic; blocks implementation commit)
- Manish adds stories under the main MCP epic for RBAC, user-access toggles, and long-term enablement model

**Epic status:** The main customer-facing MCP epic is back in **Reviewing** under Tom until requirements align with engineering expectations (either add UX for first-enable flow or remove that requirement from v1).

[Source: 2026-05-12_IQ_Bi_Weekly_Sprint_Grooming.md]

---

## A2A vs MCP — Integration Pattern Distinction (May 8, 2026)

The May 8 Turner/Himanshu/Jon sync clarified two distinct integration patterns for partner and ecosystem positioning:


| Pattern | Description                                                                                                                         | Ownership      |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------- | -------------- |
| **MCP** | External vendors or customers build agents on top of IQ; they own reliability and behavior                                          | External party |
| **A2A** | Infoblox-owned agents collaborate with external agents as peers; agent identity (certificate-based), IAM, observability, audit logs | Infoblox       |


**Distinction matters for partners:** MCP is "you build on us"; A2A is "our agent works with your agent."

**Current focus:** Q1 priorities remain MCP server, webhooks, and ServiceNow integration for specific partner demos. A2A is directionally where the industry is heading and is largely within reach on the current stack with modest conversation service extensions.

**ThousandEyes scenario:** External agents like ThousandEyes detect latency anomalies and emit events into IQ's event stream; the conversation service routes these to the Interaction Agent for investigation. This is the near-term A2A pattern being designed.

[Source: 2026-05-08_Turner_Himanshu_Jon_weekly_IQ_Sync.md]

---

## On-Prem MCP — Deferral Confirmed (May 7, 2026)

The May 7 weekly IQ Platform sync formally confirmed that on-prem MCP for IOS customers (non-air-gapped NIOS) is deferred to approximately Q1 FY27. This deferred track is distinct from the NIOS MCP needed for Maryland/air-gap deployments. The SaaS MCP server is the sole focus for the end-May launch and the Cisco Live announcement.

[Source: 2026-05-07_Weekly_IQ_Platform_Sync_up.md, iq-update-may-7-2026.md]

---

## EAP Launch — Customer-Facing Confirmation (May 14, 2026)

At the May 14, 2026 EAP office hour, the product team publicly confirmed the MCP server EAP timeline to customers:

- **EAP availability:** approximately 4 weeks from May 14 (~June 2026)
- **Authentication:** OAuth-based connectivity
- **Write operations:** gated behind server-side approval workflows; human-reviewable approval widgets or secure portal links for each configuration-changing action
- **Read operations:** accessible per user's RBAC authorization without additional approval
- **Cisco AI:** confirmed as the first cross-vendor partner using the MCP server; announcements expected at Cisco Live

This EAP targets customers who want to use their own AI tools or internal agent frameworks to query IQ data, receive analytics, and trigger tightly controlled change workflows under server-side enforcement.

[Source: 2026-05-14_IQ_Product_Office_Hour_Whats_New_Whats_Next.md]

---

## RBAC and Auth Design — Finalized (May 14, 2026)

The May 14 MCP TSA session produced the most detailed auth/RBAC design to date, adding specifics that supersede earlier general descriptions. Two distinct release moments were formalized:

### Cisco Live (Late May / Early June) — Demo Only

- Use existing MCP server + CSP APIs + service API key (pasted into MCP client config)
- Single/few tenants; limited configuration for the demo event
- No RBAC, OAuth, or customer-ready UX required
- Explicitly labeled as marketing/demo phase only — not the customer release baseline

### Customer GA (Mid/Late June, Possibly July) — Phase 1

**Authentication — Dual support:**

- **Service API key:** Created in CSP portal UI; stored in MCP client config file; for bots, automation, and early adopters
- **OAuth/OIDC (preferred):** Okta-backed OIDC; user authenticates via Okta sign-in widget; headless callback to agent URL (not to `csp.infoblox.com`); tokens used by agent without placing keys in local config

**Authorization — MCP gateway as proxy:**

- Gateway extracts tool name and "destructive" flag from MCP request body
- Synthesizes logical endpoint (`mcp.read` or `mcp.write`) for Aussie middleware
- Parks configuration maps synthesized endpoints to permission labels
- Two-layer check: (1) Gateway RBAC — caller must have MCP role to proceed; (2) Downstream RBAC — service enforces its own permissions
- Destructive tools blocked at gateway for MCP Read users (HTTP 403 before MCP server reached)

**Role model:**

- `MCP Read` — non-destructive operations only; `mcp.read` synthesized endpoint
- `MCP Write / Manage` — destructive + configuration operations; `mcp.write` synthesized endpoint
- Roles added via standard CP identity migration; appear in RBAC UI like all other CSP roles; visible to all tenants once migration applied

**Phase 1 out of scope:** Dynamic client registration, service-principal/OBO tokens, per-tool permission labels beyond coarse Read/Write

### Phase 2 (Enhanced OAuth + Connector Marketplace)

- OAuth 2.1 best practices: PKCE, standard `.well-known` discovery, dynamic client registration
- Required for Anthropic/Claude connector library listing (API-key-only flows not eligible)
- Connector UX: user finds "Infoblox" in connector library → Okta sign-in → headless callback → Infoblox appears as connected account in agent UI
- Pre-listing: "Integrator Guide" documents manual setup (discovery URL, scope config) before marketplace listing

### vNext (Phase 3+)

- **Service principal + OBO token minting:** Agent assigned service principal; gateway computes intersection of user and agent permissions; mints constrained OBO token; downstream APIs receive only permitted subset of user's rights
- **Instance-level guardrails:** Admins restrict agents to specific networks/objects using existing access views

### Key Decisions (May 14)

- MCP will not ship to customers without MCP-specific RBAC at the gateway and proper roles/groups in CSP
- Cisco Live treated as demo only; Phase 1 is the real customer baseline
- Phase 2 OAuth required before Anthropic/Claude connector marketplace listing
- OBO tokens and service principals = vNext; not required for initial GA

[Source: 2026-05-14_MCP_Server_TSA_MCP_Release_Plan_and_RBAC.md]

---

## Legal Review Requirements — Pre-GA (May 2026)

A May 18, 2026 legal alignment session produced requirements that must be satisfied before MCP server customer GA.

**Olga Phillips (Global Counsel, AI/Product/Privacy) pre-GA requirements:**

- Deep dive on MCP server architecture and security — Olga wants to understand the full technical picture before legal documentation is drafted. This is not gating EAP but is required before GA.
- Questions added to the "L&C IQ Stuff" working document (shared with Jon Abbe).
- Legal documentation work occurs before GA, not before EAP.

**MCP server and IQ T&Cs — key position:**
Jon Abbe and Mehul Patel aligned May 13: the MCP server should **not** be gated behind IQ-specific supplemental T&Cs. The MCP server is a general API-layer capability available across all Infoblox products — not an IQ-exclusive feature. It should follow the same T&C model as the existing API, with no additional IQ-specific acceptance required.

This means:

- Customers who use MCP server but have not purchased IQ should not be required to accept IQ T&Cs.
- IQ data returned via MCP is entitlement-gated; the T&C model for access follows the API/product-specific entitlement, not a separate MCP wrapper.

**NIOS MCP Server path:**

- NIOS MCP Server GA: October 2026 (confirmed Olga, May 18)
- This is a separate track from IQ for NIOS (cloud telemetry). NIOS MCP server operates on-prem; it does not require the IQ telemetry pipeline or the new PII-covering T&C addendum.
- Seller pitch: NIOS MCP Server is an on-ramp that does not require telemetry consent. IQ for NIOS requires consent and the new data.

[Source: 2026-05-18_legal_requirements_before_ga_infoblox_iq.md]

---

## Tenant-Per-Session Architecture (May 18, 2026)

A platform workshop on May 18 produced a confirmed architectural decision for how MCP and agent sessions handle multi-tenant users.

**Problem:** A single authenticated CSP user may have access to 5+ tenants. Without an explicit selection step, an authorized agent could gain implicit jurisdiction over all accounts linked to that user.

**Decision: one tenant per agent/LLM session.** Before an agent "starts its journey," the agent session must be bound to a single concrete CSP account/tenant. This is enforced consistently across:

- CSP-to-Azure flows (account pre-selected on CSP side)
- Azure SSO-to-CSP/MCP flows (explicit selection step required after Okta auth)
- External agent (MCP) flows (agent detects multiple tenants, prompts user to narrow to one)

**Why this matters for LLM trust:** Long-running conversations where users switch accounts cause interleaved data from multiple tenants in the LLM context window, increasing risk of cross-tenant data leakage and hallucinated cross-tenant relationships. One account per session eliminates this by ensuring all prompts, tool calls, and results in a session belong to a single tenant.

**Switching tenants:** User starts a new session rather than switching mid-conversation. Any mid-session switch would require re-authentication and fresh token mint — the complexity is not worth the minor UX convenience.

**Token/JWT strategy:** Selected tenant must be embedded as a claim in the agent JWT. For initial authorization, the user's default account may be used; after explicit user selection, a fresh token is issued with the selected tenant claim.

**Enabling work required (PTCI-4674):** A new RBAC story for MCP Server Role-Based Access Control was created May 19, 2026. Separate from this, a "role in groups" permissions feature story is needed as a downstream dependency for TSA and agent authorization work. Tenant listing APIs and session-binding endpoints also need design.

[Source: 2026-05-18_Platform_Workshop_MCP_Server_tenant_selection.md]

---

## Cisco Live — Cross-Vendor MCP Interoperability Demo (May 2026 Update)

IBIQ-1200 (Customer-Facing MCP Server) updated May 19, 2026: the Cisco Live demo (May 31/June 2) is explicitly a **cross-vendor MCP interoperability demo** — Cisco MCPs and the Infoblox MCP calling each other within the Cisco AI Canvas environment. This is not a standalone Infoblox demo but a joint cross-vendor demonstration.

Multiple Cisco workstreams are running in parallel: ThousandEyes, Meraki, Luna, and IQ Launch.

Pricing: MCP access is included with existing product licenses at no additional charge at launch. If platform load or margin becomes a concern, usage-based charges or limits may be introduced. The primary paid tier remains IQ Actions.

[Source: IBIQ-1200 via Glean, 2026-05-19; 2026-05-18_Jon_Himanshu_catch_up.md]

---

## IQ Architecture Confirmed — Full Technical Brief (May 20, 2026)

Tom Hayward (IQ architect, team-iq) provided a comprehensive architecture walkthrough to Olga Phillips (Global Counsel, AI/Product/Privacy) on May 20 as part of legal pre-GA review. This is the authoritative technical description of IQ's architecture. 8 architecture PDFs were attached (IQ-Architecture-Overview, IQ-Architecture-Detailed, IQ-Service-Agent, IQ-Service-MCP-Gateway, IQ-Service-MCP-CSP, IQ-ADR-027-LiteLLM-Gateway, IQ-ADR-042-ExtAuthz-Bridge, IQ-Customer-Facing-MCP-State).

### IQ Has 4 Distinct Surfaces


| Surface                                    | Description                                                                                                                  | Owner                      |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| **IQ Assistant (in-portal chat)**          | Conversational agent calling Infoblox APIs via MCP tools                                                                     | team-iq                    |
| **Customer-facing MCP server**             | Same MCP Gateway exposed externally; customers connect own AI clients (Claude Desktop, IDEs, scripts) using portal API keys  | team-iq                    |
| **IQ for DDI / NIOS insights (Insighter)** | Telemetry-driven RCA via Insighter Research Agent; recommendations only; no API tool calls; humans approve/dismiss in portal | team-iq                    |
| **IQ for Threat Defense**                  | Separate implementation owned by TD team; does not use Insighter Research Agent                                              | TD team (Shadid Chowdhury) |


**Customer-facing MCP server is already in production today** — quiet, unadvertised, read-only. Customers can connect using existing Infoblox Portal API keys.

### Two MCP Servers Backing IQ


| MCP Server        | Purpose                                                       | Backend                     |
| ----------------- | ------------------------------------------------------------- | --------------------------- |
| **MCP-CSP**       | Infoblox Portal APIs (UDDI, Threat Defense, Assets, Platform) | CSP REST APIs, Atlas CubeJS |
| **MCP Insighter** | Analytics / telemetry queries                                 | ClickHouse                  |


Both exposed through the centralized MCP Gateway. Tool inventories declared via MCP tools/list protocol.

### DESTRUCTIVE_NOOP — Production Safety Mechanism (Current State)

MCP-CSP runs with `DESTRUCTIVE_NOOP=True` in production. Write tool calls (make_post_request, make_put_request, make_patch_request, make_delete_request) return API usage instructions describing what would happen — no mutation against the customer's tenant. Read tools operate normally. This is the production safety mechanism for both IQ Assistant and the customer-facing MCP.

The caller's JWT (Identity Service / Okta for external clients; cluster identity for the IQ Assistant) propagates to the downstream CSP API, which enforces its own RBAC against the caller's identity.

### LiteLLM Gateway (ADR-27)

All LLM requests go through LiteLLM on the shared services EKS cluster — this covers both IQ Assistant and Insighter Research Agent. LiteLLM is the only path to Azure OpenAI. Provider keys in HashiCorp Vault; traffic via AWS PrivateLink (no public internet). Automatic failover between Azure regional endpoints (US-East-1 primary, US-East-2 failover). Separate LiteLLM instances for EU and GCP are planned but not deployed.

### Data Storage


| Data                                      | Storage                                            | TTL                    |
| ----------------------------------------- | -------------------------------------------------- | ---------------------- |
| Conversation history                      | PostgreSQL/RDS (NLP Interface)                     | 30 days                |
| MCP-CSP session cache (tool call results) | AWS S3 as Parquet, tenant-scoped                   | TBD — to be documented |
| Engineering traces                        | Databricks via MLflow (async, not customer-facing) | TBD — to be documented |


### GA Blockers (Technical + Legal)

**Technical — in flight:**

- **IBIQ-1193**: MCP Gateway RBAC via ext_authz + OPA (ADR-42) — design complete, rollout in flight
- **IBIQ-926 / PTCI-3285**: Agent role + platform permission model — same milestone
- **IBIQ-648**: Approval Workflow — PI 26.3 target
- **IBIQ-1228**: Approvals Service core — PI 26.3 target
- **IBIQ-958**: "Always Allow" delegation for repeatable low-risk operations
- **IBIQ-40**: Identity as OAuth 2.1/OIDC provider for MCP — P1, **blocked in Platform; largest GA-marketing blocker for customer-facing MCP**

**Legal/procurement (must be addressed by Olga/legal):**

1. **Add Azure OpenAI, Microsoft Azure, and Databricks to the public subprocessor list** (infoblox.com/company/legal/data-subprocessors-list/ — last updated 12/2025; only AWS and Okta are listed). The 30-day customer notification clock for IQ for TD starts when these are posted. This is the most concrete pre-GA blocker.
2. Azure OpenAI enterprise data-processing terms confirmation (no retention, no training, Modified Abuse Monitoring opt-out)
3. Databricks data-processing agreement confirmation
4. Customer-facing T&C and addendum language

### Security Testing

Product Security pen test complete: April 2026 (202604_Infoblox_IQ_Final_Report.docx from PSE/Red Team). Per-service unit tests, integration tests for MCP Gateway auth path, continuous synthetic monitoring in production, MLflow evaluation pipeline for offline regression testing.

[Source: Re- Legal requirements before GA of Infoblox IQ 2.eml, 2026-05-20 — wiki/summaries/re-_legal_requirements_before_ga_iq_2.md]

---

## IQ for DDI — Naming Confirmed (May 20, 2026)

Per Mukesh, the official product name is **"IQ for DDI"** — chosen to cover both NIOS and UDDI/SaaS deployments. For the June 2 announcement PR and blog, messaging uses "Infoblox DDI" (umbrella term) rather than UDDI or NIOS specifically. DNS-AID is not mentioned in the announcement. Two FAQ documents (Les Dunston technical FAQ and Mehul Patel sales FAQ) need consolidation before announcement.

[Source: Re- Bluecat Announcement 2.eml, 2026-05-20 — wiki/summaries/re-_bluecat_announcement_2.md]

---

## Internal vs. GA Scope — Messaging Alignment Needed (May 18, 2026)

Jon and Himanshu identified a recurring confusion pattern: some PMs (specifically Anand) conflated their desire to build internal apps and customer-facing skills with GA readiness, creating premature or inaccurate external messaging about MCP availability.

**Clarified position:**

- MCP server is currently available only via internal APIs; teams can build internal experiments and sales tools
- Enterprise GA expected June–July but must pass infosec scrutiny, documentation review, and operational-readiness checks — not just "feature complete"
- Internal apps and skills are fine as long as they are not represented as external GA and do not create support expectations

**Skill ownership policy:** Any Infoblox-published skills or connectors imply a higher level of support burden and lifecycle responsibility. Teams may build and share skills (even via GitHub), but Infoblox should avoid building a large official "library" of skills unless owners are clearly named. Creators must fully own code, documentation, revisions, and breakage due to upstream changes.

[Source: 2026-05-18_Jon_Himanshu_catch_up.md]

---

## Open Risks

**Technical:**

- OIDC callback routing — risk of mixed sessions and token leakage across tenants if callback URL validation is loose
- Pagination and API limits — CSP calls currently cap at ~100 items; must be fixed in API/client code, not prompt tuning
- FIPS compliance and audit logging architecture not yet specified for GovCloud/FedRAMP deployments
- Tenant-per-session implementation: tenant listing APIs and session-binding endpoints not yet designed; "role in groups" permissions feature needed before TSA/agent work can proceed

**Organizational:**

- Senior contributors (e.g., Julian) split across IQ, chart pinning, and CSP-as-ADUI explorations simultaneously — undermines focused MCP delivery
- Recurring pattern of multiple groups independently exploring AI/MCP frameworks without consolidation
- Past pattern of 75–80% completion across many initiatives; risk of MCP joining that list

**Strategic:**

- Scope of write operations not finalized — overly narrow Phase 1 risks disappointing customers; overly broad risks security incidents with autonomous agents
- Monetization model for IQ-as-MCP-tool is unresolved and should not block MVP but needs a roadmap

---

## IQL for Assets — Gateway Integration Decision (May 20, 2026)

The team decided to expose IQL (Infoblox Query Language) for the assets dataset through the customer MCP gateway rather than wiring it directly into the IQ agent. A PR from the Assets team integrating IQL directly into the agent was rejected. The correct pattern: IQL appears as a tool in the MCP gateway's tool list; the agent decides autonomously when to call it.

Rationale: blocking IQL at the gateway would require extra work; enabling it costs minimal incremental effort. If customers discover and use it, acceptable; if not, no downside.

API publication ownership: documentation and maintenance of APIs (IQL-related and SSO group mapping) rest with the owning teams (Assets, Apps), not the IQ team.

[Source: summaries/2026-05-20_mcp_server_exposing_iql_for_assets_cloud.md]

**Related:** Full IQ platform architecture now has a dedicated wiki article — see [[IQ-Architecture]] for bounded contexts, authorization model (ext_authz bridge, OPA, DESTRUCTIVE_NOOP), LiteLLM gateway, and technical design decisions.

---

## Source Files

**10 source files · Apr 2026**

`raw/2026-04-13_MCP_Server_MVP_Roadmap`, `raw/2026-04-20_IQ_MCP_syncup_with_Turner`, `raw/2026-04-24_MCP_Approval_and_Integration_Plan`, `raw/Re-_Claude_MCP_connector.eml`, `raw/Re-_Thoughts_on_MCP_-_Agent_Distribution_for_FY27_Investment.eml`, `raw/Re-_AWS_OPPORTUNITY-_Amazon_Bedrock_AgentCore.eml`, `wiki/summaries/2026-04-09_IQ_MCP_Agent_Distribution_Investment_Proposal`, `wiki/summaries/2026-04-09_MCP_Agent_Distribution_FY27`, `wiki/summaries/2026-Agent_Distribution_Platform_Proposal`, `wiki/summaries/2026-04-13_AWS_Bedrock_AgentCore`, `outputs/MCP_Server_Requirements_and_Outcomes`

**See also:** [FedRAMP-Certification-Strategy.md](FedRAMP-Certification-Strategy.md) · [IQ-Operations-Target-State-Architecture.md](summaries/2026-IQ_Operations_Target_State_Architecture.md) · [IB-IQ-Backend-Technology.md](summaries/2026-IB_IQ_Backend_Technology.md)