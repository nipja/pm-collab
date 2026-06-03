---
title: "Infoblox IQ — Platform Strategy"
created: 2025-06-01
last_updated: 2026-05-25
source_count: 198
status: reviewed
summary: "Infoblox IQ platform strategy — the AI assistant layer spanning DDI, Security, and Assets; covers vision, roadmap phases, agentic workflow design, EAP status, and GTM positioning."
type: strategy
tags: [iq, ai, platform, product]
---

# Infoblox IQ — Platform Strategy

Infoblox IQ is the company's AI assistant layer across its product portfolio. It began as a DNS/DHCP/IPAM diagnostic assistant (IQ DDI) and has expanded into IQ Security (SOC/threat detection) and IQ Assets (asset lifecycle intelligence). The strategic goal is a unified "AI force multiplier" for NetOps and SecOps teams — positioned as a platform capability, not a per-product feature.

IQ is a Vista board priority: AI-attributable ARR is a board-level reporting requirement, which drives the SOC Insights repositioning and the IQ umbrella brand.

---

## Strategic Vision

**Core premise:** AI does not replace the network admin; it multiplies their effectiveness by surfacing what to look at, explaining what it means, and — progressively — taking pre-approved actions.

**Three value pillars:**

1. **Improved UX** — eliminate context switching; surface insight at the point of work
2. **Deeper analysis** — go beyond "something is wrong" to root cause and recommended action
3. **Labor reduction** — long-term goal of closing support tickets before they're opened

**Platform-level positioning:** IQ is intentionally product-agnostic. It unifies DNS, security, and assets under a shared AI layer (the "circle with AI sprinkled" model) rather than building separate assistants per product. Shared primitives: conversation thread, Actions framework, guardrail/evaluation system, memory/context, RAG index.

**Roadmap arc (4 phases):**


| Phase   | Mode                                      | Status        |
| ------- | ----------------------------------------- | ------------- |
| Phase 1 | Read-only diagnostic assistant            | Current (EAP) |
| Phase 2 | Guarded write with human approvals        | 2026 target   |
| Phase 3 | Semi-autonomous (approved action classes) | Future        |
| Phase 4 | Closed-loop autonomous with oversight     | Vision        |


---

## IQ DDI — DNS, DHCP, and IPAM Intelligence

IQ DDI is the original and most mature IQ use case. It targets the ~60–70% of support tickets attributable to DNS, DHCP, and IPAM configuration issues — problems where the data exists in the platform but the operator lacks time or context to interpret it.

### DNS Use Cases


| Symptom                      | IQ Capability                                                                      |
| ---------------------------- | ---------------------------------------------------------------------------------- |
| NXDOMAIN spike               | Detect, explain cause buckets (typo, missing record, zone delegation), propose fix |
| SERVFAIL spike               | Identify upstream forwarder failure vs. local misconfiguration                     |
| Latency (high response time) | Root cause analysis across zone, view, RPZ interactions                            |
| Dead DNS forwarders          | Detect lame/dead forwarder chains in recursive query path                          |
| Lame delegation              | Identify NS records pointing to non-authoritative servers                          |
| DORA sequence issues         | Detect broken discover-offer-request-ack handshake chains                          |
| KIA log interpretation       | Translate raw KIA log errors into plain-language root cause                        |


**37 documented DNS support sub-scenarios** across these buckets, each with distinct data signatures and recommended remediation paths.

**DNS detection notes:**

- **NX domain noise:** dev/test traffic, known-good test traffic, and automated health checks generate expected NX patterns — suppressible per-customer threshold tuning required
- **DNS latency distinction:** DNS latency (processing time, platform performance metric) is distinct from SERVFAIL responses, which split into internally generated (NIOS failing) vs. externally received (upstream resolver failing); conflating these leads to incorrect root cause attribution
- **Serve-fail routing:** when upstream resolver fails, IQ must distinguish the failure type before recommending remediation

### DHCP Use Cases


| Symptom                          | IQ Capability                                                    |
| -------------------------------- | ---------------------------------------------------------------- |
| HA failover failure              | Auto-diagnose HA pair state, sync status, failover log sequence  |
| Handshake failures               | Identify DHCP discover/offer/request/ack breaks by scope         |
| "Everything green but no leases" | Detect exhaustion, exclusion range conflicts, MAC filtering      |
| Cache hit ratio anomaly          | Alert on sudden ratio drops (new query type, missing forwarders) |


**Three priority DHCP/IPAM use cases (April 2026, IBIQ-1180 and related):**

1. **Lease & Scope Optimization** *(MVP target — highest priority)*: Monitor discover/request volume and pool shrinkage trends per subnet/scope; project time-to-exhaustion ("90% in 3 days"); recommend shorten lease times, reclaim stale leases, surface fastest-burning scopes. Signals: discover/request counts, "no free leases" error codes, leases-per-second per subnet over rolling windows, customer-defined utilization thresholds (e.g., 80%). Start threshold-driven + simple trend projection; layer data-science forecasting (James/Vidhi) later.
2. **DHCP Config / Design Pattern Standards** *(AI-intensive)*: Infer de facto configuration templates from customer's own environment; detect deviations (different gateway placement, unexpected DNS servers, missing options, inconsistent reservation ranges); present inferred template + deviation list per scope/server.
3. **DHCP Service Resiliency / Relay & HA Validation**: Compare DHCP service logs per HA pair — if server1 sees relay messages but server2 doesn't, server2 cannot take over. Detect relays pointing at non-HA servers. Surface root-cause categories: misconfigured IP helper, DHCP snooping/spoofing, asymmetric routing. Primarily rule-based detection; LLM generates readable explanations and recommendations.

**Scope constraint:** All use cases must use existing NIOS portal metrics only — no new data collection (DHCP team overloaded). IQ Ops dashboard runs entirely on metrics data, independent of "30-day active search" token consumption.

**DNS/DHCP action backlog tickets:** IBIQ-1180 (DHCP Actions, Sprint 8, architectural review pending), IBIQ-889 (DNS Cache Hit Ratio Drop, Honeywell escalation RCA attached), IBIQ-797 (DDNS Spikes), IBIQ-796 (DNS Query ANY type). DHCP/IPAM actions take priority over DNS action types.

**Near-term DHCP/IPAM anomaly actions (EAP office hour, May 14, 2026 — 4–6 weeks):** Specific anomaly scenarios committed for the next delivery wave:
- Rapid DHCP discover floods from misbehaving devices (especially thin-client registers and POS systems stuck in PXE boot loops — devices that may send a discover every 5–10 seconds and hundreds per day)
- Repeated reservation-level boot file failures that prevent devices from completing boot
- Scope-aware behaviors interpreting anomaly severity in light of lease times and scope sizes
- Threshold-based offender lists: devices with more than 50–100 DHCP requests per day

**Near-term DNS additions (EAP office hour, May 14, 2026 — same wave):**
- DDNS spikes
- DNS cache hit-ratio anomalies
- NXDOMAIN-heavy clients (identifying misconfigured resolvers, bad suffix search behavior, or problematic `ndots` settings)

[Source: 2026-05-14_IQ_Product_Office_Hour_Whats_New_Whats_Next.md]

### Issue Tracking and Ground Truth

IQ DDI detection scenarios are driven by a top-issues tracking spreadsheet maintained by the support team:

- **DNS tab:** last updated ~August 2025 (stale); feeds Phase 1 detection scenarios and evaluation harness
- **DHCP tab:** refreshed ~March 2026 (fresher); 26 DHCP anomaly scenarios rated, 3 pending data acquisition
- **Manual workflow:** Manu (support) manually exports Salesforce UDDI case data; Ramesh, Guru, Arun, Rishi (support engineers) validate and add runbook content
- **Target automated pipeline:** Salesforce UDDI → API export → AI agent structures unstructured case text → SME validation → feeds IQ detection + evaluation scenarios
- **Access blocker:** James and Vidhi lack Salesforce case management access; IT ticket needed

### IPAM Use Cases


| Capability                    | Description                                                     |
| ----------------------------- | --------------------------------------------------------------- |
| Stale IP cleanup              | Identify IPs with no DHCP/DNS activity for configurable window  |
| Provisioning automation       | Agent-driven subnet assignment and reservation creation         |
| Natural-language IPAM queries | "Show me all /24s with >80% utilization in the 10.x space"      |
| Capacity checks               | Predict exhaustion timelines from historical utilization trends |


### Data Retention and Baselining

IQ currently operates on a **30-day rolling window of raw, unsampled metrics** streamed from supported appliances. All active investigations and anomaly detection use this 30-day dataset.

**Customer demand for longer-term aggregates (May 14, 2026 EAP office hour):** Multiple EAP customers confirmed they see up to approximately 13 months of some aggregate metrics (total query volume, NXDOMAIN counts, response codes) in core Infoblox dashboards. They want IQ to reuse those longer-term series to support:

- Year-over-year comparisons ("is this week above or below typical for this time of year?")
- Seasonal baselines for retail/event-driven environments (Black Friday, Cyber Monday, peak events)
- "Today vs. last week vs. last year" trend overlays for NXDOMAIN, QPS, serve-fail counts
- Appliance sizing and IP space growth forecasting against historical demand

**Decision (May 14, 2026):** Product team will confirm which DNS/DHCP/IPAM metrics are already retained longer in aggregate, select a curated subset (demand, NXDOMAIN, serve-fail at minimum), and expose those to IQ so it can support seasonally-aware investigations. IQ will continue to rely on 30-day raw metrics for active investigations while this longer-term layer is added.

[Source: 2026-05-14_IQ_Product_Office_Hour_Whats_New_Whats_Next.md]

### IQ Chat — DDI Assistant

The IQ chat interface is the primary UX surface for DDI queries. Key characteristics:

- **Hybrid determinism:** Natural language → IQL (Infoblox Query Language) → user reviews/edits → executes. Keeps humans in the loop for data mutations.
- **Global search first:** All chat queries invoke global search before LLM, reducing hallucination on entity lookups.
- **Guardrail coverage:** ~9 initial evaluation scenarios across three DNS buckets; ~500 validation questions needed before full production rollout (currently awaiting SME input).
- **Chat history (IBIQ-266):** Not yet live; engineering can start May 2026; basic EAP-quality version by late May is plausible; roadmap shows June as customer-availability buffer. July reserved as polish buffer. [Source: 2026-05-06_IQ_Product_Design_Launch_Priorities.md, 2026-05-06_weekly_sync_Jon_Himanshu.md]
- **Graphing in chat (IBIQ-707):** Inline visualization of query results; in roadmap.

---

## IQ Security

### Overview and Repositioning

IQ Security is the brand for Infoblox's AI-powered security investigation and response capabilities. It encompasses two components:

- **SOC Insights** (existing product, ~$2M ARR) — being repositioned under the IQ umbrella
- **Security Actions** (new) — AI-assisted triage and response workflows

The SOC Insights repositioning is explicitly driven by the Vista board requirement to report AI-attributable ARR. The newer version of SOC Insights already uses LLM and ML components, making the rebrand defensible. Older installations running legacy SOC Insights do not qualify for the AI ARR count.

### SOC Insights → IQ Security

SOC Insights provides threat-correlated DNS/DHCP telemetry for SOC teams: anomaly detection on DNS query patterns, correlation with threat intelligence feeds, and incident timelines. Under the IQ umbrella, it gains:

- Natural-language investigation entry points
- LLM-generated summaries of security events
- AI-driven triage prioritization

### Security Investigation Workflow

IQ Security structures the investigation workflow in six stages:

1. **Triage** — surface anomalies; AI scores and prioritizes
2. **Deep dive** — LLM-generated explanation of the anomaly with contributing data
3. **Enrichment** — pull threat intel, asset context, historical patterns
4. **Decision** — human reviewer confirms action class
5. **Remediation** — execute action (block, isolate, flag) with approval gate
6. **Follow-up** — verify resolution; AI confirms anomaly cleared

### Security Actions

Security Actions are the IQ Security equivalent of DDI Actions — pre-defined remediation workflows triggered by AI-detected anomalies:


| Action                | Trigger                         | Approval Required        |
| --------------------- | ------------------------------- | ------------------------ |
| Block indicator       | DNS RPZ hit, threat intel match | Needs Approval (default) |
| Isolate asset         | Anomalous outbound DNS volume   | Needs Approval (default) |
| Propose block set     | Cluster of related IOCs         | Needs Approval           |
| Propose isolation set | Asset group with shared anomaly | Needs Approval           |


All security actions include **full audit trail** and **undo flows**. Human-in-the-loop is the default; no automated execution without explicit operator approval.

**Integration targets:** Trellix (EDR), Corelight (NDR), Nessus (vulnerability data), Okta (identity context), SIEM platforms.

### Security Testing

- Informal testing by Ingmar: no security issues found
- Formal PSB (Product Security Board) testing: pending; required before GA

---

## IQ Assets

### Overview

IQ Assets integrates Infoblox's Asset Insights product with the IQ AI layer. The goal is to bring AI-driven discovery, querying, and anomaly detection to the asset lifecycle.

### Capabilities


| Capability                          | Description                                                                      |
| ----------------------------------- | -------------------------------------------------------------------------------- |
| Natural-language asset queries      | "Show me all assets that enrolled in the last 7 days with no vulnerability scan" |
| Guided discovery setup              | AI-assisted configuration of asset discovery probes and connectors               |
| Agent-driven integration connectors | Automated connector setup for Nessus, BigFix, ServiceNow, Okta                   |
| Asset count anomaly detection       | Alert when asset count changes unexpectedly (e.g., sudden drop in a subnet)      |
| MAC-centric lifecycle tracking      | Procurement → DHCP enrollment → vulnerability assessment → decommission          |


### Entity Model

Asset Insights models a rich graph beyond simple "device" records:

- **Entities:** Devices, Users, Software, Vulnerabilities, Relationships
- **Cross-domain correlation:** "Which users operate devices with critical unpatched CVEs?"
- **Per-account overrides:** ML misclassification corrections feed back into training/fine-tuning pipelines — adaptive, not static
- **Policy mapping:** customers define rules mapping asset attributes to security policies

### Integration Agent Model

Asset integrations are agent-driven, not static connectors:

- Agents handle authentication, schema mapping, and incremental syncing automatically
- New integrations added via configuration, not code
- Per-account integrations support customer-specific data sources

### Monetization

IQ Assets monetization approach is not yet finalized. Asset Insights is currently sold separately; bundling with IQ DDI/Security for ELA pricing is under discussion. The Maryland program provides a forcing function: Asset Insights must be bundled in the IC ELA because standalone air-gap overhead makes solo pricing 4–5× commercial list.

### Comply-to-Connect Integration

For the Maryland/federal use case, IQ Assets is central to the Comply-to-Connect workflow:

- DHCP enrollment triggers AI-driven asset classification
- Asset Insights tracks vulnerability scan clearance status
- IQ drives access decision (deny DNS resolution to uncleared assets)
- Classification entry criteria (UNCLASS/SECRET/TS) enforced at network admission

---

## IQ Actions Framework

Actions are IQ's mechanism for moving from read-only diagnostics to write operations. The framework uses a tri-state permission model governing what AI can do on behalf of the operator.

### Core Agentic Loop

The IQ action loop runs 8 discrete steps with a mandatory human gate at steps 5–7:

1. Anomaly detection or operator query triggers investigation
2. IQ retrieves relevant data via tool calls (CubeJS, search, logs)
3. LLM reasons over retrieved data, identifies root cause
4. IQ proposes action with explanation and confidence score
5. **Human gate — operator approves or denies**
6. Approved action executes via platform API
7. Result confirmed back to chat thread; audit log entry created
8. Follow-up check validates resolution

**Stop-execution control** halts a running sequence at any point — critical for operators who want to interrupt mid-workflow.

**NRT pipeline:** Refactored in early 2026 using Kafka; investigation latency reduced from ~15 minutes → ~1 minute.

### Two-Stage Investigation Model (May 2026)

The IQ Actions investigation pipeline is explicitly two-stage, confirmed in the May 14, 2026 EAP office hour:

1. **Anomaly agent (first-level triage):** Continuously collects a broad set of metrics beyond those that surface as visible action cards. Applies multivariate and traditional ML anomaly detection to compare metrics against environment-specific baselines. Uses contextual factors — lease times, scope sizes, baseline message rates, configuration settings — before escalating a situation to an IQ action. Goal is to surface only materially impactful situations; false positive control is treated as a core design constraint.

