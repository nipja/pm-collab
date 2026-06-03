# IBIQ Planning Process

How work flows in the IB-IQ Jira project (IBIQ) from idea → released story.

## At a glance

Three recurring meetings drive the planning process, with Jira as the system of record:

| Stage | Meeting | Owner | Cadence | Jira artifact |
| --- | --- | --- | --- | --- |
| Architecture & epic shaping | **IQ Roadmap meeting** | Tom Hayward (Architecture) | Bi-weekly, Mondays | Epics move `To Do` → `Reviewing` → `Backlog` |
| Readiness check for next sprint | **Product Prioritization meeting** | Product Management | Friday before each new sprint, 30 min | Epic ranking confirmed; high-priority epics needing breakdown/estimation are flagged for assignment |
| Sprint planning (grooming + scheduling) | **Sprint Grooming meeting** | Katie Evans (TPM) | Once per sprint (2 weeks) | Stories scheduled into the sprint; statuses validated |

```
┌──────────┐   ┌────────────┐   ┌──────────────┐   ┌──────────────┐   ┌────────────────┐   ┌───────────────┐   ┌────────────┐
│ Idea /   │ → │ Epic       │ → │ Epic         │ → │ Epic         │ → │ Stories broken │ → │ Story in      │ → │ Story      │
│ Feedback │   │ "To Do"    │   │ "Reviewing"  │   │ "Backlog"    │   │ down &         │   │ Sprint        │   │ shipped    │
│          │   │            │   │              │   │              │   │ estimated      │   │               │   │            │
└──────────┘   └────────────┘   └──────────────┘   └──────────────┘   └────────────────┘   └───────────────┘   └────────────┘
               IQ Roadmap mtg   Tom researches    Product             Engineering leads    Sprint Grooming    Engineering
               surfaces &       and defines       Prioritization      do breakdown +       schedules stories  delivers via
               clarifies        the architecture  confirms ranking;   estimation between   into the sprint    Jira workflow
                                                  flags epics         Prioritization &     and validates
                                                  needing breakdown   Sprint Grooming      them
```

## IQ Roadmap meeting (architecture + PM sync)

- **Owner**: Tom Hayward (Architect)
- **Cadence**: Bi-weekly Mondays, ~1 hour. Renamed from "IQ Process Workshop" on 2026-02-02 to make the focus explicit.
- **Audience**: Architecture, Product Management, TPM, Engineering Management. Recent attendees include Tom, Himanshu Raval (PM), Jon Abbe (PM), Katie Evans (TPM), James Zhu (EM), Tejasvi Shiv, Mehul Patel, Hasmit Grover, Chaitrali Dhole, Daniel Garcia.
- **Agenda doc**: `Engineering/Architecture/Process/IQ Process Workshop Agenda.docx` in SharePoint. Each occurrence is appended to the same running doc.
- **Tom's framing** (Oct 2025): *"an architecture-driven sync with PM with alternating short-term/long-term story grooming."*

### Preparation

- **Tom (Architect)**: Update the running agenda doc with topics for this session. Skim `To Do` and `Reviewing` epics and flag which are ready for architecture handoff. Pre-read any new epics PM has written since the last session.
- **PM (Himanshu, others)**: Bring customer/EAP feedback or new requirements that should affect epic priority. Pre-read architecture proposals on `Reviewing` epics so you can sign off in-meeting.
- **Feedback review owner (rotating)**: Pull together the last 7 days of feedback — negative items and any tickets created — for the standing summary.
- **Engineering leads**: Skim agenda epics you'd implement; come with dependency, sequencing, or scoping concerns.
- **TPM (Katie)**: Carry forward unresolved action items from the last session.

### Standard agenda

1. **Review agenda & objectives** — quick review of what's in scope today.
2. **Feedback review summary** — short report from the weekly Feedback Review meeting (negative feedback in last 7 days, tickets created).
3. **Roadmap review** — walk the open epic list (filter: `project = IBIQ AND type = Epic AND status in ("To Do", "Reviewing")`). For each epic: clarify intent, refine scope, identify dependencies and sequencing, agree on the data model (nouns/verbs). Once an epic has enough clarity that the architect can take it for design work, it moves from `To Do` to `Reviewing`.
4. **Architecture for upcoming stories** — Tom presents architecture for epics in `Reviewing` that are ready to be handed off; engineering leads challenge / extend. After the architecture is defined, the epic moves to `Backlog`, signaling that it's ready for story breakdown and ranking.
5. **Future epics** — surface ideas that don't yet have epics so they can be written.
6. **Agenda planning for next meeting** (last 5–10 min) — owners and prep work for the next session.

