---
name: wiki-bootstrap
description: |
  Agent reads codebase, identifies architectural areas, auto-drafts current-behavior sections from code, asks 5-15 high-level questions about decisions and gotchas, then writes the initial wiki topics. Total human effort: 10-25 minutes.
---

<objective>
Bootstrap a wiki for a project that has code but no (or minimal) existing docs. The agent does the reading; the human provides the knowledge that code can't express.
</objective>

<principles>
- **Ask per-area, not per-file.** A 500-file project has ~8-12 architectural areas, not 500 questions.
- **Auto-derive what you can.** "Current behavior" sections come from code — don't ask the human what the code already says.
- **Only ask what code can't answer.** Decisions (why this over alternatives), gotchas (what breaks that tests don't catch), constraints (business rules that shaped technical choices).
- **Batch questions.** Present all questions at once, not one-by-one. Human answers in one pass.
- **Write, don't ask for perfection.** First draft doesn't need to be complete. The wiki compounds over time via review-cycle updates.
</principles>

<process>

## Phase 1: Scan (no human input needed)

**1a. Read project structure.**
- List top-level directories
- Identify framework (Laravel, Rails, Next.js, Django, Express, Go, etc.)
- Find key config files (database config, cache config, queue config, auth config)
- Find route definitions
- Find model/entity definitions
- Find service/business-logic directories
- Count: models, routes, services, migrations, tests

**1b. Identify architectural areas.**
Group what you found into areas. Typical areas for a web app:

| Area | How to detect |
|---|---|
| Auth/SSO | auth config, middleware, guards, login routes |
| Database/Models | migrations, models, relationships |
| API/Routes | route files, controllers, resources |
| Search | search service, Scout/Elasticsearch/Algolia config |
| Caching | cache config, Redis config, cache service |
| File/Media | upload handlers, storage config, media models |
| Background jobs | queue config, job classes |
| Frontend | JS framework, components, pages |
| Deployment | deploy scripts, CI config, Dockerfiles |
| Testing | test directories, test config |
| External integrations | API clients, webhooks, third-party SDKs |

Not every project has all of these. List only areas that actually exist in the code.

**1c. Auto-draft current behavior for each area.**
For each detected area, write a bullet-point "current behavior" section by reading the code:
- What framework/library is used
- How it's configured
- What entities/routes/services exist
- How they connect

This becomes the "current behavior" section of each topic file. No human input needed for this part.

## Phase 2: Interview (5-15 minutes of human input)

**2a. Present the area map.**
Show the human what you found:
```
I've read your project. Here are the {N} architectural areas I found:

1. Auth — Sanctum token-based, 3 guard types
2. Database — MySQL, 45 models, Redis cache
3. API — 120 routes across 15 controllers
4. Search — Scout with Algolia driver
5. Caching — Redis, two stores (default + shared)
6. Media — spatie/medialibrary, S3 storage
7. Deployment — GitHub Actions + Envoy
8. Frontend — Vue 3 + Inertia.js, SSR enabled

Missing anything? (add/remove/rename before I ask questions)
```

Wait for human to confirm or adjust the area list.

**2b. Ask questions — one per area, all at once.**
For each confirmed area, ask exactly one question that combines the "why" and "gotcha":

```
For each area below, give me 1-2 sentences on:
(a) why you chose this approach (or skip if obvious)
(b) the biggest gotcha or "watch out" for someone new

1. Auth (Sanctum + 3 guards):
2. Search (Scout + Algolia):
3. Caching (two Redis stores):
4. Media (medialibrary + S3):
5. Deployment (Envoy):
6. Frontend (Inertia SSR):

Skip any that are too obvious to document. "Standard Laravel setup" is a valid answer.
```

The human answers in 1-2 sentences each. Some will say "skip" or "standard setup" — that's fine, those areas get a minimal topic or no topic at all.

**2c. One open-ended question at the end.**
```
Anything else I should know that doesn't fit the areas above?
The "don't touch" list, political constraints, historical debt, 
things that look wrong but are intentional?
```

This catches cross-cutting concerns that don't map to a single area.

## Phase 3: Write (no human input needed)

**3a. For each area with substantive answers, create a topic file.**
Combine auto-drafted "current behavior" (from Phase 1) with human-provided "decisions" and "gotchas" (from Phase 2).

Format per `wiki/conventions.md`:
```markdown
---
topic: {area-name}
last-verified: {today}
priority: core
tokens: {estimate}
code-paths:
  - {relevant directories}
related-topics: [{other areas}]
---

## overview
{one sentence from Phase 1 scan}

## current behavior
{bullet points from Phase 1 auto-draft}

## decisions
{from human answer (a) — decision → why}

## gotchas
{from human answer (b) — the watch-out}

## references
{code paths from Phase 1 scan}
```

**3b. Assign ranks.**
Ask the human to rank the created topics by importance (1 = most important, 10 = least):
```
Please rank these topics by importance (1 = most critical for a new engineer to read first):

1. auth
2. search
3. caching
4. deployment
5. media

Or just say "auto" and I'll rank by code complexity + number of gotchas.
```

If "auto": rank by number of decisions + gotchas (more = higher rank, because more knowledge at risk of being lost).

**3c. For areas where the human said "skip" or "standard setup":**
Don't create a topic file. These areas don't have wiki-worthy knowledge — the code speaks for itself.

**3d. Run backlink audit.**
For each created topic, scan all other topics for mentions of the topic name or its code-paths. Propose `related-topics` entries in both directions. Apply automatically (no human input needed for cross-references).

**3e. Update index.md.**
Add all created topics to the index with token estimates, priority tags, and ranks. Sort by rank within each priority tier.

**3f. Update log.md.**
```
{today} | created | {topic} | wiki-bootstrap: auto-drafted from code + human interview
```

## Phase 4: Review (5-10 minutes of human input)

Present the full list of created topics with a one-line summary each:
```
Created {N} wiki topics:

1. auth.md — Sanctum multi-guard, SSO gotcha with token refresh (~200 tokens)
2. search.md — Scout+Algolia, chose over Meilisearch for hosted ops (~250 tokens)
3. caching.md — two-tier Redis, gotcha: must clear shared after deploys (~200 tokens)
4. deployment.md — Envoy symlink releases, gotcha: SSR restart on portal (~300 tokens)

Skipped (standard setup, no wiki-worthy decisions):
- database (standard MySQL + Eloquent)
- frontend (standard Inertia setup)
- testing (standard Pest)

Want to review any topic in detail, or approve all?
```

Human approves or asks to see/edit specific topics. Once approved, bootstrap is complete.

</process>

<output>
Print final summary:
```
Wiki bootstrapped: {N} topics, ~{total} tokens.
Next steps:
- Topics will be re-verified automatically when you change migrations, routes, or controllers.
- Run /wiki-lint periodically to check health.
- To add this to your agent's auto-loaded context, add to your CLAUDE.md / AGENTS.md:
  @{wiki-path}/index.md
```
</output>