2. **Deep research agent (second-level analysis):** Once an action is created, performs detailed investigation by invoking tools, correlating across metrics and logs, and assembling a structured output covering likely root cause, contributing factors, and targeted remediation steps. Each action includes downloadable logs for independent forensics or handoff to external teams.

**Quality grading:** The engineering team systematically grades action outputs on a quality scale from "generic" (level 3 and below) to "surgical" — recommendations that name the specific devices, scopes, and records involved and map directly onto operator workflows. The team has been iterating toward surgical quality throughout early 2026.

[Source: 2026-05-14_IQ_Product_Office_Hour_Whats_New_Whats_Next.md]

### Tri-State Permission Model


| State              | Behavior                                                            |
| ------------------ | ------------------------------------------------------------------- |
| **Allow**          | Action executes automatically; operator notified post-hoc           |
| **Needs Approval** | Action proposed with justification; human confirms before execution |
| **Block**          | Action class not permitted; AI does not attempt                     |


Default posture for all new action types: **Needs Approval**. Operators promote to Allow or demote to Block per their risk tolerance and environment.

### Current and Planned Action Types

**DDI Actions (Phase 2 target):**

- DHCP reservation create/modify
- DNS record create/update/delete
- Subnet assignment
- RPZ rule modification

**Proposed new action types (engineering backlog):**

- DHCP HA auto-diagnosis and repair trigger
- Cache hit ratio anomaly auto-remediation
- Lame delegation validator and NS record correction
- DORA sequence analysis and scope repair
- KIA log interpretation with runbook suggestion

**Security Actions:** See IQ Security section above.

### Three-Tier Tool Approval Architecture

Beyond the per-action tri-state, IQ maintains a three-tier tool approval architecture (Tom Hayward leading broader MCP/tool-approval design):


| Tier                       | Behavior                                                               |
| -------------------------- | ---------------------------------------------------------------------- |
| Point-in-time approve/deny | No memory; operator decides each occurrence                            |
| Persistent "always allow"  | Stored in delegation lookup layer; operator promoted this action class |
| Categorical deny           | High-risk operations (e.g., delete critical DNS server) never auto-run |


A **risk matrix** maintained by the IQ team defines which operations land in which tier; the matrix must be auditable.

**Noise filtering:** Dev/test traffic excluded from anomaly detection. "Mute this action" operator control suppresses known-benign patterns. Per-customer threshold tuning balances sensitivity vs. false-alarm rate.

### Prefab — Agent-Driven UI Components

**Prefab** is an open-source component system that translates structured agent intent into inline portal widgets (forms, charts, tables) without custom UI code per action type:

- Uses ECharts for chart rendering with PDS (Infoblox design system) theming
- Julian and Tejas defining the base Infoblox theme
- Replaces dummy curl command blocks with real, clickable inline forms
- Enables rich action proposals (diff views, confirmation dialogs) within the chat thread

### Playbook Pattern

Pre-defined investigation question sequences for junior operators — makes investigation accessible to non-experts:

- Example: DNS NX domain spike → 4-step guided investigation sequence
- Significant enablement use case; IQ guides operators through structured diagnostics rather than requiring expertise to formulate questions

### MCP Integrations for Actions

External system integrations delivered via MCP server (see [MCP-Server-Strategy.md](MCP-Server-Strategy.md)):


| Integration       | Capability                                              |
| ----------------- | ------------------------------------------------------- |
| ServiceNow        | Create/update incident tickets from IQ findings         |
| Slack             | Push notifications; approval requests in-channel        |
| Splunk            | Read SIEM alerts; correlate with DNS/DHCP data          |
| NAS / Log storage | Access historical data beyond Infoblox retention window |


**Customer demand:** Self-hosted/external MCP server support to connect IQ to internal systems without routing data through Infoblox-hosted infrastructure.

### IQ Actions API Access — Published APIs and MCP Server (IBIQ-1417/1416)

As of May 11, 2026, Jon Abbe created a new Epic (IBIQ-1417) and Story (IBIQ-1416) to expose IQ Action APIs via Infoblox's published API surface and the external MCP server, gated by entitlement-based access control. This is a significant expansion: it means external agents and systems can invoke IQ Actions (not just read/query data) through the same official API and MCP channels used for UDDI and Threat Defense operations. Entitlement checking ensures only authorized customers with the appropriate IQ license can invoke action-related endpoints.

Assigned to Meenal Gupta (story). This epic is expected to land in 26.4 scope.

This initiative is closely related to [[MCP-Server-Strategy]] and bridges the gap between the external MCP server (which currently focuses on UDDI/DDI read access) and IQ's agent-driven Actions framework.

---

### Approval Workflow (IBIQ-648)

Approval workflows are a Phase 2 dependency. The UX pattern:

1. AI surfaces proposed action with explanation and confidence score
2. Operator reviews diff (what will change vs. current state)
3. One-click approve or reject; rejection prompts for feedback
4. Approved action executes; result confirmed back to chat thread
5. Full audit log entry created (who, what, when, AI confidence, human decision)

---

## IQ Chat Assistant

### Capabilities (Current / EAP)

- Natural-language DNS, DHCP, IPAM queries
- Root cause analysis for common failure patterns
- Data visualization inline (charts/tables)
- Contextual recommended next steps
- Query history within session (persistent history: IBIQ-266, July 2026)

### UX Patterns

- **FYI / FIA / FYI+A pattern:** Notifications are classified as For Your Information (diagnostic), For Immediate Action (time-sensitive), or both with recommended action attached
- **Copy table (IBIQ-685):** Requested by multiple EAP customers; allows exporting AI-generated tables to clipboard/CSV
- **Graphing in chat (IBIQ-707):** Inline charts for time-series data (DNS query rate, DHCP lease count, etc.)
- **AI-first Investigate entry point (IBIQ-1162):** Redesigning the Investigate page to lead with IQ rather than manual search

### IQ Landing Page — Design Direction (Apr 2026)

The full-screen IQ landing page is designed around entitlement-driven chips and a chat-first entry point:

- **Entitlement-driven chips:** show counts of open SOC Insights / SOC Actions / DDI Actions for entitled users; non-entitled users see a "frequent tasks" chip set based on existing configuration capabilities (e.g., add a server, add DNS service) rather than marketing tiles.
- **Competing models:** (1) embedded/chat mode — chips load SOC/DDI queues inside the chat canvas; (2) dedicated page mode — chips route to standalone queue pages. Working compromise: support both; use customer data to decide later.
- **Prompt library:** a curated set of high-value frequent-task prompts; top 3–5 visible on landing page with a "prompt library" expansion. Product must map each to an existing supported capability to avoid new backend requirements.
- **SOC/DDI detail-panel parity:** Mukesh explicitly flagged inconsistency; agreed to convert DDI's vertical-tab layout to match SOC's details-left + chat-right tabbed pattern. Phase 1: structural parity; Phase 2: incremental content.
- **Color direction:** marketing review scheduled for blue vs. green; design to provide both options. Current strong blue treatment is perceived as visually heavy.
- **"Search conversations":** in nav as a concept; requires architectural validation from engineering before committing; may be deferred if it threatens timelines.
- **Left nav simplification:** collapse SOC / DDI / operations entries into a single "IQ Actions" item; show entitlement-gated content within that page.

### IQ+ Scope and Chat Governance (April 30, 2026)

A critical alignment session clarified the scope of the paid IQ+ experience and established governance for cross-product chat patterns:

**IQ+ (paid) vs. product-embedded AI:**

- **IQ+**: Premium speedboat-owned experience with unified assistant UI across Security and DDI. Security Actions + DDI Actions (merged/collapsed) are explicitly part of the paid tier. Uses "most capable" model configuration.
- **Product-embedded AI**: Non-premium AI helpers (NLQ in query bars, AI query generators, AI-powered chart creation in Asset Insights) may be described as "AI-powered" but are not IQ-branded and do not use the full IQ assistant.
- No company-wide policy yet on where IQ assistant is allowed to appear in the portal or which capabilities are paid vs. free.

**Chat pattern governance decisions:**

- Four patterns exist: hero landing chat (full-page), fixed side-panel, draggable pop-out (Asset Insights), optional full-screen expansion. Team agreed on a **clear, limited canonical set** with documented triggers; ad-hoc variants retired.
- Downstream teams (SOC, Asset Insights) must align to speedboat patterns, not the reverse.
- Cross-product IQ chat embedding deferred until Asset Insights assumptions are validated and a paid/free boundary is formally established.
- Demo-driven deployment risk: designs reaching prod quickly for board/leadership demos create execution pressure and erode velocity. New process: PM+Design forum reviews mocks before reaching engineering.

**Speedboat accountability:** Speedboat team remains the canonical source for IQ+ UX decisions. A recurring Security+DDI PM/Design forum will review and approve new IQ-related designs before they reach engineering.

[Source: 2026-04-30_Align_on_IQ_Chat_experiences_across_various_products.md]

### Near-Term Product Priorities (April 27, 2026)

A Jon/Himanshu sync on April 27 locked the near-term IQ engagement priorities and surfaced a significant multi-model strategy discussion:

**Priority stack (in order):**

1. New IQ landing page — cleaner look, modern "start a task" affordances
2. Chat history and recent activity — users resume investigations; see what they or an agent did last
3. Context-aware login summary — "What's happening in my network today?" combining IQ Insider anomalies + core platform dashboards (DNS, DHCP/IPAM, assets)
4. Canned prompt playbooks — guided multi-step question sequences (not just empty chat box)

DHCP dashboards and other non-engagement features are explicitly de-prioritized until usage improves.

**Usage context:** Only ~30 customers enabled as of April 2026; actual question volume is very low. An open-ended chat interface alone is insufficient to drive engagement.

**Context-aware summary (replace story 171):**

- On login, IQ reads IQ Insider anomalies + core monitor pages (DNS, DHCP/IPAM, assets)
- Answers "What changed since yesterday?" and "What should I pay attention to now?"
- Suppresses areas with no meaningful change (avoids repetitive daily recaps)
- Output: 3–5 information-dense sentences with optional drill-downs
- Acceptance criteria must include edge cases: dashboards not changing day-to-day, low-signal environments, sandbox/sample data

**Prompt playbooks:** Multi-step guided sequences modeled on expert investigation patterns (e.g., "six follow-up questions to fully understand an issue"). Presented on landing page as "frequent tasks" or "investigation recipes." Not optional — they are the primary mechanism to drive users from zero to value.

[Source: 2026-04-27_Jon_Himanshu_catch_up.md]

### Multi-Model Experimentation (April 27, 2026)

The Claude+MCP prototype demonstrated dramatically richer UX than native IQ even when operating on the same underlying data — raising urgent questions about LLM selection and formatting:

**Current architecture:**

- Backend: Azure OpenAI for parts of analysis within IQ Insider
- Frontend prototype: Claude connected to MCP via cloud connector

**Proposed experiments:**

- Feature-flag model selection (Claude vs Azure OpenAI vs Gemini on GCP) behind same MCP/IQ data
- Evaluation criteria: response quality/correctness, UX formatting (pills, headings, charts), hallucination rate
- System and formatting prompt tuning to see how far OpenAI/Gemini can approach Claude-like UX

**Key design observations from Claude+MCP prototype:**

- "Pill" identifiers for hosts, subnets, and IOCs dramatically improve scannability
- Flow-based output (summary → drill-down → containment playbook) feels more coherent than fragmented portal pages
- Inline graphs (queries-by-hour, threats-by-location) eliminate the need for separate dashboard pages
- Strong agent identity branding ("Security Intelligence Agent," "Network Intelligence Agent") clarifies domain and task scope
- Current IQ prototype over-indexes on IQ Insider (NX domains, serve-fails) and underuses broader platform telemetry

**Action:** Ping Turner on model-selection feature flags; architectural work needed before feature-flag model routing is feasible (involves cloud platform choices: Azure vs GCP). Capture Claude formatting patterns — prompts, pill patterns, agent identity blocks — as explicit input to IQ UX requirements.

[Source: 2026-04-27_Jon_Himanshu_catch_up.md]

**Gemini / Google AI evaluation (May 11, 2026):**

Google is actively pitching two offerings: (1) running IQ on Gemini, and (2) hosting IQ in a new Google data center in the Kingdom of Saudi Arabia. Post-Google Next, Google announced a $750M ISV investment program; an informal ~$1M figure for Infoblox was mentioned but not confirmed by leadership.

Team's evaluation philosophy:
- Focus on architecture-level evaluation, not single-vendor commitment.
- Design for multiple LLMs (Claude, Gemini, Azure models) as a pluggable layer within a stable architecture — same approach as Glean's model-per-query selector.
- Build an internal, PM-facing model comparison surface (not customer-facing) to compare response quality, token cost, and dollar cost per model/version. Turner supports; timing TBD (July–August mentioned).
- Any commitment to Google's stack or the Saudi data center deferred until concrete financial terms, architecture details, and vendor responses are available.

Current IT stack: Microsoft 365 + Glean + Azure OpenAI. Most contracts are 1-year; renewal points will be natural checkpoints to reconsider. Historical context: Infoblox was a "Google house" until ~2 years ago when it migrated to Microsoft 365.

[Source: 2026-05-11_Jon_Himanshu_IQ_Quick_Sync.md]

### Intelligence Center

The Intelligence Center is the planned unified AI home within the Infoblox portal — a single landing page combining:

- AI-generated briefings (what happened while you were away)
- Proactive anomaly cards with recommended actions
- Chat interface with memory/context
- Cross-product visibility (DDI + Security + Assets in one view)

Replaces the current fragmented entry points (separate Investigate, SOC Insights, and Asset dashboards).

### Personas and Specialization

IQ supports multiple personas with different default question types and recommended actions:

- **NetOps / DNS Admin** — NXDOMAIN/SERVFAIL diagnostics, DHCP scope management, IPAM provisioning
- **SecOps / SOC Analyst** — threat correlation, indicator blocking, asset isolation
- **Asset Manager** — discovery coverage, lifecycle gaps, integration health
- **Executive / Manager** — summary dashboards, SLA health, team activity digest

---

## IQ Product Surfaces — Architecture Clarification (May 20, 2026)

Tom Hayward (IQ architect) provided legal/compliance with a clarification that "IQ" comprises 4 distinct surfaces with different ownership and behavior profiles:

| Surface | Description | Owner | Action model |
|---------|-------------|-------|--------------|
| **IQ Assistant (in-portal chat)** | Conversational agent calling Infoblox APIs via MCP tools | team-iq | Invokes tools; writes blocked via DESTRUCTIVE_NOOP=True today |
| **Customer-facing MCP server** | Same MCP Gateway exposed externally via portal API keys | team-iq | In production today — quiet, unadvertised, read-only |
| **IQ for DDI / NIOS (Insighter)** | Telemetry-driven RCA via Insighter Research Agent | team-iq | Recommendations only; no API tool calls |
| **IQ for Threat Defense** | Separate implementation | TD team (Shadid Chowdhury) | Recommendations + human-in-the-loop per TD team |

The "IQ does not take autonomous actions" characterization is correct for surfaces (3) and (4). For (1) and (2), the agent invokes tools, but in production today no tool call mutates customer infrastructure — DESTRUCTIVE_NOOP=True in MCP-CSP intercepts all write tool calls and returns API usage instructions instead of executing. GA target for writes: human-in-the-loop approval per IBIQ-648 (PI 26.3). See [[MCP-Server-Strategy]] for full architecture detail.

[Source: Re- Legal requirements before GA of Infoblox IQ 2.eml, 2026-05-20]

---

## IQ for TD — GA Enablement Plan (May 20, 2026)

A May 20 meeting focused on the mechanics of enabling IQ for Threat Defense for GA. The core gap: customers who already paid (SOC Insights SKU or Security Tokens) cannot see IQ or IQ-TD because IQ is still feature-flagged and entitlements are not wired to auto-enable IQ-TD.

### Key Decisions

1. **Reuse existing SKUs:** SOC Insights SKU and Security Token entitlements unlock IQ-TD. No new standalone IQ-TD SKU for either perpetual or token offerings.
2. **Auto-enable model:** IQ-TD should auto-enable when SOC Insights or Security Token entitlements are present — no IQ engineering per-tenant activation required.
3. **IQ component phasing:** IQ base menu, AI assistant, and landing page remain behind feature flags and separate monetization plan. IQ-TD ships first as the primary IQ action.
4. **Legacy SOC Insights UI retirement:** Deprecate after ~2-4 week communicated transition period. New customers onboard directly to IQ-TD; existing SOC Insights customers keep legacy UI for ~1 month.

