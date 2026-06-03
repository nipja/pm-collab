---

## title: "IQ Chat Rendering Architecture"

created: 2026-05-09
last_updated: 2026-05-09
source_count: 2
status: draft
summary: "Architecture for making IQ chat responses visually rich and dashboard-like via a component registry (A2UI), widget service, and agent tool integration — targeting June–July 2026 prototype."
type: platform
tags: [iq, ux, platform, component-registry]

IQ chat currently produces text-only responses in a narrow right-hand pane, constrained by PDS Core styling. The desired end state is a flexible canvas mixing concise prose with visual widgets — metric cards, badges/status pills, call-outs, richer tables, and inline charts — governed by a component registry and delivered via structured agent tool calls. The architecture mirrors how Claude's chat delivers richness, but anchored to Infoblox's design system rather than arbitrary HTML or sandboxed iframes.

Target delivery window: June–July 2026 for a working prototype with the first widget types and foundational infrastructure in place.

---

## Why This Matters

Current IQ behavior is text-only, with limited deterministic charting when tabular data is available. The chat UI is constrained by PDS Core styling, a narrow right-hand pane, and a small set of IQ-specific components. Responses feel cramped and "wall of text" despite containing useful information.

The competitive reference point is Claude's chat, which uses an Anthropic design system with better fonts, white space, colors, richer tables, and inline widgets (metric cards, badges, status pills, call-outs, code blocks, charts) presenting information like a well-designed report. The full-screen landing page (arriving M1/May 2026) creates additional surface area to exploit richer layouts.

[Source: 2026-05-08_IQ_Chat_Rich_Response_Rendering_Response_Data.md, 2026-05-08_Turner_Himanshu_Jon_weekly_IQ_Sync.md]

---

## Architecture Components

### Component Registry (A2UI)

A2UI acts as the central catalog of renderable components and their props. It is the contract layer — not a rendering engine and not a visual constraint.

- Initially includes PDS Core components plus IQ-specific components; extensible with chat-focused widgets (metric cards, call-outs, status pills, richer tables).
- Defines schemas/contracts the agent tools and widget service rely on.
- Before the agent can request a widget, it must be registered in A2UI — ensures type safety and prevents unsupported UI.
- Part of next steps: fully register all components from PDS Core and IQ-specific libraries.

### Widget / Metric Service

Sits on top of A2UI to:

- Take structured requests from the agent.
- Validate against the registry's schemas.
- Bind data from IQ/MCP workflows into widgets.
- Manage widget lifecycle: creation, rendering, updates as conversations evolve, teardown when context changes.
- Provide a stable API for IQ agents and future platform consumers.
- Enforce security boundaries — no arbitrary HTML execution; limited or no direct JS from the model.

### Agent Tool Integration

New tools that allow the agent to request widgets inline during response generation:


| Tool                   | Purpose                                                     |
| ---------------------- | ----------------------------------------------------------- |
| `show-widget`          | Render a widget from the registry inline in chat            |
| `generate-metric-card` | Create a headline metric card with value, label, and status |
| `show-callout`         | Render a risk, recommendation, or alert callout block       |
| `visualize-table`      | Render a formatted table with improved styling              |


**Streaming model:** Agent can send a short textual summary, invoke widget tools mid-response, then continue explanation. Supports partial streaming.

**Future:** Slash/MCP-style commands for demos (e.g., `/show-off-formatting`) and regression testing.

---

## UX Design Principles

**Response structure (preferred):**

1. Short opening — one sentence or a few bullets with key outcome, questions, decisions, or next steps.
2. Visual widgets where justified — metric cards for headline numbers, badges for quick state, call-outs for risks/recommendations, charts/tables for data-dense content.
3. Explanatory text referencing (not duplicating) the visuals.
4. Wall-of-text only when explicitly requested.

**Layout considerations:**

- Right-hand chat pane: compact widgets, succinct text (limited width forces wrapping).
- Full-screen landing page: richer multi-column or stacked layouts, more simultaneous widgets.

**Styling strategy:**

- Option A: Upgrade PDS Core globally — modernize fonts, spacing, colors, tables, charts for all surfaces.
- Option B: Create chat-specific library/theme — tailor for conversational content (higher maintenance cost, risk of divergence).
- Both options: avoid PDS acting as "restrictor plates"; ensure tokens and components support richer widgets.

**Crawl-walk-run:**

- Crawl: basic font/spacing tweaks, simple metric card and call-out widgets, slightly richer tables.
- Walk: broader widget set (status pills, improved charts, better code blocks), more refined prompts.
- Run: complex dashboard-like assemblies of multiple widgets in a single answer.

---

## Current Limitations

- Agent can only output text; no general safe mechanism for arbitrary inline visuals.
- Visuals limited to simple deterministic charts generated from tabular data.
- No tool today for the agent to dynamically create a mix of widgets within a single response.
- PDS Core constraints perceived as limiting chat richness; requires cultural and architectural changes to bypass where justified.

---

## Sandbox/Iframe Trade-off

The Claude-style sandboxed iframe approach (arbitrary JS under policy constraints) was considered but is not the preferred initial path:


| Approach                     | Pros                                                  | Cons                                                                                        |
| ---------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Sandbox iframe (Claude-like) | Maximum flexibility; very beautiful and dynamic       | Higher complexity, increased security surface, performance/cost concerns, harder governance |
| A2UI + PDS-driven            | Consistent with platform, reusable, easier governance | Initially limited to registered components                                                  |
| Raw HTML from LLM            | Fastest to prototype                                  | Unsafe, brittle, expensive at scale — **not recommended**                                   |


Decision: prefer A2UI + structured widgets; sandbox approach may be considered later under strict governance.

---

## Proposed Delivery Roadmap


| Epic                      | Purpose                                                                          | Target      |
| ------------------------- | -------------------------------------------------------------------------------- | ----------- |
| Component Registry (A2UI) | Populate with PDS Core and IQ components; define schemas                         | Before June |
| Widget Service            | Secure instantiation, data binding, lifecycle management                         | June        |
| Agent Tooling             | show-widget, generate-metric-card, show-callout, visualize-table                 | June–July   |
| Dev Playground            | Sandbox for prompt/policy/widget experiments; compare layouts and themes         | June–July   |
| PDS Audit/Enhancement     | Decide what goes into PDS 3.0 vs. chat-only; remove restrictor-plate constraints | TBD         |


---

## PDS and Design System Context

PDS (Product Design System) currently constrains desired chat presentation. Teams default to it even when it clearly blocks better layouts or richer formatting. The May 2026 direction from product and engineering leadership:

- PDS is bypassable when it blocks better UX.
- Challenges to PDS are explicitly encouraged — not politically off-limits.
- IQ chat-specific overrides or a dedicated chat component library are acceptable if justified.
- Long-term: feed limitations back to PDS owners and target PDS 3.0 improvements.

Angular stack lag (currently ~15, others moving to 21) prevents using newer AUI components and is the root of perceived visual staleness.

[Source: 2026-05-08_Turner_Himanshu_Jon_weekly_IQ_Sync.md]

---

## Open Questions

- Who will own the A2UI component registry and widget service long term?
- How will IQ-specific components be governed relative to PDS Core?
- What is the minimum acceptable widget set for the June–July prototype to feel like a noticeable improvement?
- Who is concretely committed to June–July work across backend, design, platform, and agent teams?
- What execution and rendering safeguards will be standardized on (strict A2UI + PDS, limited sandboxing, server-side rendering, or hybrids)?

---

**See also:** [[UX-Design-Direction]] · [[IQ-Strategy]] · [[Platform-Architecture]]