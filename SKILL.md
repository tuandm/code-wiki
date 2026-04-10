---
name: code-wiki
description: |
  Agent-maintained wiki where code is the source of truth. Captures decisions, rationale, and gotchas that code can't express. Zero infrastructure — flat markdown files. Works with Claude Code, Codex, Cursor, Gemini CLI.
---

# code-wiki

A wiki system for codebases where:
- **Code is the source of truth.** The wiki captures what code can't express — decisions, rationale, gotchas.
- **Agents maintain it.** Writing happens as a workflow step, not a separate ceremony.
- **Zero infrastructure.** Flat markdown files. No vector DB, no indexing pipeline, no separate server.
- **Trigger-based freshness.** Topics re-verified only when relevant code changes (migrations, API routes, refactors), not on every commit.

## commands

| Command | What it does | Human effort |
|---|---|---|
| `/wiki-init` | Scaffold wiki/ in any project | ~2 min |
| `/wiki-bootstrap` | Agent reads code, asks 5-15 questions, writes the initial wiki | 10-25 min |
| `/wiki-lint [--fix]` | Health audit with severity tiers | review |

## getting started

```
/wiki-init          # scaffold wiki/
/wiki-bootstrap     # agent reads code, asks you 5-15 questions, writes topics
```

That's it. ~15-25 minutes from zero to a working wiki.

## philosophy

Most LLM wiki tools either synthesize external sources (Karpathy pattern) or auto-generate from code (DeepWiki, Google Code Wiki). Neither captures **why** — the decisions, trade-offs, and gotchas that only humans know.

code-wiki fills the gap between what code shows and what engineers need to know. It doesn't duplicate what code already says (use your LSP and code graph tools for that). It stores the things that would otherwise live in someone's head and leave when they do.

## keeping it fresh

Topics have `code-paths` in frontmatter. When your review workflow detects changes to migrations, API routes, or controllers overlapping a topic's code-paths, the agent re-verifies that topic — re-reads the code, compares claims, updates what changed, stamps a new date.

`/wiki-lint` catches topics that slipped through triggers.

## works with

Claude Code, Codex CLI, Cursor, Gemini CLI — any agent that can read markdown files.