### Rollout Details

- **GA Date:** June 2 (announcement at Cisco Live) with IQ-TD as primary GA action
- **Eligible customers:** ~100 SOC Insights customers + trial/POC customers with temporary SOC Insights-style SKUs
- **Token impact:** Enabling IQ-TD increases Threat Defense/cloud DNS token allocation by ~35% (1.35x). Over-consumption handled via standard consumption/true-up model.
- **EAP vs. GA distinction:** Largely semantic for existing customer pool; GA formalizes feature and public messaging.

### UI/Navigation Plan

- Eligible customers: IQ visible in nav with Threat Defense IQ action. DDI action and richer assistant landing page remain disabled.
- Security Token environments: always show IQ-TD entry point for discovery and opt-in
- Legacy SOC Insights: 30-day transition window; then removed via UFE nav contribution process

### Open Items (as of May 20)

- Precise token increase percentage to commit (35% is example)
- Which platform/entitlement owners implement and confirm readiness before June 2
- Exact wording for GA announcement around token overages and access to IQ assistant/landing page

[Source: summaries/2026-05-20_iq_enablement_for_iq_for_td.md]

---

## Architecture and Engineering

### Platform Architecture Model

IQ is built on a shared platform infrastructure rather than per-product AI stacks. The model separates "hard things that must be consistent" (auth, audit, compliance, RBAC) from "product differentiation" (domain-specific AI reasoning per product team).

**Shared platform services:**


| Service                   | Description                                                        | Status                |
| ------------------------- | ------------------------------------------------------------------ | --------------------- |
| Analytics instrumentation | Vendor-agnostic wrapper over Gainsight; debug overlay in dev       | In dev                |
| RBAC / permissions        | Centralized role model, OPA policy enforcement                     | Redesign in progress  |
| Notification system       | Unified alerts across products                                     | Roadmap               |
| Workflow orchestration    | Approval/deny, multi-step action pipelines                         | Foundation Sprint 5/6 |
| Audit logging             | Required for FedRAMP and enterprise compliance                     | Required for Maryland |
| Documentation ingestion   | 8 Confluence spaces + Salesforce KB + S3, daily change-based fetch | Live (Apr 2026)       |


**SDK-based contribution model:** Product teams register tools, prompts, and data sources via the shared IQ SDK. They do not implement their own auth, logging, or permission checks — the IQ runtime handles orchestration, evaluation harness, and feedback collection. Tradeoff: the platform team must maintain stable APIs and not break downstream consumers.

**Agentix:** The shared agentic runtime underlying all IQ add-ons. Provides tool registration, orchestration, approval gates, and evaluation harness integration so each product team's add-on runs on consistent, governed infrastructure rather than each building its own. Centralizing under Agentix is an open organizational decision as of April 2026 — without it, independent AI efforts (IQ, "Windu Circle," others) create inconsistent experiences and duplicated infrastructure.

**Platform resource tax model:** App teams requesting platform work "pay a tax" — either via shared developers or explicit trade-offs from their own roadmap. Platform PM/engineering capacity is insufficient for all requested work; FedRAMP, Australia realm, and geographic expansion all compete for the same scarce cloud-ops skill set. Funding alone cannot accelerate progress; hiring and ramp-up lead times are the actual constraint.

### Technology Stack


| Layer             | Technology                  | Notes                                        |
| ----------------- | --------------------------- | -------------------------------------------- |
| LLM               | Azure OpenAI GPT-5.1        | Primary; model selection via LiteLLM         |
| LLM proxy         | LiteLLM                     | Model abstraction; enables future model swap |
| Agent framework   | LangChain / LangGraph ReAct | Tool-use and multi-step reasoning            |
| Embeddings        | Ada-002 via pgvector        | Semantic search for RAG                      |
| Analytics query   | CubeJS                      | 50–100 cubes covering DNS/DHCP/IPAM metrics  |
| Analytics storage | ClickHouse                  | Time-series event data                       |
| LLM observability | MLflow / Databricks         | Prompt tracing, experiment tracking          |
| Prompt registry   | Databricks                  | Versioned prompt storage and A/B testing     |


For the unified reporting and dashboarding strategy (report builder, scheduled exports, widget library), see [[Reporting-Platform]].

### LLM Cost Management

Initial LLM cost: **~$7K/week** at baseline. Reduced **~99%** through:

- Restructuring tool calls to minimize token payload per call
- Caching frequent query patterns
- Routing low-complexity queries to smaller models via LiteLLM

**SOC Insights — April 2026 cost spike and fix:** Label generation for all submitted threat indicators was more expensive than expected during April. The SOC Insights team (Shadid Chowdhury) built an optimization: labels now only trigger when there is a traffic hit on the indicator, not on every submission. The fix was tested and deployed to production the week of May 4, 2026. Longer-term: a cost comparison between self-hosted and managed LLM models is under consideration. A GPT model performance analysis (LLM Model Test.docx) was completed to support future model selection.

[Source: wiki/summaries/fw-*litelllm_spend_summary*-_april_2026.md]

### Guardrails and Evaluation

**Evaluation framework status (April 2026):**

- 25 harness-runnable scenarios; 6 manually scored at 3/5, zero at 4–5 (target: ≥4/5 within two weeks)
- 11 MLflow traces currently tracked with LLM judge for automated scoring
- Two scoring scales: 5-point (1–5 for actions) and 4-point (1–4 for conversational answers) — 4-point scale is faster for SME sessions
- Guardrails POC showed low accuracy; ~500 SME-validated questions required before rollout

**Evaluation dimensions (five MLflow metrics):**

- Tool usage correctness
- Answer correctness (SME-validated ground truth)
- Relevancy (response addresses the actual query)
- Context precision (retrieved chunks are on-topic)
- Context recall (relevant information was retrieved)

**Category organization for ground truth dataset:**
Metrics | Non-metrics | Fact-based | Runbook/workflow | Guardrails | Multi-turn

**Customer feedback loop:**

- In-product controls designed: thumbs up/down, star ratings, "mute this action"
- **Thumbs-down → Jira automation:** customer negative feedback automatically creates an engineering ticket
- Playwright automation for dashboard interaction workflows in evaluation runs
- Support case patterns feed scenario selection; automated pipeline planned (Salesforce → AI tagging → SME validation)

**Documentation pipeline (activated April 2026):** 8 Confluence spaces + Salesforce KB + S3 IQ tech docs, ingested daily via 14 scheduled workflows (change-based fetching). Accuracy improved from ~20% → ~80% after activation. IQ now correctly identifies IQ Operations Dashboard sections and NIOS CLI commands that it previously hallucinated.

> ⚠️ **Source currency gap (confirmed Apr 2026):** docs.infoblox.com is indexed but the underlying content is out of date. EAP customer reported IQ repeatedly directing them to a CSP navigation path (`Manage → Infrastructure → Services`) that was removed years ago, causing ~1.5 hours of wasted troubleshooting. The pipeline refresh rate is not the issue — the source documentation itself lags CSP UI changes. This is a distinct problem from index staleness: refreshing the index more frequently will not fix it if the source pages are wrong. Mitigation requires either (a) docs team keeping navigation current, or (b) IQ introspecting live portal structure via MCP rather than relying on static documentation.

> ⚠️ **API object mapping and endpoint hallucination (confirmed May 2026, UDDIST-2390):** A distinct and more serious accuracy failure category. IQ Ask Anything incorrectly claimed that Access View creation is unavailable via API and must be done through the Portal UI — this is wrong. The Access View flow maps to the Compartments backend API. When subsequently asked for the Compartments API call, IQ generated the wrong endpoint (`POST /api/ddi/v1/ipam/compartment`) instead of the correct one (`POST /v2/compartments`). IQ also falsely claimed to have "verified the tenant" and confirmed no Access View API exists — a hallucinated action. This is distinct from stale docs: IQ is not citing outdated documentation; it is fabricating false API availability claims and wrong endpoints. Root cause is likely incomplete API schema coverage in the RAG corpus. [Source: UDDIST-2390, Turner Anderson, 2026-05-17]

**Blocker:** ~500 SME-validated questions not yet authored. **Bruce** (DNS/DHCP domain expert) is the sole scorer — single point of failure creating throughput constraint on the quality gate.

**Guardrail dataset review (May 11, 2026):**

Jon Abbe reviewed the Agorio guardrail dataset and submitted revised labels. Key changes:
- **Integration queries** (how IQ works with Cloudflare, Cisco, Norton, etc.): flip from block to allow — these are legitimate deployment questions from enterprise customers.
- **Competitive comparisons** ("IQ vs Zscaler"): remain blocked to avoid unsanctioned competitive claims.
- **General security posture questions**: allow and use as opportunities for IQ to demonstrate value.
- **Multi-turn drill-down**: needs richer dataset coverage; current single-turn guardrails insufficient.

Current tech stack: GPT-5.2 as base model (Azure guardrail card); evaluating GPT-5.2 mini for RLM-style checking. Self-hosting Gemma or similar is a possible future step if tighter control/latency is needed.

Sylvester to run Jon's revised dataset against the current Azure guardrail card and report gaps this week. Team committed to adding observability (log blocked queries with category/reason for periodic review queue).

**Central tradeoff:** Visible reputational risk (embarrassing screenshot) vs. invisible economic cost of over-blocking legitimate users who quietly abandon the product. IQ is B2B/enterprise-only behind authentication — reduces casual abuse risk, but internal red-teaming is aggressive.

[Source: 2026-05-11_Follow_Up_ReviewLabel_Guardrails_Evaluation_Dataset.md]

**Evaluation status (May 7, 2026 update):**

- 26 scenarios; **11 improved to score 5** (excellent); 6 at score 4+ requiring additional detail tuning (e.g., using human-readable names instead of IDs). This is a significant improvement from the April 2026 baseline of zero scenarios at 4-5.
- ~640+ evaluation samples collected across DNS, DHCP, IPAM, security, platform, and Garmin products.
- **"Guardio" guardrail dataset:** 500+ questions generated via LLM and reviewed by Jon; treated as a living dataset that grows and refines over time; may define ground-truth inputs for training and guardrail tuning.
- Auto-evaluation tied to specific endpoints; human-in-loop review required from security/SOC reviewers (Guru Raj, Bruce, Dimitri, Yugandhar) before improvements are declared done.
- Future: dedicated RCA evaluation category for IQ Actions and IQ Assistant root-cause queries.
- Requested: P1/P2/P3 tagging of evaluation issues; screenshots of improvements for security reviewer validation.

[Source: 2026-05-07_Weekly_IQ_Platform_Sync_up.md]

**Evaluation status (May 14, 2026 update):**

- **14 scenarios at rating 5**; 6 at rating 4+ still being improved by Bruce, Ravneet, and the markets team
- **600+ evaluation questions** collected; automated daily eval pipeline under development — will publish results to a dashboard accessible to engineering, PM, and leadership
- Eval improvements and ratings are based on simulation/evaluation data, not production customers; code for the 14 improved scenarios is in production but ongoing 4+→5 work is still in progress
- Event streams now include DHCP, cache hit ratio, DDNS, and DNS ANY query type; registration controller being simplified to lower onboarding barrier for other teams
- **182 cube descriptions** reviewed and deployed to US Dev 2; production deployment expected within a week

[Source: 2026-05-14_IQ_Weekly_IQ_Platform_Sync_up.md]

**Known open risks:**

- Bruce bottleneck: single SME; score throughput cannot scale without additional validators
- US stage environment: query logs not yet enabled (blocks MLflow evaluation runs)
- Salesforce access gap: James and Vidhi lack case management access (blocks automated pipeline build)
- Stale DNS scenario set: based on August 2025 data; DHCP tab fresher (~March 2026)
- False positive rate for anomaly detection not yet benchmarked at scale
- ML-based anomaly detection not yet integrated (rule-based only currently)
- Feedback storage mechanism for "thumbs up/down" not finalized
- Shared service load (QGIS/Promcula) competing with IQ inference requests
- Unresolved: whether to use prod-like staging or production sandbox for evaluation (impacts score-to-production translation)

### Infrastructure and Observability

**OAuth (Prasad):** Required for Cisco product launch end-May 2026; on track. Critical path dependency for approval/confirmation workflows and MCP server hardening.

**EventGen Dashboard:** Operational in Grafana with separate SRE and customer/support persona views. Kafka lag alerting configured for on-call integration. Replaces ad-hoc Prometheus/Kafka/Databricks queries.

**NRT Pipeline:** Refactored using Kafka, reducing investigation latency from ~15 minutes → ~1 minute. Staged rollout: US Dev2 → Stage1 → US Dom1. Data freshness validation in progress; upstream lateness issues being addressed at source rather than by relaxing freshness guarantees.

**A2UI Framework:** AI-assisted dashboard composition layer for flexible, customer-customizable dashboards. Ownership unclear — platform, app teams, and UX all touching it without clear PM leadership. Jon and Alex driving framework standardization; responsibilities to be handed to app PMs. Strict rule: no external customer data exposure without vetting.

**Analytics (Gainsight):** Vendor-agnostic wrapper deployed in dev with debug overlay. Intent: platform-level analytics service so individual product pages don't instrument separately. Abstracts over Gainsight to allow future vendor changes without re-instrumentation.

**Gainsight PX component-level analytics (May 14, 2026):** Julian Hernandez wired custom events to specific UI components; Tejasvi Shiv configured three Gainsight PX dashboards tracking component-level usage (button clicks, widget usage, section engagement). This is the first time component-granularity analytics are available for IQ. Three dashboards: (1) IQ adoption and user metrics (active users, time in zones), (2) AI assistant analytics (questions asked, feedback, suggestion chip clicks), (3) action analytics (action page engagement by section). Gap: API-user data not yet included — large blind spot for SOC/enterprise customers who don't use CSP UI directly. Access via SNOW ticket + manual add by Tejasvi; no shareable link. [Source: 2026-05-14_Weekly_IQ_Demo_Presentations.md]

**Slash-command prompts (May 14, 2026):** Julian Hernandez implemented a VS Code / Claude-style slash-command interface in IQ Chat. Typing `/` queries the MCP gateway and returns available prompts dynamically. Initial prompt: `onboarding` — runs a pre-flight configuration readiness check. Backend by Ken Turner; users can click to invoke or type partial name + Tab to autocomplete. Designed to reduce first-time user friction and surface high-value capabilities without browsing. [Source: 2026-05-14_Weekly_IQ_Demo_Presentations.md]

### Sprint Execution

- 2-week sprint cadence; weekly demos
- Sprint 4: 153 committed vs. 112 completed — systemic overcommitment identified
- TPM team taking over sprint running to enforce capacity discipline
- Key process change: stories must be estimated and sized before sprint commit
- TargetProcess deprecated; full migration to Jira
- "Speedboat team" for IQ UDDI: 3 DNS + 3 DHCP engineers identified

### Sprint and Release Planning Improvements (May 18, 2026)

The May 18 IQ Roadmap Workshop surfaced persistent planning discipline issues and agreed on concrete improvements.

**Current problems:**
- Stories enter sprints without estimates; teams discuss requirements and assign points days after the sprint starts
- Cross-team scope creep: new stories from TD, Asset Insights, NIOS arrive mid-sprint without trade-offs
- No GA release tag in Jira: no single field marks which stories are P0 for the first IQ GA
- Cross-team dependencies not consistently surfaced on the platform Jira board

**Agreed improvements:**
- Enforce a consistent pre-sprint cadence: (1) product alignment meeting to choose sprint candidates from M1/M2/M3; (2) engineering planning to handle spikes, dependencies, and estimates before the sprint starts so stories are ready on day one
- Introduce a GA release tag or label in Jira once leadership names a concrete GA release; tag only stories that truly must ship for that release
- Implement either: hard Jira workflow rule blocking "In Progress" without story points, or a soft dashboard/report that highlights such stories for same-day intervention by James or other managers
- Mid-sprint additions from other squads are reprioritizations, not free extras — something must come out when something goes in; this decision should be visible and acknowledged
- Leads expected to actively resist mid-sprint scope additions without explicit trade-offs; where external leaders override this, platform leads should escalate

**M1/M2/M3 interpretation:** Treat these roadmap buckets as prioritized sequencing, not as hard delivery commitments. Some epics span multiple months; capacity-aware planning will cut or slide lower-priority items when estimates show everything won't fit.

