# basic-harness

Starter scaffolding for Claude Code. Process discipline, knowledge management, and delegation patterns that work for any kind of structured work — not only coding.

**Claude Code only.** Works on the CLI and the desktop app (macOS / Windows). No external infrastructure required.

## Who it's for

Three personas drive the design:

- **Researchers and analysts** — read, take notes, write briefs, run multi-week investigations.
- **Writers and journalists** — long-form work with sources, drafting, editing.
- **Project leads and consultants** — engagements with deliverables, planning, stakeholder updates.

If you write software all day, the parent project [`coder-plugins`](https://github.com/sureserverman/coder-plugins) is the better starting point.

## Install

From any Claude Code session (CLI or desktop):

```text
/plugin marketplace add sureserverman/basic-harness
/plugin install process-skills@basic-harness
/plugin install delegation-agents@basic-harness
```

Restart Claude Code (or `/exit` and reopen) to register the skills. **That's the minimum viable install** — process discipline + delegation patterns. No vault, no notes, no extra dependencies.

### Optional: notes vault

If you want a local notes vault to ingest sources into and query from inside Claude Code, also install:

```text
/plugin install vault-librarian@basic-harness
/plugin marketplace add sureserverman/obsidian-wiki-plugin
/plugin install obsidian-wiki@obsidian-wiki
```

Then run `/vault-librarian:bootstrap-vault`. The vault is **purely local** — a directory of Markdown files at a path you pick (default `~/dev/knowledge`). No network, no cloud, no account. If you want sync across machines later, layer on Obsidian Sync, Syncthing, iCloud Drive, or whatever you already use — none of it is built into basic-harness.

If you don't want a notes vault at all, skip this section. Everything else still works.

## What's inside

### `process-skills` plugin

Six domain-agnostic skills:

- **`brainstorming`** — turn a vague idea into a validated design, one question at a time. Hand off to `planning-projects` when ready.
- **`planning-projects`** — produce a staged plan with explicit gates, dependencies, and rollback notes. Every task has a concrete way to know it's done.
- **`executing-plans`** — drive a plan to completion stage by stage. Won't move on until the current stage's evidence-of-done passes.
- **`evidence-first-investigation`** — when something looks wrong, gather evidence before proposing fixes. Useful for journalists, auditors, analysts, anyone who can't afford to act on a guess.
- **`review-document`** — holistic review of any reader-facing document (README, brief, memo, one-pager) against the 30-second rule. Never grow the document to fix it.
- **`peer-review-deliverable`** — review a deliverable in flight against a baseline (prior version, plan, template). Surfaces a Critical / Important / Suggestion triage. Pairs with `document-rewriter` for the actual edits.

### `vault-librarian` plugin

A thin companion to the upstream `obsidian-wiki` plugin. One slash command:

- **`/vault-librarian:bootstrap-vault`** — sets up a fresh Markdown notes vault from inside Claude Code (no shell snippets, works on macOS / Windows / Linux desktop). Asks which schema fits your work — researcher, writer/journalist, or generic — and writes `CLAUDE.md`, `log.md`, `Home.md`, and the small config file the upstream plugin reads.

After bootstrap, run upstream commands as documented: `/obsidian-wiki:ingest`, `/obsidian-wiki:ask`, `/obsidian-wiki:lint`.

### `delegation-agents` plugin

Three model-pinned worker subagents and one dispatching skill that let a parent session on Opus offload bulk reading, editing, and drafting to cheaper tiers:

- **`bulk-reader`** (Haiku) — bulk file reads, greps, link checks, frontmatter extraction. Read-only.
- **`document-rewriter`** (Sonnet) — rewrites existing documents to a spec. Read + Edit only, never creates or deletes files.
- **`draft-generator`** (Sonnet) — generates new documents from a brief (agendas, outlines, scaffolded pages, templated deliverables). No installs, no commits, no sending.
- **`dispatching-parallel-agents`** skill — fans out independent tasks from a `planning-projects` plan to the three agents above in a single concurrent batch, integrates results, propagates the dependency graph.

Pairs naturally with `process-skills/executing-plans`, which is the typical caller of the dispatching skill.

## Golden paths by persona

Three concrete walkthroughs. Steps marked **(optional)** require the vault stack from the install section above; skip them if you don't keep a notes vault and the rest of the path still works.

### Researcher / analyst

You have a multi-week investigation: a question, a folder of source PDFs, a small set of stakeholders waiting for a brief.

```text
1. /vault-librarian:bootstrap-vault            → (optional) pick "researcher" schema; vault at ~/dev/research-knowledge
2. (drop your PDFs into <vault>/raw/)          → (optional) source inbox
3. /obsidian-wiki:ingest raw/<each-source>     → (optional) files them under Sources/ with citations and frontmatter
4. brainstorming                               → "I want to investigate <question>" — produces a validated design
5. planning-projects                           → staged plan; preflight checks source access; stages produce Findings
6. executing-plans                             → drives the plan; dispatches parallel work through delegation-agents
7. evidence-first-investigation                → invoked when a finding doesn't add up; gathers evidence before you write the conclusion
8. peer-review-deliverable                     → review the brief against the plan before you send it
```

Without the vault, your sources live wherever you already keep them and the Findings live in `docs/` or your working folder — same skills, same discipline.

### Writer / journalist

You're working on a feature with multiple sources, a deadline, and an editor who will mark up the draft.

```text
1. /vault-librarian:bootstrap-vault            → (optional) pick "writer" schema; vault at ~/dev/notes
2. (drop interview transcripts, scans into <vault>/raw/)  → (optional)
3. /obsidian-wiki:ingest raw/<each-source>     → (optional) files them under Sources/ with on-record status
4. brainstorming                               → angle, scope, audience, embargo; produces the design doc
5. planning-projects                           → outline + per-section plan; tracks status per section
6. executing-plans                             → drives drafting; document-rewriter applies your editor's marks in parallel
7. /obsidian-wiki:ask "what did <source> say about X"  → (optional, vault-only) cited answers with on-record enforcement
8. peer-review-deliverable                     → before you file: vs. the outline, vs. last week's draft, vs. the brief
9. review-document                             → final 30-second-rule pass on the lede / nut graf
```

Without the vault, on-record status and citation tracking become *your* discipline rather than the plugin's, but every other step works the same way against a folder of source files.

### Project lead / consultant

You're running an engagement: a kickoff document, weekly status updates, deliverables on a timeline, stakeholders who need short briefs.

```text
1. /vault-librarian:bootstrap-vault            → (optional) pick "generic" schema; vault per engagement
2. brainstorming                               → kickoff doc — purpose, audience, success criteria, risks
3. planning-projects                           → engagement plan; risk-flagged stages; rollback notes for each
4. executing-plans                             → drives the plan week by week; dispatching-parallel-agents fans out
                                                  independent tasks (interviews, draft sections, vendor asks)
5. peer-review-deliverable                     → review every deliverable before circulation: vs. plan, vs. template, vs. prior version
6. review-document                             → 30-second-rule pass on the executive summary
7. evidence-first-investigation                → when a status flag goes red, before you propose remediation
```

The vault is most useful here for cross-engagement memory — patterns, gotchas, prior client briefs you reference across projects. For a one-off engagement, it's optional and you can skip step 1.

## How the pieces connect

```
brainstorming  →  planning-projects  →  executing-plans  →  (deliverable)
                                              │                    │
                                  dispatching-parallel-agents      │
                                       │       │       │           │
                                bulk-reader  document-  draft-     │
                                  (Haiku)    rewriter  generator   │
                                            (Sonnet)   (Sonnet)    │
                                                                   ↓
       evidence-first-investigation  ←──── (when something looks wrong) ───── peer-review-deliverable
                       ↑                                                            ↓
            review-document  (30-second-rule pass on the deliverable as a whole)   /
```

`vault-librarian` + upstream `obsidian-wiki` runs underneath as an **optional** notes layer if you want one. Without it, source files live wherever you already keep them; the rest of the pipeline is unchanged.

## Roadmap

Optional future work (deferred until basic-harness has real users):

- **`ever-learn` port** — gradual, reviewable improvement of skills / playbooks / templates from session evidence. Worth the effort only if someone maintains a living rule set and reports drift.

See [`docs/plans/2026-05-04-non-it-port.md`](../docs/plans/2026-05-04-non-it-port.md) in the parent repo for the staged port plan.
