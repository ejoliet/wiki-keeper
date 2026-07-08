---
name: wiki-keeper
description: >
  Build and maintain LLM-maintained personal knowledge wikis (the "LLM Wiki"
  pattern): a persistent, interlinked markdown wiki that the agent writes and
  the human curates, compounding knowledge across sources instead of re-deriving
  it per query (anti-RAG).

  Trigger whenever Emmanuel says: "new wiki", "llm wiki", "knowledge base for X",
  "start a wiki on X", "ingest this into my wiki", "add this source", "query the
  wiki", "lint the wiki", "wiki health check", or asks to track/accumulate
  knowledge over time on any topic (research deep-dive, book companion, health
  protocols, competitive analysis, project memory). Also trigger when a repo
  contains a CLAUDE.md declaring "This repo is an LLM-maintained wiki".
  Read this BEFORE creating or touching any wiki repo.
---

# Wiki Keeper

Operate the LLM Wiki pattern: human curates sources and asks questions; agent
builds and maintains a persistent markdown wiki. Knowledge compounds — it is
compiled once at ingest and kept current, never re-derived per query.

## Two modes

| Mode | When | What to do |
|------|------|-----------|
| **Init** | No wiki exists yet | Scaffold from `assets/template/` (below) |
| **Operate** | Repo has a wiki CLAUDE.md | Follow THAT repo's CLAUDE.md; it is authoritative. This skill is only fallback context. |

## Init workflow

1. Ask one batched question if unknown: wiki topic + directory name. Infer everything else.
2. Copy `assets/template/` into the target directory (include dotfiles: `.claude/`, `.gitignore`).
3. Replace every `__INIT_DATE__` placeholder with today's date (`grep -rl __INIT_DATE__ . | xargs sed -i "s/__INIT_DATE__/$(date +%F)/"`).
4. Customize `CLAUDE.md` for the domain: add a 2-3 line "Domain" section at top (topic, what counts as an entity/concept here, emphasis).
5. `git init && git add -A && git commit -m "init wiki from wiki-keeper template"`.
6. Tell the human: drop sources into `raw/` (Obsidian Web Clipper for web articles), then run `/ingest raw/<file>`.

## Core invariants (mirror of template CLAUDE.md)

- One wiki per domain — never merge unrelated domains.
- `raw/` immutable; agent owns `wiki/`; every claim links to a source.
- Ingest touch cap: index + source page + ≤3 related pages. `/lint` handles deeper propagation.
- Contradictions get `> ⚠️ Conflict:` callouts, human resolves.
- Log lines: `## [YYYY-MM-DD] ingest|query|lint | Title` — grep-parseable.
- Index-first retrieval; `rg` fallback; NO search infrastructure until the human asks (~100+ sources).
- Good query answers get filed back as pages — this is the compounding mechanism. Always offer.

## Failure modes to avoid

- **Drift**: agent summaries with errors get cross-referenced and look authoritative → sourcing invariant + lint check #1 exist for this.
- **Bloated ingests**: touching 15 pages per source burns tokens and is unreviewable → touch cap.
- **Lost answers**: synthesis vanishing into chat history → file-back offer in `/query`.
- **Schema amnesia**: new sessions ignoring conventions → everything lives in the repo's CLAUDE.md, auto-loaded.

## Assets

- `assets/template/` — complete starter repo: CLAUDE.md schema, `/ingest` `/query` `/lint` commands, seeded index/log/synthesis, .gitignore.
