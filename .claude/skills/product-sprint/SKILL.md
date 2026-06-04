---
name: product-sprint
description: Run the full product sprint chain end to end — frame, synthesize, spec, review — stopping only for judgment calls. Use to take an idea to a reviewed PRD in one flow.
disable-model-invocation: true
argument-hint: "[feature or problem idea]"
---
Run the product sprint for: $ARGUMENTS

Execute the chain below, passing each step's output into the next. Only stop for the
user at the marked decision points — handle everything else without prompting.

After EACH step completes, write/update `.sprint-state/<kebab-slug>.json` with the
current stage, artifact paths, open questions, and a one-sentence next_action. This
allows the sprint to be resumed in a future session via `/resume`.

1. FRAME — run the `frame` skill. Produce the problem brief.
   → Save state: stage=frame, brief_path set, open_questions = the forcing questions.
   → STOP. Show the brief and the forcing questions. Wait for the user's answers, then
     fold them into the brief. Update state with answers received.
2. SYNTHESIZE — run the `synthesize-research` skill against the brief + research/.
   Write the synthesis note with sources and any contradiction callouts.
   → Save state: stage=synthesize, note_path set.
   → STOP only if a contradiction materially changes scope. Otherwise continue.
3. SPEC — run the `write-spec` skill. Write the PRD from the brief + synthesis note,
   following `templates/_PRD_TEMPLATE.md`.
   → Save state: stage=spec, prd_path set.
4. REVIEW — invoke the `doc-reviewer` agent. Critique structure, clarity, measurable
   metrics, scope, cross-PRD conflicts, and unresolved blocker TODOs.
   → Save state: stage=review, open_questions = blocker findings from review.
   → STOP. Present the PRD plus the review findings for the user's taste decisions.

After user approves the PRD:
   → Save state: stage=ticketing, next_action="Run /ticket-from-spec prds/<slug>.md"

Do NOT create Jira tickets in this chain — that is a separate, explicit `/ticket-from-spec`
step the user runs after the PRD is approved. Never push to main; propose a PR at the end.
