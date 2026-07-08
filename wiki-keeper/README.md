# wiki-keeper

> LLM Wiki as a Claude Code skill: the agent builds and maintains a persistent, interlinked markdown knowledge wiki; you curate sources and ask questions.

## Purpose

RAG re-derives knowledge on every query; nothing accumulates. This package makes the agent compile knowledge **once at ingest** into a wiki that stays current — cross-references, flagged contradictions, and evolving synthesis included. The tool is a discipline layer (schema + commands), not an app: Claude Code is the engine.

## What's in the box

```
wiki-keeper/                     # install as a skill (~/.claude/skills/ or ejoliet/claude-skills)
├── SKILL.md                     # triggering + init/operate workflows
└── assets/template/             # complete starter wiki repo
    ├── CLAUDE.md                # THE schema: invariants, page conventions, workflows
    ├── .gitignore
    ├── .claude/commands/
    │   ├── ingest.md            # /ingest raw/<file>
    │   ├── query.md             # /query <question>
    │   └── lint.md              # /lint
    ├── raw/assets/              # immutable sources + downloaded images
    └── wiki/                    # agent-owned: index.md, log.md, synthesis.md, sources/, entities/, concepts/
```

## Architecture

Three layers per the LLM Wiki pattern:

| Layer | Owner | Contents |
|-------|-------|----------|
| `raw/` | Human | Immutable sources (Obsidian Web Clipper output, papers, notes) |
| `wiki/` | Agent | Interlinked markdown: source summaries, entities, concepts, synthesis |
| `CLAUDE.md` | Both | Schema — auto-loaded each session, co-evolved over time |

Key design decisions (rationale in the earlier design discussion):

> 💡 **Index-first retrieval, no RAG.** `index.md` catalog + `rg` fallback. Holds to ~100 sources.
> 💡 **Ingest touch cap** (index + source page + ≤3 related pages) keeps turns cheap; `/lint` batch-propagates.
> 💡 **File-back mechanism**: `/query` offers to save good answers as pages — the compounding loop.
> 💡 **Claim sourcing invariant** counters wiki drift (agent errors laundered into "authoritative" pages).

## Stack

No library decisions — deliberately zero-dependency.

| Layer | Chosen | Why | Rejected / deferred |
|-------|--------|-----|---------------------|
| Storage | Markdown + git | Portable, diffable, Obsidian-compatible | Databases (overkill) |
| Retrieval | index.md + ripgrep | Sufficient at target scale, zero infra | Embeddings/RAG (rebuilds the problem) |
| Search (Phase 2) | qmd or 50-line BM25 script | Only if index degrades | Adopt now (premature; verify qmd health first) |
| Browsing | Obsidian (optional) | Graph view, web clipper, Marp/Dataview plugins | — |

## Quick start

1. Install the skill: copy `wiki-keeper/` into your skills location (or add to `ejoliet/claude-skills` and sync).
2. In Claude Code: "new wiki on <topic> in ~/wikis/<name>" → skill scaffolds from template.
3. Clip 2-3 articles into `raw/` (Obsidian Web Clipper → markdown).
4. `/ingest raw/<file>` — agent discusses takeaways, then files everything.
5. `/query <question>` — answers with citations; accept the file-back offer for good syntheses.
6. Weekly-ish: `/lint`.

## Interface contract

| Command | Input | Output | Side effects |
|---------|-------|--------|--------------|
| `/ingest` | path in `raw/` | Takeaway discussion, then report of pages touched | +1 source page, ≤3 page updates, index + log entries |
| `/query` | free-text question | Cited answer (md/table/chart); file-back offer | Optional answer page + index/log entries |
| `/lint` | — | Issue table (issue \| page \| fix) | Approved fixes applied; log + burned-log entries |

## Non-goals (v1)

- Search infrastructure, embeddings, MCP server
- Multi-user / team review flows
- Automated batch ingest pipelines
- Cross-wiki federation

## Open questions

- [ ] Dogfood domain: health/longevity research (recommended) — confirm
- [ ] Frontmatter tags: adopt a controlled vocabulary per wiki, or free-form? (suggest: free-form until lint shows tag sprawl)
- [ ] Image handling: enforce local downloads in schema, or per-wiki? (currently: convention noted in ingest command only)

## Next steps

1. Drop `wiki-keeper/` into `ejoliet/claude-skills`, commit.
2. Init the health/longevity wiki; seed `raw/` with 5-10 already-read sources.
3. Ingest one source end-to-end; fix schema friction; log fixes in "Recently burned".
4. After ~10 sources: first `/lint` pass; evaluate whether the touch cap and index scale.
5. Phase 2 trigger: index misses answers or exceeds ~100 sources → add search script or evaluate qmd (check repo activity first).

## Acceptance criteria

- [ ] Fresh init produces a git-committed wiki with dated index/log/synthesis
- [ ] `/ingest` on a real article: pauses for takeaway discussion, respects touch cap, every claim sourced
- [ ] `/query` spanning ≥2 sources: cited answer + file-back offer; filed page appears in index
- [ ] `/lint` on a wiki with a planted unsourced claim and an orphan page: catches both, fixes only after approval
- [ ] New Claude Code session on the wiki repo follows the schema without re-instruction
