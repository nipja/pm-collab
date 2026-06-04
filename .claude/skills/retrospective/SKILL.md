---
name: retrospective
description: After a feature ships, record what changed vs. the spec and why. Closes the feedback loop and builds institutional memory. Use when a feature is marked shipped or a sprint is closing.
argument-hint: "[path to PRD, e.g. prds/active/iq-daily-digest.md]"
---
Run a retrospective for: $ARGUMENTS

1. Read the PRD at the given path. Extract: original problem statement, success
   metrics, non-goals, key decisions, and acceptance criteria.

2. Ask the user 4 questions (wait for answers before writing anything):
   a. **What shipped as specced?** Which requirements landed exactly as written?
   b. **What changed, and why?** Which requirements were cut, modified, or added
      mid-build? What was the root cause (bad assumption, platform constraint,
      customer feedback, timeline)?
   c. **Did the metrics move?** What did the data show vs. the targets in Section 2?
      If too early to measure, what leading signals exist?
   d. **What would you frame differently?** If you were writing the PRD today with
      hindsight, what would the problem statement, scope, or success metric look like?

3. Write a retro note to `notes/<slug>-retro.md` following `templates/_NOTE_TEMPLATE.md`:
   - YAML frontmatter with type: retrospective, status: reviewed
   - Sections: What shipped, What changed (with root causes), Metric outcomes,
     Reframe (what the better problem statement was in hindsight)
   - Every claim sourced to either the PRD, the user's answers, or Jira
   - A "Lessons" section with 2-3 transferable takeaways for future sprints

4. Update the PRD's YAML frontmatter: set status: shipped, last_updated: today.
   Move the PRD file from prds/active/ to prds/archive/.

5. Check if the retro note surfaces any facts that should update `research/` files
   (e.g., a validated architectural decision, a corrected assumption about user
   behaviour). If so, flag them explicitly — do not auto-update research/ files;
   propose the edits and let the PM decide.

6. Clear the sprint state file at `.sprint-state/<slug>.json` (set stage: done).

7. Suggest opening a PR for the retro note + PRD status update.