[Source: 2026-05-18_IQ_Roadmap_formerly_Process_Workshop.md]

**Sprint 5/6 key epics:**

- Stop-execution experience (IBIQ-852)
- Approval/deny foundation (IBIQ-648) — platform-level primitive shared across all IQ add-ons
- Deep linking from actions table (IBIQ-607)
- Dynamic charting (IBIQ-707)
- Chat history epic punted from Sprint 6 — AI/extended IQ vision not settled, UX design incomplete

### Sprint 8 Engineering Stories (April 30, 2026)

New stories active in Sprint 8 per the IQ Update April 30 deck:


| Story                                                   | Description                                                                                                                                                  | Owner     |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------- |
| Config Readiness Check (IBIQ-834)                       | MCP prompt for config readiness check; checks 30d active search; extensible for more checks; frontend integration in Sprint 8                                | Abhishek  |
| Conversation Service API Redesign (IBIQ-937)            | Redesigns conversation service to fix P1 bug (timeouts dropping connection); also allows users to navigate away from chat without stopping response creation | Abhishek  |
| Cross Product Anomaly Intelligence Expansion (IBIQ-800) | Config-based pipeline for anomaly modeling with notebooks; new approach produces fewer anomalies with better explanations; testing against known outages     | Vidhi     |
| Out of Scope Guardrails (IBIQ-161)                      | Testing various guardrail approaches; curating eval dataset for proper tuning                                                                                | Sylvester |


[Source: 2026-04-30_iq_update_april_30.md]

### Quality Status (Sprint 8, April 30, 2026)


| Area         | Status     | Detail                                                                                                                                                               |
| ------------ | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dashboard    | Green      | —                                                                                                                                                                    |
| Actions      | Green      | 26 DNS anomaly scenarios rated ≥4/5 (26 rating=4, 6 rating=5); 4 scenarios improved from rating 4→5 post human review; DHCP anomaly data collection kick-off in prep |
| IQ Assistant | **Yellow** | Alignment with stakeholders complete; eval data collection and auto-generation kicking off                                                                           |
| Defects      | Green      | No significant defects                                                                                                                                               |


**Anomaly scenario testing (NXDOMAIN/LATENCY/SERVFAIL):** 37 total scenarios identified; 30 simulated; 27 re-ran via test harness; 26 rated ≥4, 6 rated 5. 3 open defects (2 LATENCY, 1 NXDOMAIN).

[Source: 2026-04-30_iq_update_april_30.md]

### Official Launch Positioning (May 2026)

The IQ Launch Positioning Brief (Mehul Patel, May 2026) serves as the source document for PR, the product webpage, and Mukesh's CPO blog at Cisco Live 2026. Confirmed scope for Phase 1 (June 2 announcement, EAP — not GA):

**Three pillars:**

| Pillar | Description |
|--------|-------------|
| **IQ Actions** | Product-tied actions: IQ for Threat Defense (threats, triage, remediation), IQ for DDI (config insights, approved changes), IQ for Asset Insights (coming later) |
| **Agentic AI Assistant** | Conversational layer across the full portfolio — natural language queries, config translation, surface right data at the right moment |
| **Infoblox MCP Server** | Standards-based MCP; third-party AI tools and agents connect to Infoblox data; agent-to-agent workflows |

**Tagline:** "The agentic operations layer for network and security teams."

**Key customer outcomes:**
- Security: 504,000 events → 24 actionable insights in one deployment
- Network: 45–90 min investigations now surface with context to act
- Assets: "Stale CMDB means wrong decisions at machine speed" — IQ for Asset Insights addresses the foundation

**June 2 = announcement, not GA.** Confirmed on May 12 (Himanshu → Olga Phillips): the June 2 Cisco Live date is a public announcement via PR + new product page + Mukesh blog. The product remains in EAP. CTA: book a meeting. Himanshu/Jon/Mehul lead inbound conversations.

**Revised GA timeline (confirmed May 12):**
- IQ for Threat Defense: GA target end of June / early July
- IQ for DDI: Limited Availability for "few more months" after June 2
- Catalyst for acceleration: BlueCat announcement the prior week

**Legal note:** Adding Azure OpenAI to existing SKUs as a default-on feature triggers a 30-day advance customer notice obligation. Olga Phillips (Legal/Privacy) flagged this May 12 — requires resolution before GA.

> ⚠️ CONTRADICTION: The May 12 guidance says "June 2 = announcement, not GA" (IQ for TD GA target: end of June / early July). The May 20 GA Enablement Plan section below (line ~561) says "GA Date: June 2." The BlueCat Field Response v3a (May 20, Paul Nguyen) also states "IQ for Threat Defense is GA now" in anticipation of June 2. Resolution: June 2 appears to have been solidified as the IQ for TD GA date by May 20. The May 12 "announcement only" position was likely overtaken by subsequent decisions. User should verify which date is authoritative.

**SRS 2026 target for DDI and Asset Insights (per field response, May 20):** The BlueCat competitive field response v3a officially states IQ for DDI and IQ for Asset Insights will reach GA at SRS 2026 (Summer Revenue Summit). Universal token pricing for these add-ons will be finalized at SRS 2026 as well.

[Source: iq_launch_positioning_brief_may_2026.md, re- reconnect on legal T&Cs for Infoblox IQ.md, Glean: BlueCat Agentic AI Field Response v3a 2026-05-20]

---

### Launch Readiness and Quality (May 8, 2026)

**June 2 Cisco Live launch confirmed.** The May 8 Platform Team meeting formally confirmed the June 2 Cisco Live public launch date for IQ, aligned with the Cisco AI Canvas ecosystem story and a joint press release with BlueCat. Himanshu heavily involved in ensuring the narrative reflects product intent. MCP server collaboration with Cisco: Infoblox provided MCP server technology; Cisco is building a demo showing AI Canvas working with third-party capabilities including IQ.

**IQ broadly available — principle decision.** Mukesh and Himanshu aligned that IQ can appear across the portal including "free" asset areas used by PLG users; access is not limited to paying customers by default. Practical limits (usage caps, advanced feature tiers, realm-level restrictions) may be added later.

**SOC Insights / IQ entitlement:** Existing SOC Insights customers (~$2M ARR) will be grandfathered. Provisioning details (who flips the switch, how entitlements are encoded, how to differentiate legacy SOC Insights buyers from new TD Advanced customers) to be clarified with Anant and platform by end of June.

**Two assistant flavors converging:** Action-specific chat and generic IQ assistant converging into a unified model with: a dedicated IQ landing page, persistent chat history, page-aware assistance (later May/June), and global context for cross-product queries.

**EU realm:** Phase 1 (May) — MCP server API-based auth for Cisco Live scenarios and basic EU-realm access. Phase 2 (June/July) — full OAuth with user credentials and approval workflows, bringing IQ deployments back into the normal platform deployment pipeline.

**M1 sprint lock (May 8 bi-weekly planning).** Six core M1 epics confirmed for May delivery:

| Epic | Phase / Notes |
|------|--------------|
| Chat error / response timeout | Flexible; may slide to M2 if near-complete |
| Graphing template results (Turner) | Appears done or near-done; may drop from active M1 list |
| PDR Action Detail Panel (Epic 516) | P0 Sprint 9; UX re-layout only (no new backend); Rockwell/Julian to confirm M1 |
| Full-screen IQ Chat Landing Page | Engineering confirmation needed; Tejasvi Figma designs ready |
| Customer-facing MCP Server | Phase 1 (May): API key read-only. Phase 2 (June): AAUTH + hardening |
| EU Realm | Small config change; "M" realm addition; non-negotiable May |

Non-negotiable for May: MCP server Phase 1, full-screen landing page, Epic 516 UX alignment, EU realm. Chat timeout and graphing are flexible M1-or-M2. DNS anomaly epics 796/797/889 moved to M2/M3. DHCP onboarding epics 1180/1191/1192 not M1 platform scope.

**A2A vs MCP integration architecture (May 8 Turner/Himanshu/Jon sync).** Two distinct integration patterns clarified for partners and ecosystem positioning:

- **MCP**: External vendors or customers build their own agents that call into IQ; external party owns reliability and behavior. "You build on us."
- **A2A (Agent-to-Agent)**: Infoblox-owned agents communicate with external agents as peers; Infoblox owns reliability, IAM, observability, and audit trails. "Our agent works with your agent."

The **Interaction Agent** is IQ's de facto orchestration agent — handles tool calls, retries, and is the intended coordination point for research flows, actions, sub-agents, and future event-driven workflows. The **conversation service** is the primary conduit for external agent-initiated events (e.g., ThousandEyes latency alerts triggering IQ investigations). A2A is viewed as directionally where the industry is heading and largely within reach on the current stack with modest conversation service extensions.

Sub-agent policy: require concrete eval failures before adding sub-agents; avoid unnecessary architectural complexity. Market analogies: CrowdStrike/Elastic (agent frameworks plus monetized marketplaces), Google (company-built agents plus framework with certificate-based identity and IAM).

[Source: 2026-05-08_Platform_team_meeting.md, 2026-05-08_IQ_Bi_weekly_Product_prioritization_for_next_Sprint.md, 2026-05-08_Turner_Himanshu_Jon_weekly_IQ_Sync.md]

---

### Launch Readiness and Quality (May 7, 2026)

**Accelerated public announcement timeline:** The May 7 IQ Platform sync confirmed an internal decision to accelerate the Infoblox IQ public announcement to end-May / early June, coordinated with Cisco Live, in direct response to competitor (Bluecoat) announcing an AI offering with an MCP server and limited assistant. The underlying DDI experiences remain EAP — the announcement is a marketing-facing event.

**Three major May 7 decisions:**

1. Publicly announce IQ around end-May/Cisco Live while keeping DDI in EAP.
2. Focus MCP server on SaaS for launch; on-prem MCP for IOS customers deferred to ~Q1 FY27.
3. IQ for Threat Defense is the first IQ component expected to reach GA (June/July timeframe); design and onboarding experience for existing Threat Defense customers must be solved soonest.

**Updated launch milestones (May 7):**

| Area | Planned Timing |
|------|---------------|
| IQ Assistant (DDI + limited security ~28 SOC questions) | Mid-May EAP |
| IQ Actions — DNS / Threat Defense | Active May EAP; TD actions likely GA June/July |
| IQ Actions — DHCP/IPAM | June, possibly into July |
| Chat History | Work starts May; delivery ~June |
| MCP Server (SaaS, API key read-only) | Non-monetized; part of end-May IQ launch |
| Landing Page with Playbooks | May; guides EAP customers with sample questions |

**Arch/PM Sync (May 7) — roadmap triage:**

- M1 (May): Full-screen landing page (UX reformat only; no new APIs); simple redirects; MCP Cisco Live read-only slice; EU realm support reclassified as M1.
- M2 (June): Chat assist (architectural components); copy table / copy code blocks; API auth accuracy; API documentation cleanup; chat refresh standardization.
- M3 (July+): Cross-portal page awareness; operations/workspace page summarization; visual agent responses (embedded HTML/CSS, charts); advanced chat assist flows.
- Super prompts, entire-minutes flows, full chat assist, and generic cross-portal awareness explicitly deferred from May.

[Source: 2026-05-07_Weekly_IQ_Platform_Sync_up.md, 2026-05-07_IB_IQ_ArchPM_sync_up.md]

---

### Launch Readiness and Quality (May 1, 2026 — Updated)

**Phased launch plan (finalized May 1 across three PM-lead sessions):**


| Milestone                               | Target                 | Description                                                                                                                        |
| --------------------------------------- | ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Press release + "Cisco-ready MCP"       | End of May 2026        | IQ announcement tied to Cisco Live; limited, read-only, API-key MCP for Cisco only — not IQ GA or broad customer enablement        |
| Broader IQ availability                 | June–July 2026         | Controlled pilots expand; July preferred for UX cleanup, performance, and corpus; IQ disabled by default for general customer base |
| Public purchasing/ordering (IQ Actions) | ~July 2026             | Token allocation for IQ Actions through standard commercial channels                                                               |
| IQ for Security GA potential            | ~August 2026           | Sales readiness conference; Security squad may reach GA readiness sooner than DDI                                                  |
| DDI-focused IQ GA                       | September–October 2026 | Contingent on systemic performance work, OAuth enterprise MCP, UX polish, PSEC signoff                                             |


**MCP launch split (May 1 decision):**

- "Cisco-ready MCP": API-key auth, read-only, limited tools, for Cisco Live partner demo. Deliberately below Infoblox's long-term enterprise security bar; labeled as special-case partner configuration.
- "OAuth-ready enterprise MCP": OAuth, client registration, per-client tool visibility via RBAC, server-side workflow approvals — the real enterprise-grade offering, planned after Cisco launch.

**Performance — systemic approach (May 1):**
Root cause of IQ perceived slowness varies by query/cube/data path, creating "whack-a-mole" pattern. PM leads agreed to flip the approach: treat **all queryable data** as needing a defined latency bar; create systemic performance epics with owned accountability rather than per-scenario fixes. Concrete issues: frontend timeouts triggering after backend work is done; cached retries appear instant; UDDI→ClickHouse migration not yet complete. Ghanana hire and eval framework to provide broad latency coverage and measurable before/after tracking.

**Agent branding cleanup (May 1):**

- Legacy labels like "Blocks 1 Cloud" and "Blocks 1 Threat Defense" leak from data sources into LLM context and user-visible responses.
- Docs/platform cleanup stories needed to normalize to "Universal DDI" and current product names.
- System prompt find-and-replace mappings being added as short-term fix while source data is cleaned.
- Concern about uncontrolled "agent" proliferation, especially in Security where many "agents" are static playbooks. Decision: avoid over-branding "agents" at launch; keep visible agent surface small, coherent, and well-defined.

**Guardrail workflow (May 1):**
AI-generated guardrail sets have misclassified real product questions as off-topic (e.g., "Can you use Infoblox B1 DDI with Azure?" placed on block list). PMs must own in-scope vs. blocked definitions. New process: AI proposes candidate guardrail items → PMs curate via explicit white/blacklists with rationales → batched review, not dump-all-items-on-one-PM.

**Internal model selector (May 1 decision):**
Feature-flagged model selector dropdown for PMs/engineers to compare LLM providers on real scenarios; per-user, hidden from customers. Backend already supports per-request model choice. Covers OpenAI variants initially; Claude and GCP models when AWS/GCP approvals and platform work are done.

**Q1 integrations (May 1 roadmap):**
ServiceNow, Slack, Teams, and CSP notifications confirmed as Q1 priorities so IQ artifacts/insights/actions can surface in existing workflows. Configuration intelligence (IQ actions become more configuration-aware, reducing manual setup) also Q1 priority.

[Source: 2026-05-01_Reconnect_on_legal_TCs_for_Infoblox_IQ.md, 2026-05-01_Turner_Himanshu_Jon_weekly_IQ_Sync.md, 2026-05-01_Weekly_IQ_Product_Engineering_Sync_up.md]

### Launch Readiness and Quality (Apr 2026)

Leadership sync Apr 29 surfaced that product quality, speed, and reliability remain below GA bar. Key issues:

- **Mukesh's board demo** exposed normalized bugs (chat history gaps, broken flows, reliability failures) that daily users had stopped noticing.
- **DDI Actions:** explicitly not GA-ready; will remain in preview/limited availability/EAP. Multiple leaders stated they would refuse to call DDI actions GA until quality and completeness are acceptable.
- **Launch timing:** leadership consensus now leans toward **September/October 2026** rather than July/August, despite go-to-market pressure from Anant/VPs. Preference is to ship a strong, reliable product rather than rush a marketing-heavy early launch.
- **UDDI → ClickHouse migration not yet complete:** UDDI responses are slow because the migration hasn't happened, directly degrading IQ speed and demo quality. This dependency is not visible in current roadmaps/Jira.
- **Cross-product divergence:** SOC, DDI, and Asset Insights are making parallel decisions on agent settings, naming, and assistant usage — creating inconsistent workflows. A cross-product PM+design alignment session has been proposed.
- **Product–market fit not established:** insufficient real customer usage data; unclear whether customers will pay for IQ at current quality level, especially for DDI actions.

**Sprint 7 and July 2026 milestone blockers:**

