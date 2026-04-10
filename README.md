# code-wiki

Agent-maintained wiki where **code is the source of truth**. Captures decisions, rationale, and gotchas that code can't express. Zero infrastructure — flat markdown files.

## the problem

Your codebase has documentation that's either:
- **Outdated** — written 6 months ago, never updated
- **Fragmented** — same topic in 7 different files, no canonical version
- **Wrong** — describes code that's been refactored since
- **Missing** — the decisions and gotchas live in someone's head

Auto-generated docs (DeepWiki, Google Code Wiki) solve some of this — they describe **what** the code does. But they can't tell you **why** it's this way, what was tried and failed, or what breaks in production but not in tests.

## the solution

code-wiki fills the gap between what code shows and what engineers need to know:

```
code (raw)  →  wiki (processed)  →  index (navigation)
  ↑                ↑
  source of        decisions,
  truth            rationale,
                   gotchas
```

- **Code is the source of truth.** When wiki and code disagree, code wins.
- **Agents maintain it.** Writing happens as a workflow step, not a separate ceremony.
- **Zero infrastructure.** Flat markdown files. No vector DB, no indexing pipeline.
- **Trigger-based freshness.** Topics re-verified only when relevant code changes.

## quick start

```
/wiki-init          # scaffold wiki/
/wiki-bootstrap     # agent reads code, asks 5-15 questions, writes topics
```

That's it. ~15-25 minutes from zero to a working wiki.

## what a topic file looks like

```markdown
---
topic: search-api
last-verified: 2026-04-10
priority: core
tokens: 350
code-paths:
  - app/Services/ArticleQueryService.php
  - app/Http/Requests/BaseSearchRequest.php
related-topics: [api-standards, caching]
---

## overview
Unified search architecture using three-layer pattern: request validation → thin controller → query service.

## current behavior
- Reference implementation: ArticleQueryService (standalone, not extending base class)
- CSV parameter convention: filter[region_id]=1,2,3 parsed into arrays
- Region filter uses direct ID lookup (not nested whereHas) for index performance

## decisions
- Standalone QueryService over base class — why: more explicit, easier to customize per entity, avoids Template Method complexity.
- Direct ID lookup for region filters — why: nested whereHas produces full table scans. Measured: significant query time difference.

## gotchas
- PlaceQueryService still extends the old base class with unoptimized region filter. If debugging slow place queries, check this first.
- Route URLs are /api/articles on api.domain.com, not /portal/v1/ as some older docs claim.

## references
- app/Services/ArticleQueryService.php — reference implementation
- app/Http/Requests/BaseSearchRequest.php — CSV parsing, common validation
```

~350 tokens. An agent reads this in one load and knows what to do, what not to do, and where to look.

## commands

| Command | What it does | Human effort |
|---|---|---|
| `/wiki-init` | Scaffold wiki/ directory | ~2 min |
| `/wiki-bootstrap` | Agent reads code, interviews you, writes topics | 10-25 min |
| `/wiki-lint [--fix]` | Health audit (error/warning/info) | review |

## how it stays fresh

Topics have `code-paths` in frontmatter. When your review/CI workflow detects changes to:
- **Database migrations** overlapping a topic's code-paths → re-verify
- **API routes, controllers, resources** overlapping code-paths → re-verify
- **Refactoring** (flagged explicitly) → re-verify

Re-verification means the agent re-reads the code, compares claims, updates what changed, and stamps a new `last-verified` date.

`/wiki-lint` catches topics that slipped through triggers (180-day staleness warning).

## what it's not

- **Not auto-generated docs.** It doesn't describe every class and function. Your LSP and code graph tools do that better.
- **Not a RAG system.** No vector DB, no embeddings, no retrieval pipeline. At project scale (<50 topics), flat markdown + index outperforms retrieval.
- **Not a knowledge base for external sources.** Use Karpathy's LLM Wiki pattern for that. code-wiki is for codebase-internal knowledge.

## how it compares

| Tool | Source of truth | Captures decisions? | Infrastructure | Scale |
|---|---|---|---|---|
| Karpathy LLM Wiki | External docs | Yes | Markdown | Personal |
| DeepWiki | Code (auto-gen) | No | Server + LLM API | Any repo |
| Google Code Wiki | Code (auto-gen) | No | Cloud service | Enterprise |
| repowise | Code + git history | Partial (markers) | SQL + vector + graph | Enterprise |
| **code-wiki** | **Code (verified)** | **Yes (human-curated)** | **Markdown files** | **Team** |

## works with

- Claude Code (skills in `.claude/skills/`)
- Codex CLI (via AGENTS.md)
- Cursor (via .cursor/rules/)
- Gemini CLI (via GEMINI.md)
- Any agent that can read markdown files

## inspired by

- [Andrej Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — three-layer architecture (raw/processed/index)
- [llms.txt standard](https://llmstxt.org/) — priority tiers and token budgeting
- [wiki-skills](https://github.com/kfchou/wiki-skills) — severity-tiered lint
- [repowise](https://github.com/repowise-dev/repowise) — decision intelligence from code

## license

MIT
