---
name: write-spec
description: Turn a problem statement or feature idea into a structured PRD in prds/. Use when asked to write a spec, PRD, or requirements doc.
argument-hint: "[feature name or idea]"
---
Create a new PRD for: $ARGUMENTS

Steps:
1. Read `prds/_TEMPLATE.md` and follow its structure exactly.
2. If a `prds/<slug>.brief.md` or a synthesis note in `knowledge-base/<slug>.md` exists
   for this topic, read them first and build on them — carry their open questions and
   sourced facts into the PRD rather than starting from scratch.
3. Pull product context, terminology, and voice from `knowledge-base/`. Use glossary
   terms precisely and match `voice-and-style.md`.
3. Write the PRD to `prds/<kebab-case-feature>.md`. Fill every section. Where a fact
   isn't established, insert `TODO: confirm with <role>` rather than guessing.
4. Prioritize requirements with RICE and write acceptance criteria as Given/When/Then.
5. Do NOT push to main. Summarize what you created and suggest opening a PR.