- Chat history (design now finalized; IBIQ-266)
- MCP server consolidation (IBIQ-1200) — code review and security plan required
- Guardrails/out-of-scope detection (IBIQ-1201) — **launch-critical**; ~500 SME questions still needed
- Approval confirmation (IBIQ-648) — depends on **platform OAuth** (Prasad; required by end-May for Cisco launch)
- Visual context (in-page interpretation; IBIQ-171) — visible-only scope accepted

**October 2026 milestone features:**

- Broader context/memory across pages
- Advanced guardrails (automated evaluation pipeline)
- Additional form types

**Cross-board fragmentation:** IBIQ (IQ features) and IQP/IB Platform board overlap causes confusion; IQP board is largely uncurated and PM capacity focuses on IBIQ. IQ assistant has one primary implementer — single-point-of-failure risk.

**Team updates (April 2026):** Jyoti (AI test engineer) onboarded; new PM starting early May; Paul Lawrence running 3–4 customer demos/week; Bhaskar assigned as GM contact for UDDI tracking coordination.

---

## UX and Design

### Design Principles

- AI surfaces insight; human makes the call (especially for mutations)
- Minimal mode switching — AI lives within the existing workflow pages, not a separate tool
- Progressive disclosure — simple answer first, drill down available on demand
- Explainability — every AI finding includes "why" (data sources, confidence, reasoning trace)

### Core UX Shift: Dashboard-First → Assistant-First

The fundamental product philosophy change (validated February 2026 design review): the portal entry point is now chat, not a dashboard. KPI cards appear as **action triggers** that pre-seed the chat context, rather than as the primary information surface.

- Old model: dashboard as entry point → operator navigates to IQ
- New model: chat-first landing page → contextual KPI "diamonds" surface anomalies proactively

### Chat Layout — Three-Column Design

Selected over slide-out overlay to give chat visual weight as the primary surface:


| Column | Content                                                       |
| ------ | ------------------------------------------------------------- |
| Left   | Chat history panel — unified list across all chat types       |
| Center | Active chat thread                                            |
| Right  | Context panel — insights, recommended actions, entity details |


### Floating Action Button

Two modes serving different operator intents:

- **Global (persistent FAB):** open-ended questions from anywhere in the portal
- **Contextual (pre-seeded):** opens with detail panel context already loaded — avoids re-stating what the operator is already looking at

### Split-Pane Pattern

Standardized across all IQ surfaces (SOC Actions, IQ Actions, CTEM, DDI):

- Left panel: context/detail view
- Right panel: IQ chat
- Replaces previous "details above, chat below" layout

### Actions UX Details

- Explicit approve/deny controls inline in chat thread
- Stop-execution capability at any point in a running workflow
- Deep linking from actions table back to originating IQ conversation
- "Mute this action" control for operators who want to suppress a known-benign pattern

### Prototyping Process — Alignment Fixes (March 2026)

Root causes of prior engineering/design misalignment identified and resolved:

- Engineering was unaware of the GitHub prototype repo
- PMs excluded from IQ UI/IQ UX Slack channels
- Epics were too broad for clear delivery scope

**Agreed approach:**

- Each story references a specific GitHub prototype file, flow, and version tag
- PM sign-off required before development begins
- PMs added to IQ UI/IQ UX channels
- Short cross-functional alignment meetings for blockers
- UI automation in progress: Figma → PDS component generation → GitHub PR

### Active Design Work

- **AI-first Investigate page** (IBIQ-1162) — entry point redesign to lead with IQ chat
- **Approval UX for Actions** (IBIQ-648) — diff view, one-click approve/reject, audit log
- **Intelligence Center** — unified home (see IQ Chat section)
- **Chaitrali Dhole** leads UX design for IQ; **Tejasvi** handles broader platform UX

For the canonical details panel pattern (horizontal tabs + chat-right), SOC/DDI convergence plan, severity sort decisions, and chat behavior rules, see [[UX-Design-Direction]].

### IQ Chat UX Consolidation — Two-Pattern Model (May 18, 2026)

The May 18 IQ Roadmap Workshop produced alignment on consolidating the current proliferation of IQ Chat experiences.

**Current state (problematic):** Three visually different chat patterns in play:
1. Landing page assistant — full-screen experience for IQ Insights
2. Record-tied right sidebar chat panel on detail pages (e.g., specific actions)
3. A chat experience tied to the actions list or other pages that behaves differently again

**Target model:** Two core patterns only:
- Full-screen/landing page view
- Single reusable collapsible side panel (page-aware, record-aware, or list-aware) — consistent behavior regardless of which page it appears on

**VS Code Copilot model (agreed mental model):** IQ Chat should maintain one persistent conversation thread across navigation, while the current page or record is supplied as implicit context as the user moves. The user does not need to restate what they're looking at; context shifts are signaled by visual markers (dashed line, timestamp divider labeled "Context updated to current page") inside the transcript.

**Key behavioral requirements:**
- When chat is visibly attached to a record detail view, cue clearly that the record's data is primary context — but the user can still ask general questions beyond that record
- Global chat history lists conversations by IQ ID (IQ-123 style) and allows re-opening past conversations from any view
- Suggested prompts near a record must be scoped to that record; avoid prompts that imply global behavior while the UI signals record-specific context
- Navigation from landing page to action detail pages must preserve the ongoing conversation thread — do not start a new chat on navigation

**Actions required:** Fix prototype so landing page → TD actions opens with chat panel visible and existing conversation loaded; share all Figma chat patterns to the IQ chat group as a single canonical reference; align UX stories for June (M2) chat flows before pointing.

[Source: 2026-05-18_IQ_Roadmap_formerly_Process_Workshop.md]

For the IQ chat visual richness architecture (component registry, widget service, A2UI, agent tooling), see [[IQ-Chat-Rendering]].

---

## Roadmap and Milestones

### IQ PLG / Activation and Nav Workflow (May 11, 2026)

**Navigation change (by July 2026):** SOC Insights and SOC Actions will be removed from the Security nav and relocated under a new IQ parent nav entry. IQ becomes the left-hand parent; sub-entries include AI Assistant and AI Actions (with Threat Defense and DDI as children). No duplication; existing Security nav entry deprecated.

- Existing SOC Insights customers: grandfathered into the new IQ-based experience via entitlement check on legacy SKU.
- TD customers without SOC Insights: discover IQ via the new IQ section or store.
- Badging framework (EAP/beta/GA): ready by end of June so non-GA subfeatures (Assistant, DDI) can display appropriate labels when IQ for TD goes GA in July.

**Self-activation / store concept:** A unified in-product catalog combining Labs-style trials and GA feature activation:
- Reuse/extend existing Labs area; no new left-hand nav item.
- Two activation models: turnkey (fixed token allocation on enable, ~35% of tokens for TD+Cloud DNS) vs. usage-based (tokens deducted on actual usage, e.g., UDDI, Asset Insights).
- Goal: 30-day trials from store before committing tokens. First version may simply activate for known buyers.
- Backend enablement (sales/SE-driven) as interim path until store ships.
- Token type (security vs. universal) treated as interchangeable currency in the UX.

**Open ownership question:** Who finalizes user stories for nav change and store flows — Platform PM, product PM, or shared? Explicit epic/user story required to prevent miscommunication between Security and Platform teams.

[Source: 2026-05-11_IQ_PLG_Enablement_Workflow.md]

### Product Nomenclature (Confirmed May 12, 2026)

**Finalized naming (per Himanshu, Jon, and Janet Noe — Sr. Director, Brand), confirmed in 15-minute alignment meeting May 12:**

| Surface | Label |
|---------|-------|
| Umbrella product | **Infoblox IQ** (shortened to "IQ" only in space-constrained nav) |
| Sub-offerings | **Infoblox IQ for Threat Defense**, **Infoblox IQ for DDI**, **Infoblox IQ for Asset Insights** |
| MCP server | Infoblox MCP Server (no "IQ" appended) |
| In-nav assistant | **AI assistant** |
| In-nav actions hub | **AI actions** |
| Actions children | **Threat Defense**, **DDI** (not "Threat Defense actions," not "DDI actions") |
| Page headers | "Infoblox IQ for Threat Defense" / "Infoblox IQ for DDI" |

**Labels to avoid:** "IQ Assistant," "IQ Actions," "Threat Defense IQ Actions," "DDI IQ Actions" — these risk sounding like separately purchasable sub-products. Threat Defense should be spelled out; do not invent a new abbreviation (risk: BloxOne naming déjà vu). DDI is an accepted, well-understood abbreviation.

**Working compromise:** PMs and designers may use "IQ Assistant" / "IQ Actions" informally in Jira stories and internal documentation. Marketing keeps "Infoblox IQ" as the external umbrella brand with generic descriptive sublabels.

[Source: 2026-05-11_Mind_Mastery_Day_and_IQ_with_Himanshu.md, 2026-05-12_Finalize_Infoblox_IQ_DDI_TD_Nomencalture.md, Re- Finalize Infoblox IQ DDI & TD Nomencalture 2.md]

---

### Sprint 9 / Sprint 10 Scope (May 12, 2026)

The May 12 bi-weekly sprint grooming session finalized M1 production priorities for Sprint 9 and 10.

**Deferred to Sprint 10+:**
- **IQ Chat landing page** — UX complete (Tejas/Himanshu approved), but SOC/DDI backend and frontend normalization must stabilize first; IQ Chat variations (modes per product) need to be extracted before landing page inline experiences are safe. Engineering/architecture review (Tom) required before any sprint commitment.

**Active in Sprint 9:**
- **SOC/DDI actions normalization** — In Progress: Mark (backend payload normalization, dynamic templates), Rockwell (frontend component convergence). Prerequisite for landing page.
- **MCP / Cisco Live** — Focused story created for API key authentication + documentation; full RBAC/enablement UX deferred to post-Cisco Live. Cisco Live runs May 31–June 4.
- **Guardrails** — Stand-alone runtime service (no LangChain/LlamaIndex integration in Sprint 9). Sylvester owns POC → formal diagrams and story breakdown.
- **EU realm** — Sprint 9 = prep work; Sprint 10 = deploy-to-prod CMRs; EU deployment used to minimize per-realm overrides for Australia (October).
- **DHCP metrics (V1)** — Scope narrowed to 5 specific DHCP metrics for EAP. Tom creates a dedicated IQ Platform onboarding enabler epic (EventGen validation, pipeline checks) separate from Andrew Nell's feature epic. Vidhi Vazirani owns validation/sanity checks, not full model development. The broader DHCP IQ Operations Dashboard metrics epic (IQP-145) was explicitly deprioritized — confirmed May 12 by Himanshu: this epic was never prioritized for DHCP given early negative EAP feedback on the DNS dashboard; the IQ Platform team made this call.

**DHCP Anomaly Detection Engineering Status (May 12, 2026):**
- All NiosX EAP metrics have been pushed to IQP (UDDI Speed board) — Owner: UDDI Speed board. [Source: Re- Question on IQ Dashboard Metrics.md]
- EventGen building a univariate model for anomaly detection — Owner: James Zhu
- IQ Insighter working with UDDI team to provide runbacks for DHCP anomaly detection — Owner: Mark Phillips
- QA validation starting after prioritized list shared — Owner: Irfan

**Evaluation pipeline:**
- Epic 942 = primary eval runner service (first consumer: IQ Assistant; designed for org-wide reuse); in Reviewing under Tom/Turner
- Epic 1241 = data collection and contribution workflows only; active automation stories migrate from 1241 → 942

[Source: 2026-05-12_IQ_Bi_Weekly_Sprint_Grooming.md]

---

### IQ Monetization Strategy — Data Moat and Operational Workflows (May 18, 2026)

Jon and Himanshu aligned on a clear strategic focus for where IQ investment should go:

**Primary investment:** Premium data sets — the "data moat." Proprietary DDI telemetry and enriched network intelligence that cannot be replicated by an external agent + generic MCP server. This is the primary monetization and differentiation vector.

**Secondary investment:** Operational workflows. Scheduling reports, pinning items, Slack/email integration, recurring task automation — capabilities that turn IQ into an operational assistant running in the background, not just an on-demand Q&A tool.

**MCP's role:** MCP should expose IQ's free and paid capabilities rather than becoming a competing feature layer. Customers who pay for IQ see those premium capabilities via MCP; the MCP server is a conduit, not an alternative.

**What to avoid:**
- Expanding custom dashboards or MCP feature "knobs" — these are platform-level commodity work, not IQ's differentiated story
- Absorbing every customer request from office hours into the IQ backlog — tag feedback as "IQ work" vs. "core product work" vs. "platform capability"
- FOMO-driven feature requests (e.g., Asset Insights wanting IQ "mainly because others have it") before core IQ experience is mature
- Shipping capabilities without PM sign-off and clear quality bars to get early feedback — this risks customer trust more than it generates useful signal

**IQ's "third pillar" trajectory:** Over time, IQ should mature beyond chat assistant and projects/sharing into operational workflow pillars — customer-support agents, automated network hygiene tasks, and recurring insights delivered to the channels where operators already work.

[Source: 2026-05-18_Jon_Himanshu_catch_up.md]

### IQ Pricing and Packaging — Early Exploration (May 13, 2026)

Jon Abbe is developing hypotheses for free vs. paid IQ boundaries:

| Tier | Capabilities |
|------|-------------|
| Free | Read-only insights, basic guidance, "answer" experiences |
| Paid | Orchestration, configuration changes, approval workflows |

**Architectural implication:** System must detect entitlement and route accordingly. Read-only queries available to all users; state-changing operations gated behind paid entitlement, potentially requiring an approval workflow step.

**Constraints:** Current token-based pricing model limits clean tiered packaging; makes it harder to align clearly differentiated assistant capabilities with price points.

**Industry analog being researched:** Cisco good-better-best tier model; IQ-like AI value bundled into product tiers.

[Source: 2026-05-13_Jon_Himanshu_catch_up.md]

### IQ Assistant Differentiation Debate — Paid vs. Free Tier (May 15, 2026)

An internal debate surfaced on whether the AI Assistant itself should carry a paid premium or serve as a free adoption driver.

**Himanshu's proposal (May 15):** Differentiate paid vs. free IQ Assistant by **capability and experience quality**, not by usage quantity (number of questions). Six potential dimensions analyzed (deck: `infoblox_iq_assistant_differentiation_v3.pptx`). Key differentiated capabilities proposed for paid tier:
- IQL (Infoblox Query Language) query generation alongside answers (especially for Asset Insights/UAI)
- Deep links to portal pages embedded in responses
- Platform skills: custom dashboard creation, configuring cloud discovery jobs
- Page context awareness (today IQ doesn't know what page you're on; roadmap item)
- Cross-domain answering as stronger capability in paid tier
- In-product enforcement requires platform architecture investment (not needed for customer comms immediately)

**Mehul Patel's counter-argument (May 15):** This breaks the existing "IQ for X = Assistant + Actions" messaging already aligned with marketing. Charging a premium for the assistant is increasingly hard to justify as AI assistants trend toward table stakes in SaaS (e.g., Salesforce moving to headless/assistantless UI model). Preferred alternative: use the Assistant as a **free adoption driver** — surface Action summaries proactively, demonstrate operational value, then drive PLG to paid Actions adoption.

**Tim Bardzil's input:** Assistant should be accessible from anywhere in the portal (not just IQ pages); IQL query generation is especially valuable (query can be saved, shared, embedded in dashboards).

**Status:** Unresolved as of May 15; team alignment needed before any engineering investment in in-product enforcement.

[Source: raw/Re- Differentiating between the AI Assistant capabilities & experience in the paid add-on vs. outside of It.md]

### IQ Landing Page — Phased Delivery (V1/V2/V3) (May 14, 2026)

The May 14 landing page alignment session established a phased delivery model replacing a single "boil the ocean" launch:

| Phase | Scope | Status |
|---|---|---|
| V1 / MVP | Current side-panel chat re-platformed onto IQ Common library; chat history improvements possible; no new landing | P0 blocker: Deepak's IQ Chat commonization |
| V2 | New landing page with full-screen chat; updated action navigation + deep links with query parameters | P0 blocker: Mark's normalized action model + REST APIs |
| V3 | Floating/minimized assistant; polished variant-switching UX | Deferred |

**P0 prerequisites:** (1) Deepak extracts IQ Chat into versioned IQ Common library with documented contracts. (2) Mark normalizes action data model (DNS/DHCP/asset/SOC = unified "action" type) and replaces cube queries with REST insider APIs. (3) Rockwell/Anuj normalize DDI/SOC action page skeleton (KPI widgets, filter bar, charts, table, slide-out).

