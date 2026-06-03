# pm-collab — the IQ product team's shared workspace

This repo is the single source of truth for Infoblox IQ product documentation. It is also the standardized toolkit — templates, skills, agents, and integrations — that every PM on the team shares.

The problem it solves: PRDs end up scattered across Google Drive, Slack threads, and people's heads. By the time engineering picks up a ticket, the original context is gone and the acceptance criteria are vague. This repo fixes that by making docs the artifact that everything else is generated from — not an afterthought.

---

## Core design decisions

### 1. Truth lives in Git, not the wiki

The **GitHub Wiki is a read-only mirror**. The markdown files in this repo are the canonical source. When something changes, you edit the markdown and open a PR — you never edit the wiki directly. A GitHub Actions workflow syncs docs automatically on every merge to `main`. This means:
- Version history is always in Git
- Reviews happen via pull request (engineering can comment on specs like code)
- Everyone always reads the same thing — no "which version is current?" confusion

### 2. Shared context loads into every AI session automatically

`CLAUDE.md` tells Claude to always load `knowledge-base/product-context.md`, `glossary.md`, and `voice-and-style.md` at the start of every session. This means:
- Every teammate's Claude has the same grounding in who the product is for, what stage we're at, and what terms we use
- You don't have to re-explain the product every time you start a new conversation
- The knowledge base compounds: each new synthesis note makes the next spec better

### 3. The knowledge base is a living asset, not a wiki

Notes in `knowledge-base/` follow a strict template (YAML frontmatter, inline `[Source: ...]` on every claim, `⚠️ CONTRADICTION` callouts, `[[wikilinks]]` between notes). The discipline matters: sourced claims prevent the AI from presenting guesses as facts, and contradictions stay visible rather than silently overriding each other.

### 4. Docs before tickets

Tickets are generated *from* PRDs, not written from scratch in Jira. This means acceptance criteria, context, and non-goals travel with the ticket from the start — engineers don't have to chase the PM to understand scope. The `/ticket-from-spec` skill handles the translation; it always shows you the full draft and waits for confirmation before creating anything in Jira.

### 5. Onboarding is: clone + authenticate

All skills, agents, settings, and MCP connections are committed. There is no per-person configuration. A new teammate clones the repo, authenticates with GitHub and Atlassian, and their Claude behaves identically to everyone else's.

---

## How collaboration works — the full flow

### Starting new work

Every new feature or initiative follows the same chain. Each step produces an artifact that feeds the next:

```
FRAME         SYNTHESIZE        SPEC             REVIEW           TICKETS
─────────     ──────────────    ─────────────    ─────────────    ─────────────
/frame        /synthesize-      /write-spec      doc-reviewer     /ticket-from-
"<idea>"  →   research      →   (reads brief  →  agent        →   spec
              (reads KB +       + KB note)       critiques        prds/<f>.md
brief.md       brief, writes                     structure,
saved to       new KB note)     PRD saved to     metrics,
prds/                           prds/            scope
```

**Run the whole chain at once:** `/product-sprint "<idea>"` drives frame → synthesize → spec → review in one flow, stopping only at the two judgment-call moments (after framing to get your answers, and after review to get your taste decisions).

**Run steps individually** when you already have a brief or want to re-run just the spec.

### What each step does

| Step | Command | What it produces | Where it's saved |
|------|---------|-----------------|-----------------|
| **Frame** | `/frame "<idea>"` | Problem brief: sharpened problem statement, forcing questions, known facts, open questions, success metric | `prds/<feature>.brief.md` |
| **Synthesize** | `/synthesize-research` | Sourced synthesis note pulling everything the KB knows about the topic, with contradiction callouts | `notes/<topic>.md` |
| **Write spec** | `/write-spec` | Full PRD following `prds/_TEMPLATE.md`: personas, RICE, user stories, acceptance criteria (Given/When/Then), success metrics, non-goals | `prds/<feature>.md` |
| **Review** | automatic after spec | Critique of structure, clarity, measurable metrics, and scope completeness | presented inline; you decide what to act on |
| **Tickets** | `/ticket-from-spec prds/<feature>.md` | One epic + scoped stories with ACs and context | draft shown first; created in Jira after your confirm |

### After the spec is approved

```
PM opens PR on GitHub
  → doc-reviewer runs as a check (optional but recommended)
  → teammates review and comment
  → merge to main
  → GitHub Actions syncs changed files to the GitHub Wiki automatically
  → /ticket-from-spec → Jira tickets created (with explicit confirmation)
  → engineering implements in the code repo; their PR references the Jira ticket
```

### Other day-to-day commands

| Command | When to use |
|---------|------------|
| `/roadmap-update` | After PRDs are approved or work ships — keeps `roadmap.md` current |
| `/stakeholder-update` | Drafts a TL;DR → RAG status → risks → asks update for leadership or partners |
| `/frame` alone | When an idea feels vague and you want to pressure-test it before writing a spec |

