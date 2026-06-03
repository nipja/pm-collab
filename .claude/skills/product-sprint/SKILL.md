---
name: product-sprint
description: Run the full product sprint chain end to end — frame, synthesize, spec, review — stopping only for judgment calls. Use to take an idea to a reviewed PRD in one flow.
disable-model-invocation: true
argument-hint: "[feature or problem idea]"
---
Run the product sprint for: $ARGUMENTS

Execute the chain below, passing each step's output into the next. Only stop for the
user at the marked decision points — handle everything else without prompting.

1. FRAME — run the `frame` skill. Produce the problem brief.
   → STOP. Show the brief and the forcing questions. Wait for the user's answers, then
     fold them into the brief.
2. SYNTHESIZE — run the `synthesize-research` skill against the brief + knowledge base.
   Write the synthesis note with sources and any contradiction callouts.
   → STOP only if a contradiction materially changes scope. Otherwise continue.
3. SPEC — run the `write-spec` skill. Write the PRD from the brief + synthesis note,
   following `prds/_TEMPLATE.md`.
4. REVIEW — invoke the `doc-reviewer` agent. Critique structure, clarity, measurable
   metrics, and scope.
   → STOP. Present the PRD plus the review findings for the user's taste decisions.

Do NOT create Jira tickets in this chain — that is a separate, explicit `/ticket-from-spec`
step the user runs after the PRD is approved. Never push to main; propose a PR at the end.
