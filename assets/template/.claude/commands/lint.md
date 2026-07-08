---
description: Health-check the wiki and propose fixes
---

Run a wiki health check per CLAUDE.md. Checks, in priority order:

1. **Unsourced claims** — factual statements with no source link (invariant #1).
2. **Conflicts** — pages that disagree; unresolved `> ⚠️ Conflict:` callouts older than 2 lint passes.
3. **Stale claims** — statements superseded by newer sources (compare page `updated` dates against recent ingests in log.md).
4. **Orphans** — pages with no inbound links (`rg -L` the slugs against wiki/).
5. **Missing pages** — entities/concepts mentioned ≥3 times across pages but lacking their own page.
6. **Index drift** — pages on disk missing from index.md, or index entries pointing at nothing.
7. **Gaps** — questions the wiki raises but can't answer; suggest sources or web searches worth ingesting.

Procedure:

1. Read index.md + log.md, then scan pages (use `rg`/`ls`, not full reads of everything — sample where the wiki is large).
2. **Report findings as a table** (issue | page | proposed fix) before changing anything.
3. Apply fixes only after human approval. Batch them.
4. Append `## [YYYY-MM-DD] lint | <n> issues, <m> fixed` to log.md.
5. If a mistake pattern caused issues (e.g., ingests skipping index updates), append a line to "Recently burned" in CLAUDE.md.
