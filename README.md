# pm-collab — the product team's AI-native workspace

**This repo is the heart of how we work.** It's the single source of truth for product
documentation *and* the standardized toolkit — templates, skills, agents, and
integrations — that every PM and engineer shares. Clone it once, and your AI assistant
(Claude Code in the terminal, or Claude in VS Code) behaves identically to everyone
else's: same templates, same voice, same connections to GitHub, Jira, and Confluence.

> **The idea:** stop scattering PRDs across Drive, Slack, and people's heads. Docs are
> authored here in markdown, reviewed via pull request, and auto-published to Confluence
> for everyone to read. Tickets are generated into Jira from those docs. One repo, one
> workflow, one set of standards.

---

## The operating model

```
   author in Claude             review            auto-publish        link out
  ┌────────────────────┐    ┌──────────────┐    ┌────────────┐    ┌──────────┐
  │ /write-spec → prds/ │ ─► │ Pull Request │ ─► │ Confluence │    │   Jira   │
  │ edit knowledge-base │    │ (eng review) │    │  (mirror)  │    │ (tickets)│
  └────────────────────┘    └──────────────┘    └────────────┘    └──────────┘
                                   │                                    ▲
                                   └── merge to main triggers sync ─────┘
                                       /ticket-from-spec → Jira via MCP
```

1. **Author** — `/write-spec <feature>` writes a structured PRD into `prds/`, using our
   shared context and voice automatically.
2. **Review** — open a PR. Engineering reviews the spec like code. The `doc-reviewer`
   agent pre-checks structure and clarity.
3. **Publish** — merge to `main`; the docs auto-sync to our Confluence space. No copy-paste.
4. **Plan** — `/ticket-from-spec prds/<feature>.md` drafts Jira tickets (with context +
   acceptance criteria) and creates them via the Atlassian integration — after you confirm.
5. **Build** — engineers implement in the relevant code repo; that PR updates the Jira ticket.

**Truth lives in Git. Confluence is a read mirror — never edit it directly.**

### Starting new work: the product sprint chain

Each step hands an artifact to the next — like a sprint, not a pile of commands:

```
/frame → /synthesize-research → /write-spec → doc-reviewer → /ticket-from-spec
brief  →  sourced KB note     →  PRD draft  →  critique    →  Jira issues
```

Run `/product-sprint "<idea>"` to drive frame → synthesize → spec → review in one flow
(it stops only for your judgment calls), or run any step on its own. The synthesis step
reads the `knowledge-base/` notes and writes its own sourced note back into it, so the
base compounds over time.

---

## What's standardized here (why this repo is the hub)

| Folder | What the team gets |
|--------|--------------------|
| `knowledge-base/` | Product context, glossary, and voice — loaded into every AI session |
| `prds/_TEMPLATE.md` | The PRD format everyone uses (RICE, Given/When/Then, metrics, non-goals) |
| `roadmap.md` | Now / Next / Later roadmap |
| `.claude/skills/` | Shared slash commands: `/product-sprint`, `/frame`, `/synthesize-research`, `/write-spec`, `/ticket-from-spec`, `/stakeholder-update`, `/roadmap-update` |
| `.claude/agents/` | Shared subagents: `doc-reviewer` (read-only PRD review) |
| `.claude/settings.json` | Permission guardrails (blocks secrets, requires confirm on risky actions) |
| `.mcp.json` | GitHub + Atlassian (Jira/Confluence) connections — teammates just authenticate |
| `CLAUDE.md` | Repo-level instructions the AI loads automatically here |

Because all of this is committed, **onboarding a teammate is: clone + authenticate.**
No per-person configuration.

---

## Getting started (each team member, ~10 min)

1. Install Claude Code (`claude --version`, need ≥ 2.1.59) **or** the Claude extension in VS Code.
2. Install GitHub CLI and sign in: `brew install gh && gh auth login`.
3. Clone and enter the repo:
   ```
   git clone https://github.com/nipja/pm-collab.git
   cd pm-collab
   ```
4. Start a session (`claude`) and run `/mcp` — complete the browser OAuth for **Atlassian**
   and **GitHub**. (The connections are pre-configured in `.mcp.json`; you only log in.)
5. Run `/memory` to confirm `CLAUDE.md` and the knowledge base are loading. You're ready.

Optional but recommended: copy `_global-CLAUDE.md.sample` to `~/.claude/CLAUDE.md` and
fill in your personal defaults — it makes the knowledge base available in *every* repo
you work in, not just this one.

---

## Conventions

- One PRD per file, `prds/<kebab-case-feature>.md`. Start from `_TEMPLATE.md`.
- Keep `CLAUDE.md` under ~200 lines; detailed standards go in `knowledge-base/`.
- Never commit secrets. Confluence/Jira tokens live in **GitHub Actions secrets** only.
- Don't push to `main` directly — open a PR. Use plan mode for multi-file changes.

---

## Admin setup (one-time, owner only)

- In `.github/workflows/sync-confluence.yml`, set your Confluence domain, space key, and
  parent page id, then add `CONFLUENCE_USER` and `CONFLUENCE_TOKEN` as repo Actions secrets.
- Test the Confluence sync against a throwaway page before pointing it at the real space.