### What this meeting produces

- Epics moved from `To Do` → `Reviewing` (Tom takes them for architecture research) and from `Reviewing` → `Backlog` (architecture defined; ready for story breakdown).
- A shared understanding of architecture for upcoming work.
- Action items captured directly in the agenda doc.

## Product prioritization meeting

A short readiness check before the next sprint. PM confirms the priority order, and engineering identifies any high-priority epics that aren't yet sprint-eligible so they can be assigned for breakdown and estimation **before** Sprint Grooming. Story-to-sprint scheduling does **not** happen here — it happens at Sprint Grooming.

- **Owner**: Product Management
- **Cadence**: The Friday before each new sprint, 30 min. Most recent example: 11:30 AM – 12:00 PM PT, Friday March 27, 2026.
- **Audience**: PM (Himanshu Raval), Engineering Manager (James Zhu), Architecture (Tom), TPM (Katie).

### Preparation

- **Himanshu (PM)**: Confirm the `Backlog` epic ranking is current before the meeting. Identify which epics are high-priority for the next sprint. Pre-flag which of those are still missing story breakdown or estimation.
- **Katie (TPM)**: Bring a current-sprint workload report — points remaining, in-progress vs. blocked, expected completion — so the team can size next-sprint capacity.
- **James (Engineering Manager)**: Know which engineering leads have capacity for breakdown work; be ready to make assignments live.
- **Tom (Architect)**: Be ready to clarify architecture or scope on any epic that resurfaces.

### Agenda

1. **Review remaining workload from the current sprint** to assess bandwidth and story-point capacity for the next sprint.
2. **PM confirms epics have the latest prioritization.** The ranked list of `Backlog` epics is the authoritative input to the next Sprint Grooming meeting.
3. **Identify high-priority epics that need story breakdown and estimation.** Any high-priority epic that is not yet broken down + estimated is flagged. James Zhu (Engineering Manager) assigns those epics to engineering leads so the breakdown and estimation are completed **before** the Sprint Grooming meeting; otherwise the epic won't be eligible for the next sprint.
4. **Rank Epics.** Reorder `Backlog` epics if priorities have shifted (Himanshu owns the ranking).

### What this meeting produces

- A confirmed ranked list of `Backlog` epics.
- A list of high-priority epics still needing story breakdown and/or estimation, each assigned to an engineering lead by James Zhu, with the expectation that they're ready by Sprint Grooming.
- A capacity read for the next sprint based on remaining current-sprint workload.

## Sprint grooming meeting

- **Owner**: Katie Evans (TPM). Scheduled as a team-owned meeting (not personally owned) so the cadence and time can be adjusted by anyone.
- **Cadence**: Once per 2-week sprint. Sprint numbering is sequential (e.g., Sprint 8 = 4/28–5/11). Moving from morning to afternoon to fit attendees' schedules.
- **Audience**: TPM, engineering leads (Mark Phillips, Turner Anderson, Vijay Collooru), PM (Himanshu Raval, Jon Abbe), architecture as needed.
- **Tooling**: Jira sprint board; Rovo is used to summarize story-point distribution across assignees ("how many story points are assigned by person").

### Where this fits

This is the team's sprint planning meeting — sprint grooming and sprint scheduling are **combined here**. Inputs (ranked `Backlog` epics, broken-down + estimated stories, capacity read) come from the Product Prioritization meeting earlier in the week. Outputs are a fully scheduled sprint and a groomed sprint board.

### Preparation

