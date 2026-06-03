# Glossary

> The canonical terms. Claude uses these exactly and avoids the "avoid" column.
> Consistent language is half of good docs — keep this current.

| Term | Definition | Use | Avoid |
|------|------------|-----|-------|
| Infoblox IQ | The umbrella AI product brand covering DDI, Threat Defense, and Asset Insights AI layers | "Infoblox IQ" (external); "IQ" (internal shorthand) | "IQ Assistant", "AI product", "BloxOne AI" |
| IQ for DDI | The IQ add-on for DNS, DHCP, and IPAM operations | "IQ for DDI" | "IQ DDI", "DDI IQ", "DDI Actions" |
| IQ for Threat Defense | The IQ add-on for security/SOC workflows; includes repositioned SOC Insights | "IQ for Threat Defense" | "IQ Security", "IQ-TD", "SOC IQ" |
| IQ for Asset Insights | The IQ add-on for asset lifecycle intelligence | "IQ for Asset Insights" | "IQ Assets", "Asset IQ" |
| AI assistant | The conversational chat interface within IQ (in-portal) | "AI assistant" (in nav/UI) | "IQ Assistant" (sounds like a purchasable sub-product) |
| AI actions | The hub page listing all IQ action cards (DDI + Threat Defense) | "AI actions" (in nav/UI) | "IQ Actions", "action center" |
| IQ Action | A proactive anomaly card surfaced by the Insighter pipeline; includes root-cause analysis and a proposed remediation | "action", "IQ action" | "alert", "incident", "ticket" |
| Insighter | The internal telemetry pipeline and research agent that powers anomaly detection and action generation | "Insighter" (internal/engineering) | Do not use externally |
| MCP Server | The customer-facing API surface that lets external agents (Claude, Cursor, etc.) call Infoblox data via the MCP protocol | "Infoblox MCP Server" | "IQ MCP", "AI API" |
| Account | A billing entity that owns one or more Infoblox portal tenants | "account" | "org", "tenant", "customer" |
| EAP | Early Access Program — controlled pre-GA rollout to a named set of customers | "EAP", "Early Access" | "beta", "preview", "pilot" |
| UDDI | Universal DDI — Infoblox's SaaS-delivered DNS/DHCP/IPAM product | "UDDI" | "B1 DDI", "BloxOne DDI" |
| NIOS | Infoblox's on-premises DNS/DHCP/IPAM appliance product | "NIOS" | "on-prem DDI", "classic" |
| QPS | Queries per second — the primary DNS load metric and IQ cost driver | "QPS" | "query volume", "DNS traffic" |
| IQL | Infoblox Query Language — the structured query syntax IQ generates from natural language | "IQL" | "SQL", "query language" |
| Approval workflow | The human-in-the-loop gate where an operator approves or denies an AI-proposed action before it executes | "approval workflow", "approve/deny" | "human review", "confirmation step" |
| RICE | Prioritization framework: Reach × Impact × Confidence ÷ Effort | "RICE score" | "priority score", "stack rank" |

## Acronyms
- **PRD** — Product Requirements Document (lives in `prds/`).
- **DDI** — DNS, DHCP, and IPAM (the three core network services Infoblox manages).
- **EAP** — Early Access Program.
- **GA** — General Availability.
- **IPAM** — IP Address Management.
- **MCP** — Model Context Protocol (Anthropic open standard for AI tool integrations).
- **QPS** — Queries Per Second.
- **RICE** — Reach, Impact, Confidence, Effort (prioritization framework).
- **SOC** — Security Operations Center.
- **SRS** — Summer Revenue Summit (Infoblox internal sales/GTM milestone event).
- **UDDI** — Universal DDI (Infoblox SaaS DDI product).
- **IQL** — Infoblox Query Language.
