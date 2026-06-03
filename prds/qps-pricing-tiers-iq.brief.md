# Problem Brief — QPS-Based Pricing Tiers for IQ

**Status:** Draft for review  
**Created:** 2026-06-03  
**Author:** Product sprint / framing step

---

## Sharpened Problem Statement

Infoblox IQ's cost structure scales near-linearly with DNS query volume (QPS), but the current pricing model does not. A flat token allocation (e.g., ~35% of UDDI management tokens for IQ Security) or a flat per-seat model produces the same revenue regardless of whether a customer generates 45 QPS (ExxonMobil: ~$26/month in pipeline cost) or 23,614 QPS (American Airlines large grid: ~$13,465/month). As enterprise customers scale and IQ adoption grows beyond EAP, this misalignment will compress gross margin on the highest-volume accounts and potentially make IQ economically unviable for large grids — even at the aggressive end of the enterprise market.

The core tension: **Infoblox cannot price IQ as if all enterprise customers are equal when their underlying cost-to-serve varies by 500×.**

---

## Who Is Affected

| Persona | Pain | Evidence |
|---|---|---|
| **Finance / GTM** | Margin compression on large-grid enterprise accounts; no mechanism to charge more for higher COGS | American Airlines large grid: $13,465/month pipeline cost with no matching revenue signal [Source: IQ-COGS.md] |
| **Sales / Solutions** | No coherent answer to "what does IQ cost me?" for high-QPS prospects; struggle to price large deals | SRS 2026 set as deadline for universal token pricing finalization [Source: IQ-Strategy.md] |
| **Product** | Service providers (T-Mobile, Verizon) excluded from IQ targeting entirely because their QPS makes COGS unviable — an upper boundary that has never been formally set [Source: IQ-COGS.md] |
| **Customers (large grids)** | Potentially face sticker shock if pricing is retroactively corrected after GA without a clear tier model established early |

---

## Scope

### In scope
- Define QPS-based pricing tier bands for **IQ for DDI** (the cost-driver is DNS query pipeline)
- Clarify how tiers interact with the existing **universal token** model
- Address **NIOS vs. UDDI cost asymmetry**: pipeline is incremental for NIOS; shared infrastructure for UDDI — tiers may differ
- Set an explicit **upper QPS ceiling** above which IQ is not offered (service provider exclusion)
- Define how tiers affect **packaging**: flat fee per tier, overage model, or token-rate adjustment

### Out of scope
- IQ for Threat Defense pricing (different cost driver: LLM inference, not DNS pipeline)
- IQ for Asset Insights pricing (not yet finalized; dependent on data model)
- Free vs. paid tier boundary for the AI Assistant (separate unresolved debate — Himanshu vs. Mehul)
- Engineering changes to enforce tiers in-product (downstream of pricing decision)

---

## Known Facts (sourced)

1. **Pipeline cost is $0.22/million DNS queries**, scaling linearly with QPS. Monthly formula: `(QPS × 2,592,000) / 1,000,000 × $0.22`. [Source: IQ-COGS.md]

2. **Cost range spans 500×** across confirmed EAP customers: ExxonMobil ($25.60/month) → American Airlines large grid ($13,465/month). [Source: IQ-COGS.md]

3. **Pipeline cost is NOT IQ-incremental for UDDI customers** — the data pipeline exists for UDDI reporting regardless of IQ. Primary incremental cost for UDDI customers is **LLM inference at $0.012/request**. [Source: IQ-COGS.md — Tom Hayward correction, 2026-05-20]

4. **Pipeline IS 100% IQ-incremental for NIOS customers** — no existing cloud pipeline today. NIOS also has a T&C gate (PII telemetry addendum required) before commercial deployment. [Source: IQ-COGS.md, IQ-Strategy.md]

5. **Service providers explicitly excluded** from IQ targeting because their QPS makes COGS unviable; exact threshold is undefined. [Source: IQ-COGS.md]

6. **Current IQ pricing placeholder:** ~35% of UDDI management tokens for IQ Security/TD. No finalized pricing model for IQ DDI. [Source: IQ-Strategy.md]

7. **Universal token pricing for IQ DDI and Asset Insights** to be finalized at SRS 2026. [Source: IQ-Strategy.md — BlueCat field response v3a, 2026-05-20]

8. **Token-based pricing limits clean tiered packaging**: existing model makes it harder to align differentiated assistant capabilities with price points. [Source: IQ-Strategy.md — IQ Pricing Kickoff, 2026-05-15]

9. **Three identified cost optimization levers**: (1) query sampling/filtering, (2) QPS-based pricing tiers, (3) decouple storage from real-time processing. [Source: IQ-COGS.md]

---

## Hidden Assumptions to Pressure-Test

- **Assumption:** QPS is the right unit for tier boundaries. Alternative: total monthly query volume (smooths spiky workloads), seat count (simpler to sell), or grid count. Needs validation with Sales.
- **Assumption:** Tiers should be based on COGS. Alternatively, tiers could be based on value (large enterprises get more queries AND better SLAs, analytics depth, etc.) — pricing by value delivered rather than cost.
- **Assumption:** NIOS and UDDI customers share the same tier model. Given asymmetric COGS, they may need separate pricing tracks entirely.
- **Assumption:** The pipeline cost dominates at all scales. At low QPS (ExxonMobil-class), LLM inference could become dominant as AI assistant usage grows — tiering must account for both cost vectors.

---

## Contradictions Found in KB

> ⚠️ **Pipeline cost attribution unresolved:** IQ-COGS.md flags an unresolved contradiction: original analysis treated the $0.22/million queries pipeline as IQ-incremental; Tom Hayward (IQ architect) says it is a UDDI feature that exists without IQ. Partial resolution: incremental for NIOS, not for UDDI. The pricing model must reflect this split — a single QPS-tier model applied uniformly would misprice one customer class or the other.

> ⚠️ **Free vs. paid IQ Assistant boundary is unresolved** (Himanshu vs. Mehul, May 15): this intersects with tier design. If the assistant is free, tier bands only gate Actions and pipeline depth — but the assistant still consumes LLM inference. Any QPS tier model must clarify whether assistant usage counts against the tier or is priced separately.

---

## Open Questions

1. **What is the right tier unit?** QPS, monthly query volume, grid count, or seat count — which is most legible to buyers and most aligned to COGS?
2. **How many tiers?** Good-better-best (3 tiers) vs. a continuous ramp (metered overage above a base allocation)?
3. **Where is the NIOS/UDDI seam?** Should NIOS IQ carry a pipeline surcharge (since cost is 100% incremental) or be priced at parity with UDDI once T&C is resolved?
4. **What is the service provider ceiling?** At what QPS does IQ become commercially unviable — and should that be a hard exclusion or a premium "carrier-grade" tier?
5. **How do tiers interact with existing token currency?** Does a higher tier grant more tokens, a lower token rate, or a flat fee + token topup model?
6. **Does tier affect feature access or just quantity?** Pure volume tiers (same features, different caps) vs. capability tiers (higher tiers unlock deeper analytics, longer retention, faster SLAs)?

---

## Success Metric

**Primary:** Gross margin on IQ DDI maintained at ≥ X% (TODO: confirm target margin with Finance) at GA, across a representative mix of EAP-class accounts, without excluding any current EAP customer.

**Guardrail metric:** IQ attach rate in new UDDI deals does not decline after tier model is introduced (pricing complexity must not become a sales friction point).

---

## Recommended Next Step

Proceed to `/synthesize-research` to pull all relevant KB material on commercialization, token strategy, and customer segmentation into a single sourced synthesis note before writing the PRD.