- **Katie (TPM, owner)**: Have the sprint board open with eligible candidate stories filtered (stories under in-progress epics + stories under top-of-rank `Backlog` epics that have been broken down and estimated). Use Rovo to summarize per-assignee story-point distribution. Bring forward the capacity number from Product Prioritization.
- **Engineering leads (Mark, Turner, Vijay)**: Story breakdown and estimation must be complete for any epic you were assigned at the prior Product Prioritization meeting. Be ready to discuss dependencies, ownership, and scope of each candidate story.
- **Engineers / assignees**: For each story you currently own, make sure status, estimate, and known blockers are accurate in Jira before the meeting starts so the TPM walk doesn't stall.
- **Himanshu (PM)**: Be available to confirm priority calls between competing in-progress epics.
- **Tom (Architect)**: Be available for architecture questions on stories that surface new design points.

### Agenda

The TPM goes through each candidate / in-sprint story and confirms:

1. **Status** — is it actually `To Do`, `In Progress`, `Blocked`, etc.? Update if stale.
2. **Dependencies** — anything blocking it that the team needs to track?
3. **Story points** — does the estimate still make sense? Are points clustered on one person?
4. **Assignee & workload** — is the right person on it? Is anyone over-burdened?
5. **Priority** — does the priority need to change relative to other in-flight work?

For bugs, the rule is: **only pull in P3 bugs you actually expect to fix this sprint** — don't auto-roll P3s forward sprint over sprint.

### How stories get scheduled into the sprint

The goal is to fill the sprint with stories whose points sum to the team's velocity — no more, no less. To minimize WIP and finish what we've started, **stories from epics already `In Progress` are pulled first**; only after those are exhausted do we pull from the highest-ranked `Backlog` epics.

Only stories under epics that have been fully broken down **and** estimated are eligible. Epics flagged at the Product Prioritization meeting and not yet ready by this meeting are excluded.

```
   ┌────────────────────────────────────────┐
   │ Start                                  │
   │   remaining = team velocity            │
   │   queue = candidate stories, ordered:  │
   │     1. From in-progress epics          │
   │     2. From Backlog epics, in          │
   │        Himanshu's ranked order         │
   │   (only stories under broken-down,     │
   │    estimated epics are eligible)       │
   └────────────────────────────────────────┘
                       │
                       ▼
              ┌──────────────────┐
   ┌─────────▶│ Queue empty?     │──yes──▶ Sprint planned
   │          └──────────────────┘
   │                  │ no
   │                  ▼
   │          ┌──────────────────────┐
   │          │ Pop next story       │
   │          └──────────────────────┘
   │                  │
   │                  ▼
   │          ┌────────────────────────┐
   │          │ points ≤ remaining ?   │──no──┐
   │          └────────────────────────┘      │ skip
   │                  │ yes                   │
   │                  ▼                       │
   │          ┌────────────────────────┐      │
   │          │ Add to sprint          │      │
   │          │ remaining -= points    │      │
   │          └────────────────────────┘      │
   │                  │                       │
   └──────────────────┴───────────────────────┘
```

If `remaining` reaches zero (or is too small for any remaining candidate), the sprint is planned. If the queue empties with capacity left, the sprint is **under-committed** — flag it so PM/architect can promote a `Backlog` epic earlier or write more stories under an existing epic.

## How epics get prioritized

- An epic in **`Reviewing`** is being researched and architected by Tom — story breakdown is **not** yet appropriate.
- An epic moving to **`Backlog`** signals "architecture is defined; ready for story breakdown."
- **Himanshu Raval (PM) ranks epics in `Backlog`.** The ranking is confirmed at the Product Prioritization meeting.
- An epic is only eligible for the next sprint once its stories are broken down and estimated. Any high-priority `Backlog` epic that isn't yet broken down + estimated is flagged at the Product Prioritization meeting; **James Zhu (Engineering Manager) then assigns those epics to engineering leads** so the work is ready before Sprint Grooming.
- Epics that don't make it through breakdown + estimation in time are excluded from the next sprint.

## Non-functional epics and platform enablers

Not every epic comes from PM. **Architecture (Tom) and engineers write epics and stories for self-improvement, tech debt, and platform enablers** — work that doesn't appear on the customer roadmap but unlocks future product work, reduces operational risk, or makes the team faster.

These epics live in the same `Backlog` and follow the same state machine as product epics. They don't go through PM ranking (Himanshu ranks product epics; non-functional epics carry their own engineering-driven priority).

At the Product Prioritization meeting, **James Zhu balances product epics against non-functional / platform-enabler epics** when reading capacity for the next sprint, so a portion of every sprint's velocity is reserved for self-improvement instead of being fully consumed by product work.

