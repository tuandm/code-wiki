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

## installation

### Claude Code

Clone or copy into your project's `.claude/skills/` directory:

```bash
# option 1: clone into skills
git clone https://github.com/tuandm/code-wiki.git .claude/skills/code-wiki

# option 2: copy just the skills (no git history)
cp -r /path/to/code-wiki/skills/ .claude/skills/code-wiki/
```

Then create command wrappers so you can use `/wiki-init`, `/wiki-bootstrap`, `/wiki-lint`:

```bash
# .claude/commands/wiki-init.md
cat > .claude/commands/wiki-init.md << 'EOF'
---
description: Scaffold a wiki/ directory in the current project
---
Read the skill file at .claude/skills/code-wiki/wiki-init.md and execute its process steps.
EOF

# .claude/commands/wiki-bootstrap.md
cat > .claude/commands/wiki-bootstrap.md << 'EOF'
---
description: Bootstrap wiki by reading code and interviewing you
---
Read the skill file at .claude/skills/code-wiki/wiki-bootstrap.md and execute its process steps.
EOF

# .claude/commands/wiki-lint.md
cat > .claude/commands/wiki-lint.md << 'EOF'
---
description: Audit wiki health with severity-tiered checks
argument-hint: [--fix]
---
Read the skill file at .claude/skills/code-wiki/wiki-lint.md and execute its process steps for: $ARGUMENTS
EOF
```

After this, `/wiki-init`, `/wiki-bootstrap`, and `/wiki-lint` work as slash commands in Claude Code.

### Codex CLI

Copy the skills into your project and add to AGENTS.md:

```bash
cp -r /path/to/code-wiki/skills/ .agents/skills/code-wiki/
```

Add to your `AGENTS.md`:
```markdown
## wiki skills
- Read `.agents/skills/code-wiki/wiki-init.md` when asked to set up a wiki
- Read `.agents/skills/code-wiki/wiki-bootstrap.md` when asked to bootstrap a wiki
- Read `.agents/skills/code-wiki/wiki-lint.md` when asked to lint the wiki
```

### Cursor

Copy the skills and add to `.cursor/rules/`:

```bash
cp -r /path/to/code-wiki/skills/ .cursor/skills/code-wiki/
```

Add a rule file `.cursor/rules/code-wiki.md`:
```markdown
When the user asks about wiki, documentation, or tribal knowledge:
- For setup: read .cursor/skills/code-wiki/wiki-init.md
- For bootstrapping: read .cursor/skills/code-wiki/wiki-bootstrap.md
- For health checks: read .cursor/skills/code-wiki/wiki-lint.md
```

### Gemini CLI

Copy the skills and reference in `GEMINI.md`:

```markdown
## wiki skills
- Wiki setup: read `skills/code-wiki/wiki-init.md`
- Wiki bootstrap: read `skills/code-wiki/wiki-bootstrap.md`
- Wiki lint: read `skills/code-wiki/wiki-lint.md`
```

### Any other agent

The skills are plain markdown instruction files. Any agent that can read files and follow instructions can use them. Point your agent at the skill file and tell it to execute the steps.

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