**Risk — security team silo:** Threat Defense team building interactive demo features ("mark all risky," undo flows, activity logs) using undocumented/ad-hoc backend APIs, diverging from the normalized action design and creating unrealistic customer expectations. Pattern of using P1 defects to push feature requests without grooming.

**Full scope** (landing + all chat variants + normalized actions + deep links) not achievable by end of May 2026; may extend beyond June once story sizing completes.

[Source: 2026-05-14_Landing_Page_IQ_Chat_Alignment.md]

---

### New Hire — Anomaly Detection / RAG / Data (Starting May 26, 2026)

A new PM-adjacent hire joins May 26 to focus on anomaly detection, ML insights, data pipelines, and RAG-ready data work for the IQ assistant and actions.

**Responsibilities:**
- Validate completeness of IQ data landscape for RAG use cases
- Identify missing support and customer data sources
- Review IQ actions for clarity from a new-user perspective
- Explore support-oriented IQ use cases; convert to user stories

**Coordination:** Jon's notifications work (defining triggers, audiences, channels) feeds into new hire's anomaly detection scope — anomaly events are a primary notification trigger. Turner's team has owned data work to date; new hire will validate and extend.

[Source: 2026-05-11_Mind_Mastery_Day_and_IQ_with_Himanshu.md]

---

### Phased Launch Plan (Authoritative as of May 1, 2026)

| Phase | Target | Description |
|-------|--------|-------------|
| Public announcement (PR + product page + blog) | **June 2, 2026 (Cisco Live)** | EAP status only; CTA = book a meeting. Himanshu/Jon/Mehul lead inbound. |
| IQ for Threat Defense GA | End of June / early July 2026 | Confirmed May 12 (Himanshu), reconfirmed May 18 (Olga). Six legal gating items identified May 18 (see Legal Gating section below). Legal documentation work must occur prior to GA — not EAP. [Source: 05-15-26 Product Weekly Status, Glean; 2026-05-18_legal_requirements_before_ga_infoblox_iq] |
| IQ for DDI | Limited Availability for "few more months" after June 2 | Per Himanshu, May 12 |
| IQ DDI GA | September–October 2026 | Contingent on performance, OAuth enterprise MCP, UX polish, PSEC signoff |
| IQ for NIOS (on-prem) GA | September–October 2026 | Jon's estimate (May 13); requires NIOS T&C addendum due to PII telemetry |
| NIOS MCP Server GA | October 2026 | Confirmed by Olga, May 18; on-prem, no IQ cloud telemetry required |

> Note: The prior ⚠️ CONTRADICTION between "July" and "August" for IQ Security GA is now resolved. May 12 confirmation: end of June / early July GA target for IQ for Threat Defense. August was an internal Sales Readiness Summit milestone, not the GA date.


### Legal Gating — IQ for Threat Defense GA (May 18, 2026)

From Himanshu Raval's May 18 follow-up after a legal alignment call with Olga Phillips:

**Explicitly gating IQ for TD GA:**
1. **Sub-processor confirmation** — Confirm Azure OpenAI appears in the sub-processor list for both Platform (James Zhu) and Threat Defense (Naveen Singh). The 30-day advance customer notification requirement triggered by adding a new LLM to existing SKUs cannot start until the list is finalized.
2. **SOC Insights customer review** — Anant Vadlamani to determine whether any existing SOC Insights customers have special contractual considerations before IQ is turned on for all of them.

**Conditionally gating (depends on findings):**
3. **IQ supplemental T&C addendum** — Olga Phillips and James Zhu drafting IQ-specific supplemental T&C. If this is required for IQ for TD, it becomes a GA gate.
4. **Product security sign-off** — Anant/Karthik and TD engineering must confirm James Hobbs (PSec) has tested IQ for TD and given green light.

**Required before launch (not gating but mandatory):**
5. **In-product AI disclaimer** — All IQ for TD screens must show: "Infoblox IQ uses AI. Please review responses before taking action."
6. **LLM data usage documentation** — Engineering to document exactly how IQ for TD uses GenAI and what customer data is collected for Actions/Insights.

