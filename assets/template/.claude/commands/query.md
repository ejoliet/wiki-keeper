---
description: Answer a question from the wiki, with citations
argument-hint: <question>
---

Question: $ARGUMENTS

Follow the schema in CLAUDE.md. Steps:

1. **Read `wiki/index.md` first.** Pick candidate pages from it. Fallback: `rg -il "<term>" wiki/`.
2. **Read the candidate pages** (and their linked pages if needed). Do NOT read `raw/` unless the wiki lacks the answer — the wiki is the compiled layer.
3. **Answer** with links to the wiki pages (and through them, sources) supporting each claim. Flag conflicts if pages disagree. Say "the wiki doesn't cover this" when true — never fabricate; optionally suggest a source to ingest or a web search.
4. **Offer to file it back.** If the answer synthesized across ≥2 pages or produced a new comparison/insight, ask: "File this as a wiki page?" If yes:
   - Write it to `wiki/concepts/<slug>.md` with `type: answer` frontmatter
   - Link it from index.md and from the pages it drew on
   - Append `## [YYYY-MM-DD] query | <question>` to log.md

Output formats: default markdown answer. Comparison → table. If the human asks for a chart or deck, produce it, then still offer to file the underlying analysis.