## IBIQ Jira workflow

### Issue types
Epic, Story, Bug, Sub-task, Test, Initiative.

### Epic states

| State | Meaning |
| --- | --- |
| `To Do` | New epic. Surfaced and discussed at the IQ Roadmap meeting. |
| `Reviewing` | Tom is researching and defining the architecture for the epic. Not yet ready for story breakdown. |
| `Backlog` | Architecture is defined. Himanshu ranks epics here; the ranked list drives story breakdown. |
| `In Progress` | Stories under the epic are being delivered. |
| `Done` | Released. |
| `Aborted` | Closed without delivering (e.g., duplicate, no longer needed). |

Typical happy-path transitions: `To Do → Reviewing → Backlog → In Progress → Done`.

> Note (per April 2026 Roadmap meeting): Danny has Jira admin access and is fixing IBIQ state transitions — some transitions may not yet match this intended model.

### Story states

Confirmed via Jira transitions on IBIQ-345:

| State | Category | Meaning |
| --- | --- | --- |
| `To Do` | To Do | Created; not started. |
| `Blocked` | To Do | Cannot proceed; surfaced at sprint grooming. |
| `In Progress` | In Progress | Engineer actively working. |
| `In Review` | In Progress | Code review / PR open. |
| `Awaiting Verification` | In Progress | QA / verification by reporter. |
| `Awaiting Deployment` | In Progress | Verified; waiting for release. |
| `Resolved` | Done | Released. Terminal happy path. |
| `Aborted` | Done | Closed without delivering. |

Typical happy-path transitions: `To Do → In Progress → In Review → Awaiting Verification → Awaiting Deployment → Resolved`. `Blocked` is used as a side-state and surfaced at the next sprint grooming.

## End-to-end story lifecycle

1. **Idea or feedback** lands (customer feedback, EAP, internal ask, architecture-driven need).
2. **Epic written** in IBIQ in `To Do` state.
3. **Epic discussed at IQ Roadmap meeting** until intent, scope, and dependencies are clear enough for architecture work; epic moves to `Reviewing`.
4. **Tom researches and defines the architecture** while the epic is in `Reviewing`. When the architecture is ready, the epic moves to `Backlog`.
5. **Himanshu ranks** the `Backlog` epics; ranking is confirmed at the next Product Prioritization meeting.
6. **Product Prioritization meeting (Friday before next sprint)** flags any high-priority `Backlog` epic that isn't yet broken down + estimated; James Zhu assigns those epics to engineering leads.
7. **Engineering leads break down stories and estimate** between Product Prioritization and Sprint Grooming.
8. **Sprint Grooming meeting** schedules eligible stories into the sprint up to the team's velocity (in-progress epics first, then ranked `Backlog` epics) and validates status, dependencies, points, assignees.
9. **Engineering** moves each story through `In Progress → In Review → Awaiting Verification → Awaiting Deployment → Resolved`. The epic moves to `In Progress` once delivery starts and to `Done` once the work is released.

## Roles

| Role | Person(s) | Responsibilities |
| --- | --- | --- |
| Architect | Tom Hayward | Owns IQ Roadmap meeting; researches and defines architecture for epics in `Reviewing`; aligns with PM on roadmap; **writes non-functional / platform-enabler epics for self-improvement and tech debt** |
| Product Manager | Himanshu Raval (PM), Jon Abbe (PM) | Ranks product epics in `Backlog`; owns / drives the Product Prioritization meeting; confirms ranking and capacity for the next sprint |
| Engineering Manager | James Zhu | At the Product Prioritization meeting, assigns flagged `Backlog` epics to engineering leads for story breakdown and estimation so they're ready by Sprint Grooming; **balances product epics against non-functional / platform-enabler epics so each sprint reserves capacity for self-improvement** |
| TPM | Katie Evans | Coordinates process and meeting cadence; owns Sprint Grooming and the sprint board |
| Engineering leads | Mark Phillips, Turner Anderson, Vijay Collooru | Break down + estimate stories under assigned epics; **may write non-functional / platform-enabler epics and stories**; provide technical context at Sprint Grooming; commit to stories in sprint |