**Additional open items (Olga, May 18):**
- Deep dive needed on MCP server architecture and security (pre-GA, not EAP).
- Deep dive needed on IQ architecture, existing actions, guardrails, and customer controls.
- T&C coverage for MCP server users who do not accept IQ-specific terms (Jon's position: MCP should follow API T&C model, not IQ-specific terms).
- T&C coverage for IQ Assistant embedded in core products like Asset Insights.

**NIOS T&C path (Jon Abbe, May 13):**
- SaaS IQ: No T&C changes required. IQ makes existing customer data easier to access without changing what is collected.
- NIOS IQ: Addendum required. The existing Prometheus pipeline collects only aggregate/anonymous metrics (not PII). IQ telemetry adds full DNS query/response logs and DHCP logs — these contain client IP addresses and hostnames (PII). This is a material change outside existing NIOS T&Cs.
- Two separate NIOS paths: (1) IQ for NIOS — cloud telemetry, new addendum required, GA Sept–Oct; (2) NIOS MCP Server — on-prem, no IQ cloud telemetry, no new IQ T&C required, GA October.

[Source: 2026-05-18_legal_requirements_before_ga_infoblox_iq.md, summaries/2026-05-01_reconnect_on_legal_tcs_for_infoblox_iq.md]

---

### Near-Term (4–6 weeks, ~June 2026)

From the May 14, 2026 EAP office hour — committed delivery wave:

- DHCP/IPAM anomaly actions: rapid discover floods, PXE boot loops, reservation-level boot file failures, scope-aware behaviors (see DHCP Use Cases section)
- DNS: DDNS spikes, cache hit-ratio anomalies, NXDOMAIN-heavy clients
- Per-user chat history (operators can return to past investigations, prompts, and visualizations)
- NIOS 9.0 hotfixes enabling IQ metrics forwarding from on-prem grids (~mid-June, lab/early-adopter)
- MCP server EAP with OAuth connectivity and server-side approval enforcement (~4 weeks from May 14)

### July 2026 Targets

- Chat history (persistent across sessions — IBIQ-266)
- DHCP Actions (HA auto-diagnosis, scope repair)
- IPAM Actions (subnet assignment, reservation create)
- ServiceNow, Slack, Teams, CSP notifications integrations (Q1 priority)
- Configuration-intelligent IQ actions (Q1 priority)
- MCP server for external integrations

### Post-July 2026 (Q1/Q2 FY27)

Collaboration and sharing features, confirmed at the May 14, 2026 EAP office hour as post-July roadmap:

- Assistant-created dashboards pinnable as account-wide "useful reports" accessible to all team members
- Shareable conversations and outputs
- Named and organized report collections
- Scheduled recurring reports — any ad-hoc query+visualization can become a recurring report delivered via email or integrated channels (Slack, Teams, ServiceNow, BMC Remedy Helix)
- Threshold-based scheduled reports (e.g., daily list of devices with >50 DHCP requests/day or above NXDOMAIN thresholds) for NOC and field services teams
- Mechanism for customers to submit community-sourced prompts and playbooks to Infoblox for possible productization
- Controlled write operations (blocking domains, creating IP space) via MCP behind RBAC, human approvals, and auditable change tracking

### October 2026 Targets

- Advanced guardrails (automated evaluation pipeline, SME-validated ground truth)
- Cross-page memory (IQ retains context from prior sessions and user corrections)
- Security Actions GA (block indicator, isolate asset)
- IQ Assets natural-language queries GA

### Dependencies / Critical Path


| Dependency                             | Impact                          | Status            |
| -------------------------------------- | ------------------------------- | ----------------- |
| ~500 SME-validated guardrail Q&A pairs | Production readiness gate       | Not started       |
| Approval workflow UX (IBIQ-648)        | Required for any Phase 2 action | In design         |
| MCP server GA                          | External integration use cases  | In parallel track |
| FedRAMP High (Asset Insights SaaS)     | Maryland IQ Assets deployment   | Critical path     |


---

## EAP Program and Customer Feedback

### EAP Structure

- **Pipeline:** 79 customers expressed interest → 10–12 qualified (UDDI or SaaS DDI required)
- **Two cohorts:**
  - UDDI-enabled customers: full feature access including on-premises data
  - Sandbox-only customers: portal/SaaS DDI data only

### Legal and Compliance Readiness (May 1, 2026)

A cross-functional legal/privacy/security/commercial session on May 1 identified critical pre-GA requirements:

- **Cross-product IQ supplemental addendum** (ASAP — Jon/Chris/Olga): Covers telemetry authorization from NIOS/NIOS X/Threat Defense/Asset Insights, PII handling (IP addresses, client IDs, hostnames, DNS query logs), LLM usage rules (Azure OpenAI; no customer data used for model training), and cross-cloud handling.
- **Subprocessor update** (30 days): Azure OpenAI must be explicitly listed in an AI/LLM-specific section of the subprocessor table; 30-day customer eject right must be honored when adding new LLM providers. **Confirmed blocker (May 20, 2026):** Tom Hayward verified that Azure OpenAI, Microsoft Azure, and Databricks are NOT on the current public subprocessor list (last updated 12/2025). AWS and Okta are present. All three must be added before GA; the 30-day clock for IQ for TD does not start until posting.
- **EAP enrollment formalization**: Current informal "here's a key" practice is risky; customers must formally register and accept EAP terms before access. CAB registration alone may not be sufficient for IQ/Labs features.
- **Infoblox Labs disclaimers**: Features activated via Labs must show prominent UI labels: experimental, may never GA, may be removed, not covered by standard SLAs.
- **Security posture**: Early internal testing exposed serious AI system vulnerabilities; product security and InfoSec must provide explicit GA approval; trust center to include red-team/pen-test summaries. IQ cannot be declared GA — especially for security-adjacent features — until these are resolved.
- **SOC Insights → IQ migration**: ~~$2M revenue (~~35% price uplift); will be transitioned to IQ-based experience via SKU/documentation updates without renegotiating master terms.
- **EAP sign-up form** (confirmed by Kaushik Jandhyala): Mandatory agreement form includes NDA. Customers signing up via Infoblox Labs path do NOT need to complete the form. Customer registration URL captured in EAP Weekly Status Report summary.

[Source: 2026-05-01_Reconnect_on_legal_TCs_for_Infoblox_IQ.md, 2026-05-01_turner_himanshu_jon_weekly_iq_sync.md]

### EAP Status (May 1, 2026)


| Metric                          | Value                                                                                                                                         |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Actively testing IQ (confirmed) | 7: Infoblox IT, Schaeffler AG, Target, ExxonMobil, Nationwide, American Airlines, Fresh Electric Company                                      |
| Provisioned to test             | 9: Williams Sonoma, Boortmalt/NTT, Cognizant, Nordstrom, Here Withholding Corp, NCR Voyix, eFellows Ltd, Carnival Cruise Line, Liberty Mutual |
| New customers enabled (recent)  | RSA, General Mills, Nordstrom, Viasat                                                                                                         |
| Experimenting in some form      | 30+                                                                                                                                           |
| Bi-weekly office hours          | Starting May 14                                                                                                                               |


[Source: 2026-05-01_Weekly_IQ_Product_Engineering_Sync_up.md, re-*eap_weekly_status_report*-_week_of_april_20th_2026.md]

### EAP Status (May 14, 2026 — First Office Hour)

The first bi-weekly EAP office hour session was held May 14, 2026. Multiple customers participated; no individual names recorded. Key signals from the session:

- Customers are actively using IQ against real DNS/DHCP/IPAM issues and engaging on the roadmap
- Top requests: DHCP anomaly actions, long-term baselining, configuration audit, scheduled reports, MCP EAP access
- False positive control was called out explicitly as the primary trust-building factor for IQ Actions
- Several customers want to contribute prompts/playbooks for possible productization
- Retail-oriented customers (implied by seasonality discussion) want year-over-year baselines before peak periods
- Early-stage UDDI deployments are underrepresented in current IQ capabilities; explicit user stories committed

[Source: 2026-05-14_IQ_Product_Office_Hour_Whats_New_Whats_Next.md]

### EAP Status (May 7, 2026)

| Metric | Value |
|--------|-------|
| Customers enabled (direct or IQ sandbox) | 25+ |
| New customer (week of May 7) | 1 — Catalyst Brands (JC Penney); Naga primary contact |
| Enterprise prospects onboarding | 9+ |
| Total customers engaged (UDDI pipeline) | 40+ |
| Enabled directly in account | 4 — Infoblox IT, Boortmalt, Cognizant, Nordstrom |
| Ready to enable | 8 — Here Holding, Exxon, Williams Sonoma, Nationwide, Carnival, Target, Liberty Mutual, American Airlines |

- Usage still below target; bi-weekly office hours starting May 14 to drive activity and feedback.
- Capitalist Brands (JC Penney) is the most engaged EAP customer as of May 7; only one sustained tester per logs.
- Upcoming multi-customer "office hour" to bring several customers together for live scenario testing.
- Strategic planning workshop planned (May/June) with 4-5 accounts holding known AI budget for IQ showcase and launch quotes targeting August SRS.
- Next up: Salesforce, S&P Global, Sidarion, new EAP leads from Tech Summit and CAB.

[Source: 2026-05-07_Weekly_IQ_Platform_Sync_up.md, iq-update-may-7-2026.md]

### EAP Status (April 29, 2026)


| Metric                       | Value                                           |
| ---------------------------- | ----------------------------------------------- |
| Customers enabled            | 20+                                             |
| New customers (as of Apr 29) | 5 — Viasat, RSA, Avid, Nordstrom, General Mills |
| Enterprise prospects         | 9+                                              |


- **Usage:** Below target — team pushing to drive up activity and feedback
- **Bi-weekly office hours:** Starting May 14
- **Strategic Planning Workshop:** May/June — 4–5 accounts with AI budget; IQ showcase + launch quotes targeted for SRS (August)
- **Growing AI interest:** Cleveland Clinic, Nordstrom, Fairview Health, Schneider Electric, T-Mobile, Verizon, Citizens Bank, Caterpillar, Lowe's, Wipro
- **Engaged accounts in motion:** American Airlines (workshop scheduled), Salesforce (InfoSec/Privacy FAQ pending), Nationwide & ExxonMobil (follow-ups scheduled)

[Source: 2026-04-30_iq_update_april_30.md]

### Observability Gap — EAP Usage Stats (April 30, 2026)

Himanshu Raval flagged that Brent @ Viasat does not appear in EAP Roundup usage statistics (2026-04-24), even though Viasat is an active EAP customer providing feedback. Full conversation history is stored in MLFlow but not captured in the EAP usage roundup reports.

**Implication:** EAP usage stats likely undercount actual customer activity. Himanshu planned to review Brent's session history in MLFlow (week of May 4).

[Source: 2026-04-30_fw_iq_feedback_brent_viasat.md]

### Customer Feedback (Named)


| Customer                 | Key Feedback / Use Case                                                                                                                                                                                                                                                                                                                      |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Salesforce               | Slack-first interaction model; MCP server integration priority                                                                                                                                                                                                                                                                               |
| WilmerHale               | Guided runbooks for recurring DNS incidents                                                                                                                                                                                                                                                                                                  |
| Western Bank Alliance    | IPAM capacity checks; ServiceNow/Teams/Slack connector requests                                                                                                                                                                                                                                                                              |
| Liberty Mutual           | Proactive anomaly detection (don't wait for tickets)                                                                                                                                                                                                                                                                                         |
| Nordstrom                | $100M AI investment; self-service portals; wants IQ-driven automation                                                                                                                                                                                                                                                                        |
| ServiceNow               | 95% automation target; Infoblox as CMDB quality data source                                                                                                                                                                                                                                                                                  |
| Einstein Medical School  | Zero DNS admins goal; IQ must handle L1/L2 independently                                                                                                                                                                                                                                                                                     |
| Exxon                    | IP scavenging automation; "pull out the garbage" (unused DNS zones / records)                                                                                                                                                                                                                                                                |
| Cleveland Clinic         | EBC scheduled May 2026                                                                                                                                                                                                                                                                                                                       |
| Viasat (Brent)           | Stale navigation guidance caused 1.5h+ troubleshooting; session not reflected in EAP usage stats                                                                                                                                                                                                                                             |
| Here Holding Corp (SE)   | IQ answered top queried domains query that UDDI reporting could not; led directly to a sale                                                                                                                                                                                                                                                  |
| Caterpillar (CAB Apr 30) | Long-term NIOS on-prem customer; top use cases: log RCA (Zgrep replacement), DHCP utilization alerts, forecasting, ticket enrichment, operator enablement; sandbox access granted post-CAB                                                                                                                                                   |
| NYC DOE (prospect)       | Largest SLED org in US (~1,900 schools, 1M Chromebooks, 3M devices); Microsoft DDI with cascading outages, no RCA; top asks: misbehaving client detection, per-school baselines, capacity planning for Chromebook rollout. May 7: IQ enablement approved for their UDDI POC tenant; Mehul recommends sandbox access as primary demo vehicle. **May 21: demo with DOE infrastructure team confirmed** (Thursday 11am EST recurring meeting). Andrew Bertuglia (AM) coordinating; Mehul Patel, Himanshu, and Jon Abbe attending. |
| Nationwide Mutual        | Active EAP; workshop (AI day) planned June 10, 2026 (11am–1pm lunch-and-learn). Mehul Patel drafting 3 use cases (network, security, visibility) for George Huang review by ~May 11. Deal in Negotiate stage: $238K, close March 2027.                                                                                                       |
| Abbe National Bank (ANB) | 3 months in EAP; Sarah Cho (VP NetEng) and Marcus Webb (Sr. NetOps) actively using IQ. Core pain: IQ value is trapped in individual sessions — no sharing, no export, no way to show teammates or non-IQ stakeholders what was discovered. Three distinct needs: (1) shareable conversation links (read-only or re-runnable); (2) annotated example queries for onboarding new users; (3) scheduled reports for manager stakeholders outside IQ. RBAC-aware sharing is a hard requirement: Marcus flagged a past incident where a junior analyst had excessive IPAM access. [Source: meeting-notes-abbe-national-bank.md] |


### Customer Voice (Field Quotes)

> **Einstein Medical School:** "We want to make most IT workflows as autonomous as possible, even for DNS so that he can have ZERO people managing DNS. Not even one or two. Zero."

> **Exxon Mobil:** "I would love to see IQ help me pull out the garbage, we never have time for that — like an AI scavenging assistant: here are your top 10 unused DNS zones, or your top 100 unused records or subnets."

> **SE for Here Holding Corp:** "I asked IQ for top queried domains and it came back with the answer that UDDI reporting wasn't able to provide. I said sold and asked the customer for the PO."

> **Stephen Holland (Field SE):** "What IQ changes is that it doesn't just surface the metrics — it understands the context behind why I'm asking. That's the difference between a reporting tool and something that actually encodes how Infoblox SEs think."

> **Sarah Cho, VP Network Engineering, Abbe National Bank (May 2026):** "I'll figure something out in IQ — like I asked it to show me all the subnets expiring in the next 30 days and it just did it — and then I want to share that with Derek or my manager but I can't. I don't need some big complicated sharing system. I just want to be able to say here, look at this."

[Source: 2026-04-30_iq_update_april_30.md, meeting-notes-abbe-national-bank.md]

### IQ Assistant Platform-Wide Expansion (May 7, 2026)

A May 7 meeting to align on IQ Assistant placement across the full CSP platform identified a core UX tension: should a floating assistant button (bottom-right FAB) be an omnipresent platform pattern, or remain limited to IQ-specific areas?

**Current state:**
- IQ has a full-screen landing page with AI assistant and actions.
- DDI and SOC use right-side assistant/chat panels on their existing dashboards.
- The Assets team has mockups using a floating FAB on asset pages.

**Agreed direction (May 7):**
- All product pages (DDI, SOC, threat defense, assets) should share the same basic structure: KPIs at top, records/content in primary view, IQ chat panel on the right. Product-specific styling only, not different behavior patterns.
- Backend scope is confirmed global: the assistant can answer about assets, DDI, SOC, DNS, configurations from any invocation point. Customers cannot be constrained to "local-only" questions.
- Assets team can adopt the floating FAB as the initial implementation of the broader platform vision; once validated, other squads adopt the same pattern.

**Unresolved:**
- Whether the FAB is platform-standard (always present, like the Infoblox logo) or squad opt-in.
- Deep linking vs. local rendering: when a user asks "show asset" or "visualize," render within IQ or navigate to dedicated product pages?
- Ownership of the core platform-wide assistant component (IQ platform team vs. individual squads).
- Earlier platform-wide FAB prototypes were shown but not formally accepted; needs re-approval.

**Next step:** Jon to sync with Himanshu; then cross-squad PM alignment session before broader messaging or engineering commitment.

[Source: 2026-05-07_IQ_Assistant_behavior_across_CSP.md]

### Account Feature-Enablement Tool

An internal tool is in development (as of May 7, 2026) to allow PMs to enable IQ and platform features per-account without Jira tickets or engineering-mediated deployment config edits.

- Target users: product managers; intended to replace the Jira-based "please enable IQ for this account" workflow.
- Core workflow: enter account name/storage ID → view available features and current state → enable/disable with confirmation → audit log records who enabled what and when.
- External-override tag when deployment config has been changed outside the tool; feature-flag priority field arbitrates conflicts.
- Not yet in production; strong interest in making it available for lab accounts and EAP customers once hardened.
- Engineering retains ownership of feature-flag definitions, priorities, and technical guardrails; PMs manage when and for whom features are turned on.

[Source: 2026-05-07_Weekly_IQ_Demo_Presentations.md]

### SAAT Collaboration Opportunity (Apr–May 2026)

SAAT (Support Analysis Automation Tool) is an internal support tool built over ~5 years by Jelle's team that automatically processes NIOS support bundles and surfaces structured alerts, graphs, and categorized findings. It has ~350 discrete analyses, processes ~2,600–3,000 bundles/month, and saves ~50 min of engineer time per bundle.

**What SAAT does:** Analyzes NIOS support bundles, packet captures, DB backups, and endpoint bundles. Turns raw files into graphs, tables, alerts, and reports via a regex engine and backend scripts. Support-owned and operated; provides a single-UI dashboard per file.

**IQ productization opportunities:**

- Turn "black-and-white" SAAT findings (RPZ oversubscription, IP space exhaustion, VM undersizing, known bug signatures) into customer-visible IQ Actions and playbooks.
- Use SAAT logic as a backend engine feeding IQ — avoid rebuilding proven analyses from scratch.
- SAAT-based checks could power proactive case deflection: alert on looming capacity or configuration problems before customers open tickets.
- SAAT signals (DB sentinel violations, DNS anomalies) fed into IQ as an operational view.
- Guided workflows coaching support engineers through recurring investigation patterns.

**How SAAT could help IQ:**

- Rich labeled dataset of real incidents, anomalies, and bug patterns — high-value for IQ training/evaluation.
- Forensic context: drill into historical support bundles and prior analyses.
- Feedback loop: IQ outcomes and customer feedback could prioritize new SAAT analyses.

**Entitlement/commercial questions:** Which SAAT-derived IQ features are a paid support benefit (premium vs. elite maintenance tier) vs. core product entitlement?

**May 2026 escalation:** Himanshu Raval characterized SAAT as having "a lot of goodness in what it can already do in generating IQ Actions for issues that IQ isn't covering yet, especially for NIOS." He formally forwarded to Ramesh Damodaran (NIOS engineering), Manash Kirtania (NIOS product), and Tom Hayward (NIOS engineering lead) to evaluate and brainstorm. Integration will require **NIOS-specific changes** to invoke support bundles within the IQ product experience. Jelle Nabuurs requesting IQ access; follow-up meeting planned early May.

**May 4, 2026 — Ramesh Damodaran confirmed:** "SAAT analyzes logs, resource usage, and a broad set of known issues that engineering has identified and instrumented in NIOS. It can complement IQ as an additional tool." Ramesh requested access to the SAAT x IQ PowerPoint. This is the first explicit confirmation from NIOS engineering that the collaboration is viable. Next step: Tom Hayward to collaborate with Jelle Nabuurs team on a brainstorm and study.

**Key contacts:** Jelle Nabuurs (Manager Enterprise Support — SAAT lead), Theo Fietelaars (support leadership), Ramesh Damodaran (NIOS engineering), Tom Hayward (NIOS engineering lead).

[Source: wiki/summaries/2026-04-29_infoblox_iq_saat.md, wiki/summaries/saat-x-infoblox-iq-exploring-ai-driven-support-operations.md, wiki/summaries/fw-*infoblox_iq*-_saat.md, wiki/summaries/re-*infoblox_iq*-_saat.md]

### NIOS IQ Integration: Hotfix & Data Pipeline (May 2026)

A May 6, 2026 cross-team sync (Jon, Ramesh, Dave, Adam, Tom — NIOS engineering leads) aligned on the technical approach for bringing NIOS appliances into the IQ data pipeline.

**Hotfix Plan:**

- **Target releases:** NIOS 906, 907, 908 LTS (largest installed base) as first priority; 9.1.0 hotfix to follow after LTS validation.
- **Purpose:** Streams DNS query/response logs (DNSTAP), DNS metrics (RNDC/bind stats — same model as UDDI), DHCP metrics (counters only), DNS/DHCP configuration files, and audit/syslog data from NIOS appliances to the IQ cloud backend.
- **Installation:** Script-based patch from support site — not a full NIOS upgrade. Per appliance (not cluster-wide). Requires nameD and DHCPD service restarts; no full appliance reboot. Fully reversible via a standard uninstall script.
- **Behavior:** IQ data collection on by default after hotfix installation and service restarts (valid because only EAP customers will actively apply the patch). Per-appliance NIOS CLI toggle allows administrators to disable or re-enable IQ export — all-or-nothing (no partial field selection).
- **Join token required:** Each appliance must possess a valid join token to connect to IQ cloud; only internet-accessible appliances need to be enrolled.

**Data pipeline validation status (May 6):**


| Dataset                           | Status                                        |
| --------------------------------- | --------------------------------------------- |
| NIOS syslog and audit logs        | Working — visible in Grafana                  |
| NIOS DNS metrics                  | Working — stats visible in Grafana            |
| NIOS config files (DNS/DHCP conf) | Bug NIOSRFE-8345 — resolved May 6             |
| NIOS DHCP metrics                 | NIOS side done; send side not yet implemented |


**Product design questions raised (Himanshu):** When NIOS-sourced IQ Actions appear in the queue, do they need additional columns (Grid ID + server ID)? Will global search + API hyperlinking (IBIQ-607) work for NIOS servers the same way it does for NIOS X? Customer signal from Earl (American Airlines): should the IQ Actions queue be separated or filterable by product type when customer has both UDDI and NIOS.

**Privacy & Legal requirements:** IQ collects PII-containing data (IP addresses, audit logs, syslogs) — a material change from existing BlocksConnect/call-home (which was designed to avoid PII). IQ-specific T&Cs or an addendum required before broad NIOS GA. Consent capture options under evaluation: cloud portal splash page, token purchase flow, NIOS UI/CLI toggles. For 9.2+, explicit opt-in path required before any NIOS appliance streams data to IQ cloud. Key legal partner: Olga (AI legal background from Salesforce).

**Timing:**

- **Mid-June 2026 (public):** NIOS 9.0 hotfixes enabling DNS/DHCP metrics forwarding confirmed as publicly available target at the May 14, 2026 EAP office hour. Currently in lab testing; expected to reach lab and early-adopter environments around mid-June. This brings NIOS customers to parity with UDDI-based IQ experiences for IQ Actions and assistant investigations.
- 9.1.0 hotfix: after LTS validation.
- Legal work (T&C updates): aligned with NIOS EAP rather than forced into May milestones; estimated 4-6 weeks.

[Source: 2026-05-06_NIOS_Sync_up_on_IQ.md, Re-*Data_Pipeline*-_IQ_on_NIOS.md]

### EAP Documentation Readiness (May 2026)

Alex Mandel (Information Developer) completed two EAP-facing documents intended for publication on docs.infoblox.com:

1. **IQ_EAP_CustomerOverview_v1.4.docx** — high-level overview for mixed audience (technical admins + security/compliance reviewers). Section 5 (Data Handling & Security) requires PM + Legal sign-off before external publication.
2. **IQ_EAP_TechnicalDeploymentGuide_v1.4.docx** — task-oriented guide for technical admins and IT ops.

**Action items (Himanshu, May 2026):**

- Security/privacy FAQ to be separated into a standalone doc and shared with Salesforce ASAP (Anant and Karthik to update with IQ for Security information)
- IQ Sandbox environment doc to be shared with Earl @ American Airlines
- IBIQ-1292 (BloxOne legacy terminology cleanup) — Jon moved from "To Do" to "Reviewing"; Himanshu confirmed the stale "BloxOne" naming is actively degrading IQ response quality (RAG pipeline draws from public API docs); Paul Lawrence and Glenn Sullivan asked for additional cleanup guidance
- Anant + Karthik to begin IQ for Security GA documentation planning

[Source: wiki/summaries/re-_iq_eap_documentation-_review_and_next_steps.md, wiki/summaries/re-_jira_ibiq-1292_remove_legacy_bloxone_terminology.md]

### Known Geo-Availability Gap (May 2026)

**IQ is not available on the EU realm.** Confirmed by Geriet Wendler (Solutions Architect Manager, CEUR South, Infoblox Germany) in response to Himanshu's SE outreach on May 4, 2026. European customers and field teams cannot access Infoblox IQ through the EU-hosted portal. No resolution timeline documented.

[Source: wiki/summaries/aw-_need_your_feedback_and_ddi,_uai_and_td_questions_for_infoblox_iq_assistant.md]

### SE Engagement and IQ Evals Expansion (May 2026)

Himanshu Raval reached out to the SE team (via Glenn Sullivan) on May 4, 2026 to drive two goals:

1. **Improve evals coverage:** IQ has expanded to DHCP, IPAM, Asset Insights, and Security, but coverage of question types across these domains is limited. SEs are being asked to submit question banks to expand the evaluation harness.
2. **Drive real usage:** SEs encouraged to use IQ in SE accounts and submit thumbs-up/down feedback with comments via the in-product feedback controls.

**Monthly hackathon series:** A recurring monthly "hackathon" session is in place for live IQ feedback. New capabilities ship every two weeks. Next session: May 5, 2026, 9am PST.

[Source: wiki/summaries/aw-_need_your_feedback_and_ddi,_uai_and_td_questions_for_infoblox_iq_assistant.md]

### SE Feedback on Desired IQ Features (Apr 2026)

Feedback from Solutions Engineer Stephen Holland (Apr 29) — SEs are using IQ as a workaround to query NIOS data because NIOS UI has no usable reporting. Key desired features:

- **Scheduling / routines:** run queries on intervals; notify via Teams/Slack when thresholds are met
- **Embedded visualizations:** bar/pie charts inline in IQ responses; exportable for external reporting
- **Actions/alerts:** convert a query + condition into an automated notification or action
- **Best-practice scans:** automated checks for common misconfigurations (NTP, DNS daemon, forwarder setup)
- **Chat history and sharable chats** (already roadmapped)
- **Replace Splunk** for common analytics with cheaper IQ-based reporting

IQ vs. NIOS data discrepancies observed (peak QPS: 132 vs. 152) — requires investigation. Data connector onboarding process (long strings, complex setup) should be simplified.

[Source: wiki/summaries/2026-04-29_iq_feedback_from_stephen_hollland.md]

### Configuration Audit and Forensics (Customer Need — May 2026)

EAP customers at the May 14, 2026 office hour identified configuration forensics as a first-class IQ use case, distinct from operational anomaly detection:

**Core ask:** IQ should answer questions like "who deleted the DNS record that is now causing NXDOMAINs?" and "what changed right before this outage started?"

**Specific requirements:**
- Surface change timelines showing what changed and when
- Identify whether a change came from the GUI, an API client, or a dynamic source (DDNS, Kea, NAD)
- Attribute changes to the specific user or service account that made them
- Correlate configuration events (upgrades, boot file edits on reservations, scope changes) with operational anomalies like NXDOMAIN spikes, DHCP floods, and serve-fail increases

**Product direction:** IQ will incorporate audit-log awareness so it can surface change timelines for classic post-mortem questions. DHCP/IPAM actions — which are fundamentally config-centric — are the primary initial domain. These capabilities extend IQ's usefulness into incident forensics, problem management, and post-mortem analysis beyond pure anomaly detection.

[Source: 2026-05-14_IQ_Product_Office_Hour_Whats_New_Whats_Next.md]

### Deployment and Migration Visibility (Customer Need — May 2026)

In early UDDI deployment and NIOS migration phases, customers lose CLI access and want IQ to provide better visibility into synchronization status, tunnel health, and early bring-up logs.

**Core ask:** When a device or tunnel does not come up, operators want IQ to answer:
- Which configuration elements failed to transfer?
- Which step of synchronization or X-as-a-Service provisioning failed?
- Which logs or events are relevant?

Today IQ is focused on steady-state operational metrics and cannot answer these questions well. The product and PM teams committed at the May 14 office hour to take these early-stage deployment pain points back as explicit user stories and to design new actions, queries, and prompts tailored to onboarding workflows.

In the interim, customers are encouraged to ask deployment-related questions directly to IQ and to capture weak or failed answers with thumbs-down feedback so the product team can build eval cases.

[Source: 2026-05-14_IQ_Product_Office_Hour_Whats_New_Whats_Next.md]

### EAP Feedback Themes

1. **Proactive alerting** — customers want IQ to detect and notify before they ask
2. **Integrations** — Slack, Teams, ServiceNow, and Jira are the top connector requests
3. **Action confidence** — customers want to see AI confidence scores before approving actions
4. **Runbook generation** — AI-generated SOPs from recurring incident patterns
5. **Copy/export** — table copy (IBIQ-685) requested by multiple customers; currently absent
6. **Stale navigation guidance** — IQ directs users to CSP menu paths that no longer exist, causing significant wasted time. Root cause: docs.infoblox.com is indexed but lags actual UI changes. Customers tolerate it when IQ delivers value (e.g., one customer spent 1.5 hours on a connector issue but still called it an "overall great success") — but it erodes trust. Distinct from hallucination: IQ is confidently correct about an out-of-date source. Also surfaces a **nomenclature problem**: "connector" means different things in different CSP contexts; IQ correctly diagnosed the ambiguity but the underlying UI taxonomy is confusing.
7. **Long-term baselining** — customers want IQ to compare current metrics against seasonal baselines, not just the last 30 days; critical for retail and event-driven environments
8. **Configuration audit** — customers want IQ to answer "who changed what right before this outage?" — correlating config events with operational anomalies
9. **Shareable prompts and reports** — customers want to pin useful prompts and assistant-generated dashboards for team-wide reuse, and to submit proven prompts back to Infoblox for productization
10. **Support case creation from IQ** — customers want to open Infoblox support tickets directly from an IQ action or visualization, pre-populated with logs and prompt context

[Source: 2026-05-14_IQ_Product_Office_Hour_Whats_New_Whats_Next.md (items 7–10)]

---

## GTM and Pricing

### IQ vs. IQ+ Tiering (Formalized May 2026)

As of the May 1, 2026 Launch Strategy Alignment deck, the IQ/IQ+ packaging is formally defined:


| Tier                  | Capability                                                                                                                                                                        | Pricing                                       |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| **IQ (base)**         | Natural-language AI Assistant — read-only, explains/audits/recommends; no write operations. Available across UDDI, NIOS, Threat Defense, and Asset Insights.                      | Included with base product — no extra license |
| **IQ+ (paid add-on)** | Full read + write capability — executes approved changes autonomously, multi-step agentic incident response, full audit trail, human-in-the-loop. DDI Actions + Security Actions. | Paid add-on ("Agentic Ops layer")             |


**IQ+ for Security:** Existing SOC Insights pricing adopted as-is; no new SKU required; seamless access for current SOC Insights customers.

**IQ+ for DDI:** Pricing TBD — candidate is 35% of UDDI tokens (matching the IQ for Security rate card). Finalize by May 15, 2026.

**Token-based TD customers:** Self-serve path to IQ+ exists; token allocation must be disclosed before enablement; IQ-specific T&C acceptance TBD.

**Open decision (May 8 deadline):** Whether AI-assisted widgets, dashboards, or queries within Asset Insights that invoke an AI assistant should be narrow/task-scoped (IQ base) or full-capability (IQ+).

### Launch Events

- **External launch (press release + Cisco Live):** July 2026 — IQ+ for Security GA
- **Internal launch:** Infoblox SRS (August 2026)

### Strategic Partners


| Partner  | Status                                                     |
| -------- | ---------------------------------------------------------- |
| Cisco    | Confirmed — MCP Server (API key, read-only) for Cisco Live |
| HPE Mist | To be explored                                             |
| AWS      | To be explored                                             |
| GCP      | To be explored                                             |


### Positioning

- IQ (base) is included in UDDI and SaaS DDI subscriptions; not a standalone SKU
- IQ+ (paid add-on) is the agentic tier with DDI and Security Actions
- IQ Security (SOC Insights) retains existing pricing; seamless access for existing customers
- AI-attributable ARR reported to Vista board; SOC Insights newer installs qualify

### Enterprise Scaling — Target Customer Sizing

As of May 3, 2026, Himanshu initiated an exercise to characterize environment sizes for the five highest-interest enterprise DDI customers before broader IQ availability:

1. Exxon Mobil
2. American Airlines
3. Nationwide
4. Salesforce
5. Target

**SP exclusion:** IQ is explicitly **not targeting** service providers like T-Mobile or Verizon at this stage — their scale and desire to build proprietary agentic apps make them a poor fit. Infoblox IQ targets mid-to-large enterprise.

**Data availability:** Logs, metrics, token usage, and NIOS HEKA data all flow to the data lake (Databricks). Alex and Hemanth can pull custom environment sizing reports.

[Source: wiki/summaries/re-_scaling_iq_for_enterprise_customers_of_ddi.md]

### MCP Server as GTM Lever

The IQ MCP server (see [MCP-Server-Strategy.md](MCP-Server-Strategy.md)) is a distribution mechanism: customers integrate IQ into Slack, ServiceNow, or their own AI agents. Each integration deepens stickiness and creates natural expansion conversations.

---

---

## Competitive Intelligence

### BlueCat (May 2026)

BlueCat announced on April 30/May 1, 2026 that it is launching:

- **MCP Servers** (tech preview now; GA via public registry + included with base licenses **July 2026**)
- **LiveAssist** — their AI virtual engineer — expanding to **DDI starting July 2026** with usage-based pricing on BlueCat Horizon SaaS

**Timeline clash:** BlueCat's DDI+AI launch and MCP Server GA are both targeted for July 2026 — the same window as Infoblox IQ+ for Security GA and the IQ press release at Cisco Live. Infoblox's external MCP server at Cisco Live (end of May) goes live before BlueCat's July public registry GA.

**Company context (external research, May 2026):**

- Revenue: >$100M ARR; revenue tripled in 3 years under Audax PE ownership (since 2022)
- Acquired LiveAction (network observability: LiveNX, LiveWire) in October 2024 — adds flow analysis, packet capture, and the existing LiveAssist AI to the DDI portfolio
- BlueCat Horizon (SaaS-first unified NetOps platform) launched February 2026 — still tech preview
- Audax running a **$1.5B–$2B sale process via JPMorgan** (January 2026) — a potential strategic acquirer could materially change BlueCat's competitive posture
- Peer review momentum: PeerSpot IPAM #1 ranking (8.8 rating, 96% recommend); Gartner 4.5★ / 147 reviews; mindshare growing (13.3% → 17.4%) while Infoblox's declines (34.4% → 27.7%)

**Key BlueCat differentiators (per external analysis):**

- 20–30% cheaper than Infoblox per multiple independent customer reviews
- API-first architecture (every UI action = API call; OpenAPI compliant) — strong appeal to DevOps/IaC buyers
- Micetro overlay: mid-market DDI without rip-and-replace; installs in <1 hour; supports MSFT DNS/DHCP, BIND, Kea, Meraki, Route 53, Azure DNS. Infoblox has no equivalent.
- Cisco SolutionsPlus / Global Price List: enterprise procurement through Cisco SIs
- LiveAction portfolio: LiveNX + LiveWire + LiveAssist for network observability — significantly stronger than Infoblox in this domain

**Where Infoblox has durable advantages:**

- **Security AI:** BlueCat's entire AI strategy is NetOps-only — no threat detection, no SOC triage, no security actions. IQ Security (SOC Insights ~$2M ARR) GA July 2026 has no BlueCat equivalent.
- **Asset intelligence:** IQ Assets (device discovery, lifecycle, vulnerability correlation, comply-to-connect) is unopposed in BlueCat's portfolio.
- **Enterprise MCP security model:** Infoblox's OIDC/OAuth + RBAC masking + approval workflows + full audit trail design is technically mature. BlueCat's MCP has no published technical architecture, auth model, or audit logging.
- **Production AI customers:** 7 active Fortune 500 EAP testers (Target, ExxonMobil, American Airlines, Nationwide, etc.) vs. BlueCat's partner quote only.
- **Federal/FedRAMP:** FedRAMP authorization underway (3PAO audit Kratos, May 2026). BlueCat's FedRAMP Authorized status is unconfirmed.

**Infoblox internal assessment (Himanshu Raval, May 1):** "Much hoopla and fuzziness with not much substance" — customer quote from a partner (Techary UK), not a DDI customer. Assessment is accurate but: the field will still face the question in sales cycles.

**Key gap to address:** BlueCat's "AI included with base licenses" pricing claim requires a clear Infoblox counter. The counter: IQ base (read-only AI) is also included with product — IQ+ charges only for agentic write operations (DDI Actions + Security Actions), which BlueCat has not announced at all.

**Recommended field message:**

1. "Infoblox MCP is in customer hands at Cisco Live. BlueCat's is a July roadmap item."
2. "BlueCat's AI tells you when your network is slow. Infoblox's AI tells you when your network is under attack — and takes action."
3. "Ask BlueCat what happens when their AI autonomously changes a DNS record. We have OIDC/OAuth, RBAC-controlled write masking, approval workflows, and a full audit trail. They haven't published their architecture."

**Action:** Paul Nguyen + Ward Cobleigh producing a field competitive response for BlueCat. Jon Abbe / Himanshu as reference for IQ differentiators. Monitor BlueCat sale process — strategic acquisition would change dynamics materially.

**June 2 announcement update (May 20, 2026):** Internal alignment on the June 2 announcement package is in progress. Paul Nguyen shared v3 draft (includes Walnut demo). Product name confirmed as "IQ for DDI" per Mukesh. Messaging will use "Infoblox DDI" (umbrella term) rather than UDDI or NIOS specifically. DNS-AID is not mentioned in the announcement. Ward Cobleigh reviewing the package; Mehul Patel reviewing the IQ content. Two FAQ documents (Les Dunston technical FAQ + Mehul Patel sales FAQ) need consolidation before announcement. Dave Funk is PM for IQ/NIOS.

[Source: wiki/summaries/re-_bluecat_announcement.md, wiki/summaries/re-_bluecat_announcement_2.md, outputs/BlueCat_MCP_Competitive_Analysis_2026-05-04.md, outputs/BlueCat_Deep_Competitive_Intelligence_2026-05-04.md]

---

## Stakeholders

### Product and GTM


| Name            | Role                                                 |
| --------------- | ---------------------------------------------------- |
| Himanshu Raval  | PM — IQ DDI core                                     |
| Jon Abbe        | PM — GTM, architecture, customer engagement, pricing |
| Padmini Kao     | Platform strategy; IQ positioning and roadmap        |
| Turner          | Data, analytics, CubeJS layer                        |
| Tejasvi         | UX design — platform                                 |
| Chaitrali Dhole | UX design — IQ-specific                              |
| Rockwell        | Dashboard and visualization                          |


### Engineering and Architecture


| Name             | Role                              |
| ---------------- | --------------------------------- |
| Tom Hayward      | Architect — IQ platform           |
| Daniel Garcia    | Engineering lead — IQ backend     |
| Julian           | UI/frontend engineering           |
| Paul Lawrence    | DHCP/IPAM actions                 |
| Karthik Haridoss | SOC integration, security actions |
| Matthew Landry   | DDI coordination                  |
| Devendra         | Asset Insights AI integration     |


---

## Key Decisions


| Decision                                                            | Date     | Rationale                                                                                   |
| ------------------------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------- |
| Azure OpenAI as primary LLM; LiteLLM for abstraction                | 2025     | Cost control, model flexibility                                                             |
| Hybrid determinism (NL→IQL→user edits→run)                          | 2025     | Keeps humans in the loop for mutations                                                      |
| Global search invoked before LLM                                    | 2025     | Reduces hallucination on entity lookups                                                     |
| Prompt registry via Databricks                                      | 2025     | Version control and A/B testing for prompts                                                 |
| SOC Insights repositioned under IQ umbrella                         | Q1 2026  | Vista board AI ARR requirement                                                              |
| Tri-state Actions permission model                                  | 2025     | Needs Approval as safe default                                                              |
| ~500 SME Q&A pairs required before production guardrail rollout     | 2026     | False positive risk management                                                              |
| Chat history deferred to July 2026 milestone                        | 2026     | Engineering capacity; not Phase 1 scope                                                     |
| MCP server as external integration mechanism                        | 2026     | Enables Slack/ServiceNow/Salesforce without custom connectors                               |
| DDI Actions to remain preview/EAP at initial GA; not fully GA       | Apr 2026 | Quality and completeness not yet at bar; leadership will not call GA until resolved         |
| GA timing shifted to September/October preference                   | Apr 2026 | Board demo surfaced reliability/quality gaps; rushing July/August risks reputational damage |
| Cross-product PM+design alignment session proposed (SOC/DDI/Assets) | Apr 2026 | Parallel decisions creating inconsistent flows and naming across products                   |


---

## Risks


| Risk                                                                           | Likelihood      | Mitigation                                                                                                                                                                                         |
| ------------------------------------------------------------------------------ | --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SME Q&A gap blocks production guardrail rollout                                | **High**        | Identify SME owners; timebox authoring                                                                                                                                                             |
| False positive rate destroys operator trust                                    | **High**        | Benchmark pre-GA; start with low-sensitivity thresholds                                                                                                                                            |
| Sprint overcommitment continues                                                | Medium-High     | TPM-led sprint running; enforce estimation discipline                                                                                                                                              |
| Stale RAG index causes outdated answers                                        | Medium          | Automated index refresh pipeline                                                                                                                                                                   |
| Stale source documentation causes wrong navigation guidance                    | **Medium-High** | docs.infoblox.com indexed but content lags CSP UI changes — confirmed by EAP customer (Apr 2026); index refresh doesn't fix this; requires docs team action or MCP-based live portal introspection |
| API object mapping hallucination (Access View / Compartments)                  | **High**        | IQ claims APIs don't exist when they do, then generates wrong endpoints — distinct from stale docs; incomplete API schema in RAG corpus; confirmed UDDIST-2390 (May 2026) |
| Shared service load degrades IQ response time                                  | Medium          | Dedicated inference capacity; queue prioritization                                                                                                                                                 |
| LLM cost creep as usage scales                                                 | Medium          | LiteLLM routing; model tiering by query complexity                                                                                                                                                 |
| IQ Security GA blocked by PSB review                                           | Medium          | Start PSB process early; no security-action GA without sign-off                                                                                                                                    |
| UDDI ClickHouse migration incomplete: degrades IQ response time                | **Medium-High** | IQ query latency directly affected; not visible in current roadmaps; must be tracked and prioritized                                                                                               |
| Cross-product divergence (SOC/DDI/Assets): inconsistent agent settings, naming | Medium          | Proposed PM+design alignment session to harmonize; global agent settings deprioritized until ownership clear                                                                                       |


---

## Source Files

**96 raw files + 27 wiki summaries · Jun 2025 → Apr 2026** (absorbed from: AI-Evaluation-Quality, Agentic-Workflows, Asset-Insights, DNS-DHCP-Operations)

Key sources: `wiki/summaries/` IQ series, IBIQ Jira tickets (IBIQ-266, -648, -685, -707, -852, -889, -1162, -1180, -1200, -1201), Dream Team IQ sessions, EAP customer feedback files, Sprint 4–7 retrospectives, SOC Insights repositioning discussions, IQ Architecture sync files, guardrail and evaluation framework docs, DNS/DHCP top-issues tracking spreadsheets, UX design review sessions (Feb–Mar 2026).

**See also:** [[MCP-Server-Strategy]] · [[FedRAMP-Certification-Strategy]] · [[Maryland-Program]] · [[RBAC-Platform-Strategy]] · [[UX-Design-Direction]] · [[Reporting-Platform]] · [[Halo-Integration]] · [[competitive-intelligence]]