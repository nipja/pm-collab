---

## title: IQ Architecture
created: 2026-05-21
last_updated: 2026-05-21
source_count: 8
status: reviewed
summary: "Complete technical architecture for Infoblox IQ: two bounded contexts (AI Agent, Insighter), shared MCP Tool layer, authorization model (DESTRUCTIVE_NOOP, ext_authz bridge, OPA), LiteLLM gateway, and 4-milestone customer-facing MCP roadmap."
type: platform
tags: [iq, platform, architecture, mcp]

Infoblox IQ is built on two bounded contexts — AI Agent Context and Analytics & Insights Context (Insighter) — that share a centralized MCP Tool layer for API access. All components run on the Kubernetes-based SaaS platform. The product is owned by team-iq. Key technology deviations: Python for AI services (LangChain/LangGraph requirement), Go for Insighter (platform standard), and Rust for the agentgateway binary (requiring a standalone Go ext_authz bridge for OPA integration).

---

## Bounded Contexts

### AI Agent Context

Handles conversational AI: user query intake, NLP, tool orchestration, and response generation.


| Service          | Language | Role                                                                                      |
| ---------------- | -------- | ----------------------------------------------------------------------------------------- |
| IQ NLP Interface | Go       | REST entry point from Portal; manages conversation history; gRPC to IQ Agent              |
| IQ Agent         | Python   | LangChain/LangGraph ReAct agent; stateless per request; connects to MCP Gateway for tools |


**Design:** IQ Agent is fully stateless. Conversation history is passed in each gRPC request from IQ NLP Interface (stored in PostgreSQL/RDS with 30-day TTL). Horizontal scaling via HPA (1–3 replicas, max_workers=1 per pod for ReAct memory isolation).

### Analytics & Insights Context (Insighter)

Handles telemetry ingestion, event transformation, root-cause analysis, action lifecycle, and network topology.


| Service                  | Language | Role                                                             |
| ------------------------ | -------- | ---------------------------------------------------------------- |
| Insighter Gateway        | Go       | gRPC/REST API entry point for analytics                          |
| Insighter Ingester       | Go       | gRPC event ingestion; publishes to Dapr PubSub                   |
| Insighter Transformer    | Go       | Four-stage CEL pipeline; applies rules without redeployment      |
| Insighter DBStore        | Go       | ClickHouse storage abstraction (DAL for read APIs)               |
| Insighter Research Agent | Go       | LLM-powered root-cause analyst; shares MCP servers with IQ Agent |
| Insighter Actions        | Go       | Recommendation/action lifecycle (PostgreSQL)                     |
| Insighter Codex          | Go       | Knowledge base (pgvector/RDS)                                    |
| Insighter Topology       | Go       | Neo4j-backed network topology queries                            |


**Insighter Pipeline:** External events + NIOS/NIOSx telemetry → Kafka (Dapr pub/sub) → Transformer instances (one per topic, CEL rules) → Ingester gRPC → insighter.eventbus → Data Loader → ClickHouse. Enrichers concurrently augment events (recursive enrichment possible via Research Agent publishing back).

**Tech Debt:** TD-006 — Dapr pub/sub backed by Kafka instead of platform Pulsar standard.

### MCP Tool Context (Shared)

Provides the API access layer for both IQ Agent and Insighter Research Agent.


| Service                 | Language            | Role                                                                                               |
| ----------------------- | ------------------- | -------------------------------------------------------------------------------------------------- |
| MCP Gateway             | Rust (agentgateway) | Centralized aggregation point; routes both internal (IQ Agent) and external (customer) MCP traffic |
| ext_authz Bridge        | Go                  | Standalone Envoy ext_authz v3 gRPC service; enforces authorization at gateway layer (ADR-42)       |
| MCP-CSP                 | Python (FastMCP)    | Access to Infoblox Portal REST APIs and CubeJS analytics                                           |
| MCP Insighter           | Go                  | ClickHouse data queries and telemetry access                                                       |
| MCP HostApp             | Go                  | Host App platform access                                                                           |
| MCP Docsearch           | Go                  | Product documentation search                                                                       |
| MCP On-Prem Diagnostics | Go                  | On-premises appliance diagnostics                                                                  |


**Note:** MCP Insighter and MCP HostApp are shared between IQ Agent (via MCP Gateway) and Insighter Research Agent (via direct streamableHTTP).

---

## Communication Patterns


