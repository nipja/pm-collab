---
name: resume
description: Resume an in-progress product sprint. Use when returning to a feature that was started in a previous session.
argument-hint: "[optional: feature slug or name]"
---
Resume the product sprint for: $ARGUMENTS

1. Look in `.sprint-state/` for a JSON file matching the feature name (or list all
   active sprints if no argument given, and ask which to resume).

2. Read the state file. It contains:
   - feature: the feature name/slug
   - stage: which step was last completed (frame | synthesize | spec | review | ticketing)
   - brief_path: path to the .brief.md file
   - prd_path: path to the PRD (if spec stage or later)
   - note_path: path to the synthesis note (if synthesize stage or later)
   - open_questions: list of unresolved questions from last session
   - last_updated: timestamp
   - next_action: a plain-English description of what needs to happen next

3. Read the referenced artifacts (brief, PRD, note) to fully restore context.

4. Summarize to the user:
   - What the feature is
   - What stage we're at
   - What was done last session
   - What the open questions are
   - What the recommended next action is

5. Ask: "Ready to continue from [stage]?" and proceed with the appropriate skill
   when confirmed.

When saving state (call this at the end of any sprint step):
- Write / update `.sprint-state/<kebab-slug>.json` with current stage, artifact
  paths, open questions, and a one-sentence next_action description.
- State file format:
  {
    "feature": "<name>",
    "stage": "<frame|synthesize|spec|review|ticketing|done>",
    "brief_path": "prds/<slug>.brief.md",
    "note_path": "notes/<slug>.md",
    "prd_path": "prds/<slug>.md",
    "open_questions": ["<question>", ...],
    "last_updated": "<ISO timestamp>",
    "next_action": "<one sentence describing what to do next>"
  }
