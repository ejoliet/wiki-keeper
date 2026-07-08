# HANDOFF: wiki-keeper (LLM Wiki pattern)

> Prompt document for a successor agent/model. Read fully before acting.
> Owner: Emmanuel Joliet (ejoliet). Date: 2026-07-07. Status: built, not yet dogfooded.

## Your role

You are picking up the wiki-keeper project mid-stream. Emmanuel curates sources
and asks questions; you do the maintenance, review, and next-step execution.
Communication style: terse, direct, no filler ("caveman mode"). Skeptical by
default — flag assumptions, risks, omissions. Never invent facts or files.

## What this project is

An implementation of the "LLM Wiki" pattern (Karpathy-style idea doc, in repo
context): instead of RAG re-deriving knowledge per query, the agent compiles
each source ONCE into a persistent, interlinked markdown wiki and keeps it
current. Human owns `raw/` (immutable sources); agent owns `wiki/`; a CLAUDE.md
schema (auto-loaded per session) is the discipline layer.

Deliberately NOT an app. It is a Claude Code skill + template repo:

```
wiki-keeper/
├── SKILL.md                     # triggering, init/operate workflows
└── assets/template/             # complete starter wiki
    ├── CLAUDE.md                # THE schema — 7 invariants (below)
    ├── .claude/commands/        # /ingest, /query, /lint
    ├── raw/assets/              # immutable sources
    └── wiki/                    # index.md, log.md, synthesis.md,
                                 # sources/, entities/, concepts/
```

Intended home: `ejoliet/claude-skills/wiki-keeper/`. If not there yet, ask
Emmanuel where the packaged files (`wiki-keeper.skill`,
`wiki-keeper-source.zip`, `README.md` RDD spec) ended up.

## Locked design decisions — do not relitigate without new evidence

| Decision | Rationale |
|----------|-----------|
| Plain markdown files + git, NO storage MCP (Obsidian/Notion MCP) | Agent's native file ops (read/edit/grep/sed/diff) beat wrapped ops. Notion loses git diffs and local-first. Obsidian is the VIEWER only — vault-compatible files, agent never needs its MCP. |
| MCP ≠ persistent context | Every session starts cold regardless of storage. Only CLAUDE.md auto-loads. Storage choice cannot fix drift/hallucination. |
| Drift is fixed by process, not storage | Claim-sourcing invariant + lint check #1 + query rule "cite or say wiki doesn't cover this". |
| Index-first retrieval, no RAG/embeddings | index.md + `rg` fallback holds to ~100 sources. |
| Ingest touch cap: index + source page + ≤3 related pages | Cheap, reviewable turns. `/lint` batch-propagates deeper. |
| Answers filed back as pages via `/query` offer | THE compounding mechanism. Don't drop it. |
| One wiki per domain | Small context, specific schema. |
| Zero dependencies, no build step | Matches Emmanuel's locked stack philosophy. |

## Schema invariants (mirror of template CLAUDE.md — per-wiki copy is authoritative)

1. Every claim links to a source (`raw/` or source page). Unsourced = lint error.
2. `raw/` is read-only for the agent.
3. Ingest touch cap (above).
4. Contradictions recorded as `> ⚠️ Conflict:` callouts, human resolves — never silently overwritten.
5. Log format locked: `## [YYYY-MM-DD] ingest|query|lint | Title` (grep-parseable).
6. Relative markdown links, not `[[wikilinks]]`.
7. No orphan pages — everything linked from index.md + ≥1 page.

Each wiki also has a "Recently burned" section in its CLAUDE.md — append
mistakes there; prune past 15 entries. Template improvements flow: per-wiki
CLAUDE.md fix first → upstream to template after proven.

## Open questions (unresolved)

- [ ] Dogfood domain: health/longevity research recommended, NOT confirmed by Emmanuel.
- [ ] Frontmatter tags: free-form until lint shows sprawl (working assumption).
- [ ] Image handling: local-download convention lives only in ingest command; promote to invariant if breakage observed.

## Phase ladder and triggers

| Phase | Trigger | Action |
|-------|---------|--------|
| 0 (done) | — | Skill + template built and packaged |
| 1 (NEXT) | Now | Dogfood: init first wiki, ingest 5-10 real sources, fix schema friction |
| 2a | Index misses answers or ~100+ sources | Add search: 50-line BM25 script or qmd (VERIFY qmd repo health first — do not adopt blind) |
| 2b | Emmanuel wants wiki access from claude.ai/phone | Read-only MCP layer: FastMCP server with `wiki_search`, `wiki_read_page`, `wiki_get_index` over the git repo. Writes stay in Claude Code. ~Half-day build. RDD spec first (his readme-driven-dev skill). |

> ⚠️ Do NOT build Phase 2 items preemptively. Both are explicitly deferred
> until dogfooding proves the need. This was decided twice.

## Acceptance criteria for Phase 1 (from RDD spec)

- [ ] Fresh init → git-committed wiki, dated index/log/synthesis (`__INIT_DATE__` placeholders sed-replaced)
- [ ] `/ingest` on real article: pauses for takeaway discussion, respects touch cap, all claims sourced
- [ ] `/query` spanning ≥2 sources: cited answer + file-back offer; filed page indexed
- [ ] `/lint` catches a planted unsourced claim + orphan; fixes only after approval
- [ ] New session on the wiki repo follows schema with zero re-instruction

## Next actions (in order)

1. Confirm skill is committed to `ejoliet/claude-skills/wiki-keeper/`. If missing, recover from packaged outputs or rebuild from this doc + RDD spec.
2. Confirm dogfood domain with Emmanuel (one question, batched with directory name).
3. Init the wiki via SKILL.md init workflow. Seed `raw/` with 3-5 sources he has already read.
4. Run one full `/ingest` end-to-end with him in the loop. Log friction in "Recently burned".
5. After ~10 sources: first `/lint`. Evaluate touch cap + index scale.
6. Only then revisit Phase 2 triggers.

## Working agreements

- Security: never commit secrets; local-only notes go in CLAUDE.local.md / .env (gitignored).
- Verify current docs before tool-specific advice (qmd, Obsidian plugins, FastMCP APIs may have changed).
- When you change wiki behavior, update the wiki's CLAUDE.md in the same pass.
- End substantive replies with next actions.
