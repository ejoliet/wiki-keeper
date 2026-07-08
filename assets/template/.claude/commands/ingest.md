---
description: Ingest a raw source into the wiki
argument-hint: <path to file in raw/>
---

Ingest source: $ARGUMENTS

Follow the schema in CLAUDE.md. Steps:

1. **Read** the source fully. If it references images in `raw/assets/`, view the important ones.
2. **Discuss** — before writing anything, give the human 3-5 key takeaways and ask what to emphasize. Wait for their reply.
3. **Write source page** at `wiki/sources/<slug>.md`:
   - Frontmatter (`type: source`, dates, `sources: [<raw filename>]`)
   - Summary ≤300 words
   - "Key claims" bullets — each linking back to the raw file
   - "Links to" — related wiki pages
4. **Update related pages** — at most 3 (invariant #3). Prioritize: entity/concept pages this source directly informs. Where the source contradicts an existing claim, add a `> ⚠️ Conflict:` callout citing both sources — do not silently overwrite.
5. **Update `wiki/index.md`** — add the source page under Sources; add any new entity/concept pages.
6. **Append to `wiki/log.md`**: `## [YYYY-MM-DD] ingest | <Source title>` plus 1-2 lines listing pages touched.
7. **Report** — one-line summary per page created/updated, plus anything deferred to `/lint`.

Rules:
- AIDEV-NOTE: never edit `raw/`.
- If the source is huge (book chapter, long transcript), summarize section-by-section before writing the page.
- If a needed entity/concept page doesn't exist, create it minimal (frontmatter + 2-3 sourced bullets) — don't pad.