| From                      | To                               | Protocol                   | Pattern                 | Purpose                                             |
| ------------------------- | -------------------------------- | -------------------------- | ----------------------- | --------------------------------------------------- |
| Portal                    | IQ NLP Interface                 | REST (HTTPS)               | Synchronous             | User query submission                               |
| Portal                    | Insighter Gateway                | REST (HTTPS)               | Synchronous             | Analytics queries, insight browsing                 |
| Customer MCP Client       | ingress-nginx → MCP Gateway      | StreamableHTTP over HTTPS  | Synchronous             | External customer MCP tool access via Identity OIDC |
| IQ NLP Interface          | IQ Agent                         | gRPC (QueryFlow v2)        | Synchronous + streaming | Query execution with conversation history           |
| IQ Agent                  | MCP Gateway                      | StreamableHTTP (MCP)       | Synchronous             | Tool discovery and invocation (cluster-internal)    |
| IQ Agent / Research Agent | LiteLLM                          | HTTPS                      | Synchronous             | LLM inference routing (ADR-27)                      |
| LiteLLM                   | Azure OpenAI                     | HTTPS over AWS PrivateLink | Synchronous             | LLM inference                                       |
| MCP-CSP                   | CSP REST APIs                    | HTTPS                      | Synchronous             | All API operations                                  |
| Transformer/Enrichers     | Ingester                         | gRPC                       | Synchronous             | Event ingestion                                     |
| Kafka (Dapr)              | Transformer/Enricher/Data Loader | Dapr pub/sub               | Async                   | Event delivery                                      |


**User Query Flow (full trace):**  
User → Portal → REST POST /query (JWT) → IQ NLP Interface → Load conversation history → gRPC QueryFlowRequest (JWT + query + history) → IQ Agent → Extract account_id from JWT → Compact history if > 10 messages → LLM tool-call decision → Invoke tool via MCP Gateway (JWT propagated) → MCP-CSP → CSP API → Tool result → LLM continuation → Final markdown response → Stream ChatMessage tokens → IQ NLP Interface stores to history → Portal renders

---

## Authorization Model

### Current Production State (pre-IBIQ-1193)

- MCP Gateway forwards all traffic without enforcing its own authorization
- Safety guarantee for writes: **DESTRUCTIVE_NOOP=True** in MCP-CSP — write tool calls return API usage instructions instead of executing, ensuring no mutation of customer infrastructure
- Downstream CSP APIs enforce their own RBAC against the propagated JWT
- **No OPA sidecars** in IQ Agent services; authorization fully delegated to downstream services
- OPA sidecars present in Insighter Gateway only

### Target State (post-IBIQ-1193, ADR-42)

The ext_authz bridge enforces authorization at the MCP Gateway layer:

1. agentgateway calls ext_authz bridge (gRPC Check()) for every request
2. Bridge extracts JWT claims (account_id, user_id, groups) and MCP attributes (tool_name, tool_namespace, method)
3. Bridge classifies tool by destructiveHint annotation: mcp-read or mcp-write
4. Bridge calls OPA sidecar via atlas-authz-middleware AffirmAuthorization()
5. Allow or deny; fail-closed (OPA/bridge unreachable = deny all)

**Permission model:**


| Annotation                    | Permission                 | OPA fullMethod        |
| ----------------------------- | -------------------------- | --------------------- |
| destructiveHint: true         | mcp-write                  | mcp-gateway.mcp-write |
| destructiveHint: false/absent | mcp-read                   | mcp-gateway.mcp-read  |
| Tool not in cache             | mcp-write (secure default) | mcp-gateway.mcp-write |


**PARG Roles:** mcp-reader (mcp-read only), mcp-writer (mcp-read + mcp-write)

**Why a standalone bridge?** agentgateway is Rust and cannot embed the platform's Go `authz-grpc-middleware`. The standalone bridge is a new pattern (first service using it) and is reusable for other non-Go services needing OPA integration.

### Write Operations Roadmap

- **Today:** DESTRUCTIVE_NOOP=True blocks all writes at MCP-CSP layer
- **IBIQ-1193 + IBIQ-926 + PTCI-3285 (Milestone 1):** Agent RBAC via ext_authz bridge; agents default read-only; admins grant write access
- **IBIQ-648 + IBIQ-1228 (Milestone 2):** Per-API human-in-the-loop approval; DESTRUCTIVE_NOOP turned off; writes execute behind explicit "Allow / Deny" confirmation

---

## LLM Infrastructure (ADR-27)

All LLM calls from IQ Agent and Insighter Research Agent route through **LiteLLM**, deployed on the shared services EKS cluster.

- Single OpenAI-compatible endpoint translating to Azure OpenAI (active); AWS Bedrock (future)
- Provider keys in HashiCorp Vault with rotation
- Per-team virtual API keys with budget/model restrictions
- App clusters connect via AWS PrivateLink (no public internet)
- Automatic failover: Azure US-EAST-1 → US-EAST-2
- Separate LiteLLM instances planned for EU and GCP (compliance boundaries)
- Open-source version; enterprise ($42K/year) deferred

**Current LLM dependency risk:** IQ Agent has no multi-model fallback (TD-IQ-004). Azure OpenAI outage = full IQ Agent service outage.

---

## Customer-Facing MCP — 4-Milestone Roadmap

As of 2026-05-01, the customer-facing MCP server is live but unadvertised and read-only.


