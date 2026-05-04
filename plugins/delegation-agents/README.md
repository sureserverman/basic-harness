# delegation-agents

Three model-pinned worker subagents and one dispatching skill that let a parent session on Opus offload bulk reading, document editing, and document drafting to cheaper, faster models.

Domain-agnostic — works for research, writing, project management, analysis, not just coding.

## Why

Claude Code lets a subagent pin its own model via the `model` field in frontmatter. A parent session on Opus that spawns one of these agents pays Haiku or Sonnet rates for that subtree of work.

The Anthropic *Choosing a Model* guide maps the tiers cleanly:

- **Haiku** — real-time, high-volume, latency-sensitive, straightforward.
- **Sonnet** — balanced intelligence for transformation, editing, generation.
- **Opus** — complex reasoning, multi-hour orchestration, architecture calls.

Most workflows have phases that are cheaper than their caller needs to be. These three agents are the delegation targets for the common cases.

## Install

```text
/plugin marketplace add sureserverman/basic-harness
/plugin install delegation-agents@basic-harness
```

## What's inside

| Component | Tier | Tools | What it does |
|---|---|---|---|
| `bulk-reader` agent | Haiku | `Read`, `Glob`, `Grep`, `Bash` (read-only), `WebFetch` | Bulk file reads, greps, link checks, log sampling, frontmatter extraction. Never writes. |
| `document-rewriter` agent | Sonnet | `Read`, `Edit`, `Glob`, `Grep` | Rewrites existing documents to a spec — tighten a brief, apply marked-up edits, sync to a template. Never creates or deletes files. |
| `draft-generator` agent | Sonnet | `Read`, `Write`, `Edit`, `Glob`, `Grep`, `Bash` (verify-only) | Generates new documents from a brief — meeting agendas, outlines, scaffolded wiki pages, templated deliverables. No installs, no commits, no sending. |
| `dispatching-parallel-agents` skill | (caller's tier) | — | Fans out independent tasks from a `planning-projects` plan to the three agents above in a single concurrent batch, integrates results, propagates the dependency graph. |

## When to use which

| Task shape | Agent |
|---|---|
| "Read these 80 source files and extract their frontmatter" | `bulk-reader` |
| "Check that every URL in this folder of drafts still resolves" | `bulk-reader` |
| "Apply the editor's marked changes to chapters 3, 4, 5" | `document-rewriter` (one per chapter, dispatched in parallel) |
| "Tighten the executive summary on this brief to 200 words" | `document-rewriter` |
| "Generate four panel-ask emails from these four panelist profiles" | `draft-generator` (one per email, parallel) |
| "Scaffold a fresh `Sources/<name>.md` page with this frontmatter and these section headings" | `draft-generator` |
| "I have a `planning-projects` plan with 4 Parallel YES tasks" | `dispatching-parallel-agents` (which then picks the right agent per task) |

## Hard rules across all three agents

- **Stay in the caller's named scope.** No walking the tree, no opportunistic edits.
- **No installs.** If a tool is missing, report it.
- **No external mutations.** No commits, no pushes, no sends, no publishes. The caller does those.
- **One concern per invocation.** Don't bundle a rewrite with a frontmatter fix; don't bundle a draft with a research pass.

These are by design — they are the discipline that makes a delegation pattern work without surprises.

## Pairs with

- `process-skills/planning-projects` — produces plans with the Parallel YES + Confirm-Adjust max cycles fields the dispatching skill consumes.
- `process-skills/executing-plans` — typical caller of the dispatching skill.
- `vault-librarian` + upstream `obsidian-wiki` — drafting and editing targets for users who keep a Markdown notes vault.
