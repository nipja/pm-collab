---
name: roadmap-update
description: Update roadmap.md from recent PRDs, shipped work, and Jira status. Use when asked to refresh or reorganize the roadmap.
argument-hint: "[optional focus area]"
---
Update `roadmap.md`. Focus: $ARGUMENTS (or the whole roadmap if blank).

Steps:
1. Read `roadmap.md`, the PRDs in `prds/`, and recent Jira/commit activity if connected.
2. Move items between Now / Next / Later based on actual status. Move completed work into
   "Recently shipped" with the ship date.
3. Keep each item to outcome + why; link its PRD where one exists.
4. Don't invent commitments. Flag anything ambiguous as `TODO: confirm with <role>`.
5. Propose the change as an edit to `roadmap.md`; suggest opening a PR. Don't push to main.