| Milestone     | Customer capability                                    | Key epic                                 | Status                 |
| ------------- | ------------------------------------------------------ | ---------------------------------------- | ---------------------- |
| 0 — Today     | API key auth, read-only, no marketing                  | IBIQ-31, IBIQ-1200                       | Live                   |
| 1 — RBAC      | Agent scope restriction; least-privilege default       | IBIQ-1193, IBIQ-926, IBIQ-957, PTCI-3285 | In flight              |
| 2 — Approvals | Per-API human-in-the-loop write confirmation           | IBIQ-648, IBIQ-1228, IBIQ-958            | PI 26.3                |
| 3 — OIDC      | SSO login; Claude Connector listing; full GA marketing | IBIQ-40, PTCI-4167/4176                  | **IBIQ-40 P1 Blocked** |


**IBIQ-40** (Make Identity an OAuth 2.1/OIDC provider for MCP, owned by Platform) is the largest single GA-marketing blocker for customer-facing MCP. It is not on the IQ roadmap — it is the Platform dependency to escalate.

---

## Repositories


| Repo                          | Contents                                                                                                     |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Infoblox-CTO/iq-eva           | IQ Agent, MCP-CSP, MCP Shared                                                                                |
| Infoblox-CTO/iq-nlp-interface | IQ NLP Interface                                                                                             |
| Infoblox-CTO/iq-insighter     | All Insighter services (20+ Go services), MCP Insighter, MCP HostApp, MCP Docsearch, MCP On-Prem Diagnostics |
| Infoblox-CTO/mcp-gateway      | MCP Gateway (agentgateway operator), ext_authz bridge                                                        |


---

## Cross-Product Dependencies

- **Platform:** atlas_cubejs (analytics), Identity Service (OIDC/auth), Notifications (Atlas)
- **UDDI / Threat Defense / Assets:** accessed via MCP-CSP through CSP REST APIs
- **NIOS / NIOSx:** Insighter ingests telemetry from appliances via Kafka
- **Databricks (MLflow):** IQ Agent exports traces for debugging/evaluation

---

## Data Handling

- **Prompt content to Azure OpenAI:** User's query + system prompt + tool results scoped to calling user's tenant
- **Conversation history:** Carried in-request; stored by IQ NLP Interface in PostgreSQL/RDS (30-day TTL); compacted at > 10 messages
- **Session storage (MCP-CSP):** Tool results persisted as Parquet on AWS S3 (per-session, tenant-scoped; lifecycle TTL to be confirmed — see TD-IQ-007)
- **Traces:** IQ Agent exports per-request traces to Databricks via MLflow (async, fire-and-forget); internal engineering use only; retention TBD
- **No training on customer data:** Infoblox does not fine-tune models on customer prompts. Azure OpenAI enterprise tier with no Microsoft-side prompt/response retention.

---

## Key Design Decisions Summary


| Decision                           | Rationale                                                   | Reference |
| ---------------------------------- | ----------------------------------------------------------- | --------- |
| Python for AI services             | LangChain/LangGraph/MLflow ecosystem                        | —         |
| Go for Insighter                   | Platform standard; high-performance pipeline                | —         |
| MCP protocol for tools             | Standard; reusable across LLM platforms                     | —         |
| Standalone ext_authz bridge        | agentgateway is Rust; cannot embed Go middleware            | ADR-42    |
| LiteLLM as centralized LLM gateway | Avoid per-team provider fragmentation                       | ADR-27    |
| Stateless IQ Agent                 | Enables horizontal scaling                                  | —         |
| DESTRUCTIVE_NOOP=True              | Safety guarantee until approval workflows land              | IBIQ-648  |
| ClickHouse for Insighter analytics | Columnar OLAP, time-series optimized                        | —         |
| Dapr pub/sub / Kafka               | High-volume event ingestion (deviates from Pulsar standard) | TD-006    |


---

## Source Documents

- [[summaries/iq-architecture-overview]] — IQ-Architecture-Overview.pdf
- [[summaries/iq-architecture-detailed]] — IQ-Architecture-Detailed.pdf
- [[summaries/iq-service-agent]] — IQ-Service-Agent.pdf
- [[summaries/iq-service-mcp-csp]] — IQ-Service-MCP-CSP.pdf
- [[summaries/iq-service-mcp-gateway]] — IQ-Service-MCP-Gateway.pdf
- [[summaries/iq-adr-027-litelllm-gateway]] — IQ-ADR-027-LiteLLM-Gateway.pdf
- [[summaries/iq-adr-042-extauthz-bridge]] — IQ-ADR-042-ExtAuthz-Bridge.pdf
- [[summaries/iq-customer-facing-mcp-state]] — IQ-Customer-Facing-MCP-State.pdf
- [[summaries/re-_legal_requirements_before_ga_iq_2]] — Tom Hayward's authoritative legal Q&A (May 20, 2026)

## Related Articles

- [[MCP-Server-Strategy]] — MCP strategy and customer-facing roadmap
- [[IQ-Strategy]] — product vision, features, GA timeline
- [[IQ-COGS]] — cost structure and unit economics
- [[RBAC-Platform-Strategy]] — OPA and RBAC platform work (PTCI-3285)
- [[Compliance-Program]] — legal and privacy requirements for IQ GA