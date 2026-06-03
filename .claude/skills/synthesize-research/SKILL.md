---
name: synthesize-research
description: Pull relevant knowledge-base notes (plus any provided inputs) into one sourced synthesis note. Use when gathering what's known about a topic before specing.
argument-hint: "[topic, or path to a .brief.md]"
---
Synthesize research for: $ARGUMENTS

1. If given a `*.brief.md`, read it first. Then search `knowledge-base/` and read
   every note relevant to the topic.
2. Write a synthesis note that follows `knowledge-base/_NOTE_TEMPLATE.md` EXACTLY:
   - YAML frontmatter: title, created, last_updated, source_count, status: draft,
     summary, type, tags.
   - A "What we know" section where every claim carries an inline `[Source: ...]`
     pointing to the specific KB note it came from.
   - A "What we don't know / open questions" section.
   - A `> ⚠️ CONTRADICTION` callout at the top for any place two notes disagree —
     name both sources and the current resolution. Never silently pick a side.
   - A "Related" section linking the source notes as `[[wikilinks]]`.
3. Never invent facts. If it isn't in a note, it goes under open questions, not "what we know".
4. Save to `knowledge-base/<kebab-slug>.md`. Present the summary and the open questions,
   then suggest proceeding to /write-spec.
