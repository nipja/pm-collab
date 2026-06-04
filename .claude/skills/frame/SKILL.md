---
name: frame
description: Pressure-test and sharpen a problem before any spec is written. Use when starting a new piece of work, or when an idea or request feels vague.
argument-hint: "[feature or problem idea]"
---
Run a framing session for: $ARGUMENTS

Goal: sharpen the framing BEFORE any PRD exists. Do not write a spec here.

1. Ground yourself before asking anything:
   - Always read `knowledge-base/product-context.md` and `knowledge-base/glossary.md`.
   - Then grep `research/` for files relevant to the topic (use the feature name and
     related terms as keywords). Read any matching files. The real context is in
     `research/` — don't skip this step.
   - Check `prds/active/` for any existing PRD or brief that overlaps with this topic.
2. Push back on the framing. Ask up to 6 forcing questions, one idea per question:
   - What is the actual user/business pain — with a concrete example, not a hypothetical?
   - Who is affected, and is this really one problem or several bundled together?
   - What is the narrowest valuable wedge versus the full vision?
   - What would make this NOT worth doing?
   - What does the knowledge base already establish or decide about this?
   - What is the success metric, and what is the guardrail metric?
3. Surface hidden assumptions, and call out any contradictions you find in the KB.
4. Produce a short PROBLEM BRIEF: sharpened problem statement, who's affected,
   scope (in / out), known facts (each with its source), open questions, success metric.
5. Save the brief to `prds/<kebab-slug>.brief.md` (a working artifact), then present it
   and ask whether to proceed to /synthesize-research.