---

## What's in this repo

```
pm-collab/
├── knowledge-base/          # PERMANENT REFERENCE — auto-loaded into every AI session
│   ├── product-context.md   # What we build, who it's for, current stage, priorities
│   ├── glossary.md          # Canonical terms + what to avoid
│   ├── voice-and-style.md   # How our docs should read
│   └── _NOTE_TEMPLATE.md    # Template for new notes (used by research/ and notes/)
│
├── research/                # DEEP REFERENCE — large sourced notes; the RAG corpus
│   ├── IQ-Strategy.md       # Platform strategy (sourced, ~2000 lines)
│   ├── IQ-Architecture.md   # Technical architecture (sourced)
│   ├── IQ-COGS.md           # Unit economics and cost structure (sourced)
│   └── ...                  # Add new deep-dive notes here as topics are researched
│
├── notes/                   # SPRINT NOTES — synthesis artifacts generated per feature
│   └── IQ-Daily-Digest.md   # Example: generated during the daily digest sprint
│
├── prds/                    # OUTPUT — problem briefs and full specs, one file per feature
│   ├── _TEMPLATE.md         # Start every PRD here
│   ├── <feature>.brief.md   # Working brief (framing step output)
│   └── <feature>.md         # Full PRD (merged to main = published to Confluence)
│
├── roadmap.md               # Now / Next / Later — the team's deliberate source of intent
│
├── .claude/
│   ├── skills/              # Slash commands every teammate shares
│   │   ├── product-sprint/  # Full frame→synthesize→spec→review chain
│   │   ├── frame/           # Problem framing only
│   │   ├── synthesize-research/
│   │   ├── write-spec/
│   │   ├── ticket-from-spec/
│   │   ├── roadmap-update/
│   │   └── stakeholder-update/
│   ├── agents/
│   │   └── doc-reviewer.md  # Read-only PRD review agent
│   └── settings.json        # Permission guardrails (blocks secrets, confirm on side effects)
│
├── .mcp.json                # MCP connections: GitHub + Atlassian (Jira + Confluence)
├── CLAUDE.md                # Repo-level AI instructions — auto-loaded every session
└── .github/workflows/
    └── sync-confluence.yml  # Auto-publishes docs to Confluence on merge to main
```

---

## Getting started (each team member, ~10 min)

1. **Install Claude Code** — `claude --version` (need ≥ 2.1.59) or the Claude extension in VS Code.
2. **Install GitHub CLI** — `brew install gh && gh auth login`.
3. **Clone the repo:**
   ```
   git clone https://github.com/nipja/pm-collab.git
   cd pm-collab
   ```
4. **Authenticate integrations** — run `claude`, then `/mcp`. Complete the browser OAuth for **Atlassian** (Jira + Confluence) and **GitHub**. The connections are pre-wired in `.mcp.json`; you just log in.
5. **Verify** — run `/memory` and confirm that `CLAUDE.md` and the knowledge base notes are loading. You're ready.

**Optional but recommended:** copy `_global-CLAUDE.md.sample` to `~/.claude/CLAUDE.md` and fill in your personal role and preferences. This makes the product context available in every repo you work in, not just this one.

---

## Conventions

- **One PRD per file**, named `prds/<kebab-case-feature>.md`. Always start from `prds/_TEMPLATE.md`.
- **Never push to `main` directly.** Open a PR. Use plan mode (`/plan`) for multi-file changes so you can review the full impact before applying.
- **Never commit secrets.** No tokens or credentials in files — `GITHUB_TOKEN` is injected automatically by Actions.
- **Keep `CLAUDE.md` under ~200 lines.** Detailed standards and deep context belong in `knowledge-base/` and `research/`, not in the instructions file.
- **Never edit the wiki directly.** If something is wrong, fix the markdown here and merge — the wiki is overwritten on the next push.

---

## Admin setup (repo owner, one-time)

**Enable the wiki and bootstrap it:**
1. Go to your repo → **Settings → Features** → check **Wikis**.
2. Open the wiki tab and create one placeholder page manually (GitHub won't create the wiki git repo until a page exists).
3. That's it — no secrets, no tokens. `GITHUB_TOKEN` is automatically available to Actions with `contents: write` permission.

**First sync:**
Push any change to `main` touching `knowledge-base/`, `research/`, `notes/`, `prds/`, or `roadmap.md` and the workflow will populate the wiki automatically with a generated `Home.md` index.

**What the wiki URL looks like:**
`https://github.com/<org>/<repo>/wiki`

**Jira:**
Update `knowledge-base/product-context.md` with the correct Jira project key(s) once confirmed.
