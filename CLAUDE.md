# CLAUDE.md — pm-collab

This repo is the product team's shared workspace and the source of truth for product
documentation. You help author and maintain PRDs, the knowledge base, and the roadmap,
and you generate Jira tickets and stakeholder updates. Truth lives in Git; Confluence is
an auto-published mirror — never instruct anyone to edit Confluence directly.

## Always load context
- Treat everything in `knowledge-base/` as authoritative product context.
- @knowledge-base/product-context.md
- @knowledge-base/glossary.md
- @knowledge-base/voice-and-style.md

## The product sprint chain
Starting new work follows a chain where each step's output feeds the next:
frame → synthesize-research → write-spec → doc-reviewer → (then ticket-from-spec).
Run `/product-sprint <idea>` to do the front half in one flow, or run the steps
individually. Each step reads the prior artifact (brief → synthesis note → PRD).

## Knowledge-base notes
Synthesized topic notes live in `knowledge-base/` and MUST follow
`knowledge-base/_NOTE_TEMPLATE.md`: YAML frontmatter (title, status, source_count,
summary, type, tags), inline `[Source: ...]` on every claim, a `⚠️ CONTRADICTION`
callout when notes disagree, and `[[wikilinks]]` to related notes. Never present an
unsourced claim as established fact.

## When writing or editing PRDs
- Start from `prds/_TEMPLATE.md`. One feature per file, named `prds/<kebab-case>.md`.
- Use glossary terms exactly. Match the voice in `voice-and-style.md`.
- Acceptance criteria use Given/When/Then. Prioritize with RICE. Always include success
  metrics and explicit non-goals.
- Don't invent product facts. If something isn't in the knowledge base, write
  `TODO: confirm with <role>` rather than guessing.

## Workflow rules
- Propose changes as edits to markdown files, then suggest opening a PR. Never push to
  `main` directly.
- Before any side-effecting action (creating Jira tickets, posting comments, publishing),
  show exactly what will happen and wait for explicit confirmation.
- Prefer plan mode for multi-file edits; summarize impact before applying.

## Never
- Never store secrets, tokens, or PII in any file in this repo.
- Never reproduce customer PII in PRDs or tickets.
