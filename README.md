![code-wiki](https://github.com/user-attachments/assets/15ea0805-4091-4d9d-8f92-646e18a55556)

# code-wiki

**Your codebase knows *what*. This wiki captures *why*.**

An agent skill that builds and maintains a knowledge wiki from your source code — not from external documents, not auto-generated from ASTs, but from the decisions, rationale, and gotchas that live in engineers' heads and leave when they do.

> *"We had 230 documentation files across three repos. Seven of them described the same search API. None agreed with each other. None agreed with the code."*

That's the problem we built this to solve.

---

## the story

In April 2026, Andrej Karpathy posted his [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) pattern — a three-layer architecture (raw sources → LLM-maintained wiki → index) for personal knowledge bases. We loved the concept and tried to apply it to our production Laravel monorepo.

It didn't fit. Here's what we found:

**Karpathy's wiki consumes external sources** (papers, articles, tweets). Our "source" was the code itself. We didn't need an LLM to synthesize research papers — we needed it to capture why we chose Scout over Meilisearch, why the region filter uses a direct ID lookup instead of nested `whereHas`, and what breaks when you deploy without restarting the SSR server.

**Auto-generated wikis (DeepWiki, Google Code Wiki) describe structure, not decisions.** They'll tell you `ArticleQueryService` has 12 methods. They won't tell you that `PlaceQueryService` uses an older pattern that causes full table scans, and you should follow the Article pattern, not the Place pattern.

**Session-memory tools (claude-memory-compiler) capture conversations, not verified facts.** They're great at recording what was discussed. They can't verify whether the discussion's conclusions are still true six months later.

So we built something different.

We researched [10 tools](https://github.com/safishamsi/graphify) [across](https://github.com/kfchou/wiki-skills) [the](https://github.com/mduongvandinh/llm-wiki) [ecosystem](https://github.com/repowise-dev/repowise), took the best practices from each (severity-tiered lint from wiki-skills, token budgeting from llms.txt, decision intelligence from repowise), skipped what didn't fit (knowledge graphs, vector DBs, forgetting curves), and built a wiki system where:

- **Code is the source of truth.** When the wiki says X and the code says Y, the code wins. Always.
- **The agent writes it.** Humans answer 5-15 questions. The agent reads the code, drafts the topics, maintains them over time.
- **Zero infrastructure.** Flat markdown files. No database, no indexing pipeline, no separate server. Works tomorrow, not after a sprint of setup.

We used it on a real project — a Laravel monorepo with 50k articles, 200k photos, 3 sub-projects, and 7 months of doc debt. The wiki reduced agent context-loading tokens by ~90% per task and eliminated an entire class of bugs caused by agents reading stale documentation.

Then we extracted the project-specific parts and packaged the rest for you.

---

## quick start

```
/wiki-init          # scaffold wiki/ (~2 min)
/wiki-bootstrap     # agent reads code, asks 5-15 questions, writes topics (~15 min)
```

That's it. From zero to a working wiki in under 20 minutes.

**What happens during bootstrap:**

```
Agent: I've read your project. Here are 8 architectural areas I found:

  1. Auth — Sanctum, 3 guard types
  2. Search — Scout + Algolia
  3. Caching — Redis, two stores
  4. Deployment — Envoy scripts
  ...

  For each area, give me 1-2 sentences on:
  (a) why you chose this approach
  (b) the biggest gotcha for someone new

  Skip any that are "standard setup."

You:   Search — chose Algolia because team already ran it,
       ops cost of a second search backend wasn't worth the feature gap.
       Gotcha: PlaceQueryService uses an older pattern that's slow.

Agent: [writes search.md with verified current-behavior + your decisions + gotcha]
```

The agent reads the code for facts. You provide the knowledge code can't express. The wiki captures both.

---

## what a topic file looks like

```markdown
---
topic: search-api
last-verified: 2026-04-10
priority: core
rank: 2
tokens: 350
code-paths:
  - app/Services/ArticleQueryService.php
  - app/Http/Requests/BaseSearchRequest.php
related-topics: [api-standards, caching]
---

## overview
Unified search using three-layer pattern: request validation → thin controller → query service.

## current behavior
- Reference implementation: ArticleQueryService (standalone, doesn't extend base class)
- CSV filter convention: filter[region_id]=1,2,3 parsed into arrays
- Region filter uses direct ID lookup for index performance

## decisions
- Standalone QueryService over base class — why: more explicit, avoids Template Method complexity.
  *Supersedes: BaseSearchQueryService inheritance (removed 2025-12).*
- Direct ID lookup for region filters — why: nested whereHas causes full table scans.

## gotchas
- PlaceQueryService still uses the old base class. If debugging slow place queries, check this first.
- Route URLs are /api/articles on api.domain.com, not /portal/v1/ as older docs claim.

## references
- app/Services/ArticleQueryService.php — reference implementation
```

~350 tokens. An agent reads this in one load and knows what to do, what not to do, and where to look.

---

## commands

| Command | What it does | Your time |
|---|---|---|
| `/wiki-init` | Scaffold `wiki/` directory with index, conventions, log | ~2 min |
| `/wiki-bootstrap` | Agent reads code, interviews you, writes initial topics | 10-25 min |
| `/wiki-lint [--fix]` | Health audit: errors, warnings, info. `--fix` re-verifies stale topics | review only |

Three commands. No build step, no config file, no API keys.

---

## how it stays fresh

Every topic file has `code-paths` in its frontmatter — the specific files and directories it describes. When your workflow detects changes to:

- **Database migrations** overlapping a topic's code-paths → agent re-verifies the topic
- **API routes, controllers, resources** overlapping code-paths → agent re-verifies
- **Refactoring** (explicitly flagged) → agent re-verifies

Re-verification: agent re-reads the code, compares every claim in the topic against current behavior, updates what changed, bumps `last-verified`.

**Three triggers for new topics:**

1. **Query-filing** — an agent researches an area and the answer isn't in the wiki. It proposes filing the answer as a new topic. The wiki grows from questions actually asked.
2. **Review-check** — after completing a task, the agent checks if it touched uncovered code areas. If yes, proposes a new topic.
3. **Bootstrap** — the initial creation. Runs once.

`/wiki-lint` catches what triggers miss — 180-day staleness warning, broken cross-references, orphaned topics.

---

## how it compares

```
                    captures     captures     zero        code as
                    structure?   decisions?   infra?      source of truth?

Karpathy LLM Wiki      -           yes         yes            -
DeepWiki               yes          -          (server)       yes
Google Code Wiki       yes          -          (cloud)        yes
repowise               yes        partial     (3 DBs)        yes
code-wiki               -          yes         yes            yes
```

Every other tool either auto-generates structure docs (missing decisions) or synthesizes external sources (missing code verification). code-wiki is the only one that captures human decisions **and** verifies facts against code, with nothing but markdown files.

---

## what it's not

- **Not auto-generated docs.** It doesn't describe every class and function — your LSP and code graph tools do that better. It describes what they *can't*: why the code is this way.
- **Not a RAG system.** No vector DB, no embeddings. At project scale (<50 topics), flat markdown with an index outperforms retrieval infrastructure.
- **Not for external knowledge.** Use [Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) for papers and articles. code-wiki is for the knowledge inside your codebase.

---

## the philosophy

Most documentation fails because it tries to describe what the code does. The code already does that. Documentation should capture what the code *can't express*:

- **Decisions**: "We chose X over Y because Z." Without this, the next engineer re-evaluates the same options.
- **Superseded approaches**: "This replaces the old pattern from March." Without this, agents find the old pattern in git history and recommend it.
- **Gotchas**: "This looks wrong but is intentional because of A." Without this, someone 'fixes' it and breaks production.
- **Constraints**: "Legal requires session tokens stored this way." Without this, a refactor violates compliance.

If the code can tell you something, the wiki shouldn't repeat it. If only a human knows it, the wiki should capture it before they forget.

---

## works with

- **Claude Code** — skills in `.claude/skills/`
- **Codex CLI** — via AGENTS.md
- **Cursor** — via `.cursor/rules/`
- **Gemini CLI** — via GEMINI.md
- **Any agent** that can read markdown files

The wiki is just markdown. The skills are just instructions. No vendor lock-in.

---

## project structure

```
code-wiki/
├── SKILL.md                  # entry point for agent skill loaders
├── README.md
├── skills/
│   ├── wiki-init.md          # scaffold wiki/ in any project
│   ├── wiki-bootstrap.md     # code-first interview, writes topics
│   └── wiki-lint.md          # health audit, severity-tiered
└── templates/
    ├── conventions.md         # format spec, triggers, creation rules
    └── topic-template.md      # blank topic file with all sections
```

---

## inspired by

Built on ideas from across the ecosystem:

- [Andrej Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — the three-layer architecture (raw → processed → index)
- [llms.txt standard](https://llmstxt.org/) — priority tiers and token budgeting for LLM consumption
- [wiki-skills](https://github.com/kfchou/wiki-skills) — severity-tiered lint and query-filing pattern
- [repowise](https://github.com/repowise-dev/repowise) — decision intelligence and staleness tracking from code
- [Graphify](https://github.com/safishamsi/graphify) — WHY-comment extraction from code annotations
- [claude-memory-compiler](https://github.com/coleam00/claude-memory-compiler) — session-based knowledge compilation

We researched all of them, took what worked for codebase wikis, and left what didn't.

---

## license

MIT
