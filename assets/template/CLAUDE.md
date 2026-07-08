# Wiki Schema

> This repo is an LLM-maintained wiki. You (the agent) write and maintain `wiki/`.
> The human curates `raw/` and asks questions. Follow this schema exactly.

## Roles

- **Human**: adds sources to `raw/`, asks questions, reviews your edits. Never edits `wiki/` directly.
- **Agent**: owns everything under `wiki/`. Never modifies `raw/`.

## Layout

```
raw/               # immutable sources (never edit)
raw/assets/        # images downloaded from sources
wiki/index.md      # content catalog — update on EVERY ingest and filed answer
wiki/log.md        # append-only activity log
wiki/synthesis.md  # evolving thesis across all sources
wiki/sources/      # one summary page per raw source
wiki/entities/     # people, orgs, products, tools
wiki/concepts/     # ideas, methods, themes, open questions
.claude/commands/  # /ingest, /query, /lint
```

## Invariants

1. **Every claim links to a source.** Each factual claim in `wiki/` cites its origin: `([source](../sources/<slug>.md))` or a direct `raw/` file link. Unsourced claims are lint errors.
2. **`raw/` is read-only.** Never edit, rename, or delete anything in it.
3. **Ingest touch cap: index + new source page + ≤3 related pages.** Deeper propagation is `/lint`'s job. Reason: keeps ingest turns cheap and reviewable.
4. **Contradictions are recorded, not silently resolved.** When a new source conflicts with an existing page, add a `> ⚠️ Conflict:` callout on the page citing both sources. The human resolves.
5. **Log format is locked:** `## [YYYY-MM-DD] ingest|query|lint | Title`. Grep-parseable: `grep "^## \[" wiki/log.md | tail -5`.
6. **Wikilinks use relative markdown links**, not `[[wikilink]]` syntax (works in git hosting AND Obsidian).
7. **No orphan pages.** Every new page gets linked from index.md and at least one other page.

## Page conventions

Every `wiki/` page starts with YAML frontmatter:

```yaml
---
type: source | entity | concept | synthesis | answer
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [<raw filenames or source-page slugs>]
tags: []
---
```

- Filenames: kebab-case slugs (`dario-amodei.md`, `zone-2-training.md`).
- Source pages: summary ≤300 words, then "Key claims" bullets (each sourced), then "Links to" section.
- Keep pages small and linked rather than large and monolithic.

## Workflows

Detailed steps live in `.claude/commands/`. Summary:

- **`/ingest <raw-file>`** — read source, discuss takeaways, write source page, update index + ≤3 related pages, log it.
- **`/query <question>`** — read index.md first, drill into pages, answer with citations, offer to file the answer back as a page.
- **`/lint`** — health check: conflicts, orphans, stale claims, unsourced claims, missing pages, gaps. Propose fixes; apply after approval.

## Retrieval

- Phase 0 (now): index-first. Read `wiki/index.md`, then drill into linked pages.
- Fallback: `rg -il "<term>" wiki/` when the index misses.
- Do NOT build search infrastructure unless the human asks.

## Recently burned

> Append one line per mistake discovered here, newest first. Prune past 15 entries.

- (empty — new wiki)
