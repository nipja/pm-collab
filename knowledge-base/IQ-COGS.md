---
title: "IQ COGS & Unit Economics"
created: 2026-05-20
last_updated: 2026-05-21
source_count: 3
status: reviewed
summary: "IQ cost structure and incrementality are under active review. Original analysis attributed DNS pipeline cost to IQ; IQ architect Tom Hayward clarified this is a UDDI feature, not IQ-specific. LLM inference is likely the primary IQ-incremental cost."
type: strategy
tags: [iq, cogs, pricing, unit-economics]
---

# IQ COGS & Unit Economics

> ⚠️ CONTRADICTION (partially resolved): Original analysis claimed DNS query pipeline ($0.22/million queries) is the dominant IQ cost driver and nearly all IQ-incremental. IQ architect Tom Hayward clarified (2026-05-20) that the data pipeline is a UDDI reporting feature — it exists regardless of IQ and is "helpful to IQ but not required." However, a separate NIOS-specific COGS analysis from Laksh Lumba (2026-05-20) treats the pipeline as fully incremental for NIOS (on-prem) customers, because NIOS has no existing cloud data pipeline. **Partial resolution:** The $0.22/million queries pipeline cost IS incremental for NIOS customers (pipeline is 100% new); it is NOT incremental for cloud (UDDI) customers (existing reporting infrastructure). For UDDI customers, LLM inference ($0.012/request) is the primary IQ-incremental cost. — Sources: Tom Hayward IQ architect; Laksh Lumba NIOS COGS email 2026-05-20.

The figures below reflect the NIOS-specific COGS context from Laksh Lumba's analysis (applicable to NIOS on-prem customers) alongside the broader unit economics.

---

## Cost Components

### 1. DNS Query Pipeline

**$0.22 per million queries** — the dominant cost driver.

Cost scales linearly with DNS query volume (QPS). There is no tiering, compression benefit, or cost cap in the current model.

Monthly cost formula: `(QPS × 2,592,000) / 1,000,000 × $0.22`

Example: 23,614 QPS = ~61 billion queries/month = **~$13,465/month**

[Source: 2026-05-20_IQ_COGS_Unit_Economics_Analysis.md]

### 2. LLM Inference

**$0.012 per request**, averaging ~11K tokens per request.

Secondary to pipeline cost at current volumes. At 100K requests/month the cost is ~$1,200 — meaningful but not dominant. Becomes more significant as IQ adoption and request volume grow.

[Source: 2026-05-20_IQ_COGS_Unit_Economics_Analysis.md]

### 3. Baseline Infrastructure

**~$1,500/month** per customer (approximate).

Covers control plane, multi-tenant services, and analytics infrastructure. Partially exists in baseline deployments but scales up significantly with IQ enabled.

[Source: 2026-05-20_IQ_COGS_Unit_Economics_Analysis.md]

---

## Customer Cost Examples

| Customer | QPS | Monthly Cost |
| --- | --- | --- |
| American Airlines (large grid) | 23,614 | $13,465 |
| American Airlines (small grid) | 602 | $343 |
| ExxonMobil | 44.9 | $25.60 |

The QPS-to-cost relationship is near-perfectly linear: American Airlines' large vs. small grid is ~39x the traffic and ~39x the cost. There is no anomaly or multiplier effect; scale is the only driver.

[Source: 2026-05-20_IQ_COGS_Unit_Economics_Analysis.md]

---

## What Costs Are Incremental to IQ

> ⚠️ This section reflects the original analysis and is under revision. Tom Hayward (IQ architect, 2026-05-20) stated the data pipeline is a UDDI feature, not IQ-specific. The table below should be treated as disputed until confirmed.

| Cost Component | Original claim | Hayward correction |
| --- | --- | --- |
| DNS resolution (NIOS core) | Yes — on-prem, already paid | Unchanged |
| Basic logging | Yes (limited) | Unchanged |
| Full query streaming (UDDI) | No — IQ-driven | Disputed: UDDI reporting feature, exists without IQ |
| Cloud storage (OpenSearch/ClickHouse) | No — IQ-driven | Disputed: part of UDDI reporting infrastructure |
| LLM inference | No — IQ-only | Confirmed: 100% IQ-incremental |

**Revised key insight (pending confirmation):** For UDDI customers, the pipeline already exists. IQ's primary incremental cost is LLM inference ($0.012/request). For NIOS customers, no pipeline exists today, but Tom Hayward indicates it is not required for IQ — suggesting NIOS IQ may also have LLM inference as its primary incremental cost. Open question: what data does IQ for NIOS actually query against?

[Source: 2026-05-20_IQ_COGS_Unit_Economics_Analysis.md; correction from Tom Hayward, IQ architect, 2026-05-20]

---

## Cloud vs. On-Prem Cost Structure

The cost profile differs significantly between cloud (UDDI) and on-prem (NIOS) customers:

**Cloud (UDDI):** Data is already flowing through Infoblox infrastructure. The marginal pipeline cost to add IQ is lower because partial data infrastructure exists. T&Cs already accommodate data collection. Current pricing proposal: ~35% of UDDI management tokens (same model as IQ for Security).

**On-prem (NIOS):** The full data pipeline is 100% net-new. Streaming raw DNS from NIOS appliances to cloud analytics does not exist today — IQ creates it entirely. Full $0.22/million queries cost is incremental with no existing infrastructure to offset it. Additionally, NIOS standard T&Cs explicitly state that Infoblox does **not** collect or store IP addresses, even for Call Home/Heka data. IQ breaks this contract. A T&C addendum covering OpenAI and the full IQ AI stack is a legal gate before any NIOS IQ can be commercially deployed.

**Implication:** NIOS IQ has higher COGS, unresolved pricing (as of May 15, 2026), and a legal T&C gate — three compounding risks not present for cloud customers.

[Source: IQ Launch Alignment deck (May 1, 2026), IQ Pricing Kickoff (May 15, 2026)]

---

## Strategic Implications

**For pricing:** The current model is purely usage-based with no tiering. Large enterprise customers (airlines, energy majors) generate vastly more DNS traffic and therefore have structurally higher COGS. Revenue must scale accordingly or margins are squeezed. The pricing model must be QPS-aware — flat token or per-seat pricing without a QPS component will compress margin on the largest enterprise accounts. See [[Tokens-and-Commercialization]] for the pricing model and universal token strategy.

**For IQ Assistant (free tier):** IQ Assistant is positioned as $0 to drive adoption; IQ Actions are the paid surface. LLM cost at $0.012/request is manageable at current volumes but will grow as agentic workflows (AI Operations, autopilot, multi-step pipelines) ramp up. A usage gate or threshold will eventually be needed.

**For cost optimization:** Three levers identified:
1. Query sampling, filtering, or deduplication to reduce pipeline volume
2. QPS-based pricing tiers to align revenue with cost
3. Decoupling storage from real-time processing to reduce ingestion cost

**For customer segmentation:** Service providers (T-Mobile, Verizon) are excluded from IQ targeting specifically because their QPS would make COGS unviable. The upper QPS boundary for viable enterprise customers is undefined in the current model and needs to be set. See [[IQ-Strategy]] for enterprise segmentation rationale.

---

## Related

- [[Tokens-and-Commercialization]] — pricing model, universal token, PAYGO
- [[IQ-Strategy]] — product strategy, EAP customers, enterprise segmentation
- [[Platform-Architecture]] — OpenSearch, ClickHouse, data lake infrastructure
