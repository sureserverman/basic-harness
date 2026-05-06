---
description: Two-minute calm read-only walkthrough of the research / writing / project-management track in basic-harness — process-skills (the six process skills), delegation-agents (the three model-pinned workers), and the optional vault stack (vault-librarian + obsidian-wiki). Targets the researcher / writer / project-lead persona. Read-only — no plans drafted, no skills invoked.
---

# Tour (process-skills + delegation-agents + vault-librarian)

A two-minute walkthrough of the research / writing / project track. This is the depth tour for someone who came in via `/welcome:tour` and picked persona 1, 2, or 3 (researcher / writer / project lead). **Read-only.**

## Step 0 — Detect language

Same pattern as the other tours.

## Step 1 — The shape (one paragraph)

Equivalent of:

> The research / writing / project track is built around three plugins that compose: **`process-skills`** gives you the discipline (brainstorm → plan → execute → review), **`delegation-agents`** gives you Sonnet/Haiku-tier workers that handle bulk reading, drafting, and editing in parallel, and **`vault-librarian`** (optional) lets you bootstrap a Markdown notes vault with persona-tuned schemas. None of it requires the others — but they snap together cleanly.

## Step 2 — The pipeline (one disclosure level)

> The intended flow is sequential, with feedback loops:
>
> ```
> brainstorming  →  planning-projects  →  executing-plans  →  (your deliverable)
>                                                  │                  │
>                                  dispatching-parallel-agents         │
>                                       │       │       │              │
>                              bulk-reader  document-  draft-          │
>                                (Haiku)    rewriter  generator        │
>                                          (Sonnet)   (Sonnet)         │
>                                                                      ↓
>                                       evidence-first-investigation ←──┘
>                                              ↑
>                                  (when something looks wrong)
>                                              ↓
>                                     review-document
>                                     peer-review-deliverable
> ```
>
> You don't always run all of them. A small piece of work might just be `review-document`. A multi-week investigation runs the whole pipeline.

## Step 3 — Process skills (each in one line)

> Six skills in `process-skills`. Domain-agnostic — the same shape works for a research paper, a campaign brief, a lesson plan, or a non-trivial code change:
>
> - **`brainstorming`** — turn a vague idea into a validated design, one question at a time. Always the first step before anything substantial.
> - **`planning-projects`** — produce a staged plan with explicit gates and evidence-of-done per stage.
> - **`executing-plans`** — drive the plan stage by stage; won't move on until the current stage's evidence-of-done passes.
> - **`evidence-first-investigation`** — when something looks wrong, gather evidence before proposing fixes. For journalists, auditors, analysts.
> - **`review-document`** — holistic review of any reader-facing document against the 30-second rule. Never grow the document to fix it.
> - **`peer-review-deliverable`** — review a deliverable in flight against a baseline (prior version, plan, template). Critical / Important / Suggestion triage.

## Step 4 — Delegation agents (each in one line)

> Three model-pinned worker subagents. Get dispatched automatically by the dispatching-parallel-agents skill, or you can invoke directly when you know which one fits:
>
> - **`bulk-reader`** (Haiku) — bulk file reads, greps, link checks, frontmatter extraction. Read-only.
> - **`document-rewriter`** (Sonnet) — rewrites existing documents to a spec. Read + Edit only — never creates or deletes files.
> - **`draft-generator`** (Sonnet) — generates new documents from a brief. No installs, no commits, no sending.
>
> Plus the dispatching skill itself: **`dispatching-parallel-agents`** fans out independent tasks from a planning-projects plan to the three agents in a single concurrent batch and integrates the results.

## Step 5 — Optional vault stack (one paragraph)

> If you want a local Markdown notes vault that ingests sources and is queryable from inside Claude Code, run `/vault-librarian:bootstrap-vault`. It picks one of four schemas:
>
> - **researcher / analyst** — Sources / Findings / Briefs / People / Methods / Questions.
> - **writer / journalist** — Pieces / Sources / People / Places / Topics / Pitches.
> - **generic** — Architecture / Patterns / Gotchas / Platforms / Projects / Technologies.
> - **personal** — Profile / Journal / Goals / Business / Decisions / Legal / People / Reading. (For personal-coach.)
>
> The vault itself is operated by the upstream `obsidian-wiki` plugin (separate marketplace). The `vault-librarian` plugin in basic-harness only bootstraps the schema and config so the upstream plugin can find the vault.

If the user isn't going to keep a notes vault, **skip Step 5 entirely** — the rest of the track works without one.

## Step 6 — Persona-specific golden paths (only if user asked or seems unsure)

> If you're a **researcher / analyst**: bootstrap the vault with the researcher schema, drop your sources into `raw/`, ingest them, then run `brainstorming → planning-projects → executing-plans` for the investigation itself.
>
> If you're a **writer / journalist**: same vault step with the writer schema. Editing flow is `executing-plans` driving the draft, with `document-rewriter` applying your editor's marks in parallel via `dispatching-parallel-agents`.
>
> If you're a **project lead / consultant**: same brainstorm → plan → execute discipline, with `dispatching-parallel-agents` fanning out interviews / draft sections / vendor asks. `peer-review-deliverable` before every circulation.

## Step 7 — Where files live

> Without a vault, source files and deliverables live wherever you already keep them; plans live in `docs/plans/` (the brainstorming + planning skills' default).
>
> With a vault, sources land under `<vault>/raw/` (immutable inbox), then ingest moves them into the right category. Deliverables can live in `<vault>/Briefs/` (researcher), `<vault>/Pieces/` (writer), or wherever you keep them — the skills don't enforce.

## Step 8 — Closing

> That's the track. A few starting points:
>
> - Have a piece of work in mind? Just describe it; `brainstorming` will pick up.
> - Want a notes vault? `/vault-librarian:bootstrap-vault`.
> - For the personal companion track (assistant / reflection / business mentor / fintech legal triage), see `/personal-coach:tour`.
> - For the marketplace-wide map, see `/welcome:tour`.
>
> Done.

Stop. Do not chain.

## What this command will NOT do

- **Will not write to disk.** Read-only.
- **Will not invoke `brainstorming`, `planning-projects`, or any other named skill.** That's the user's call.
- **Will not run `/vault-librarian:bootstrap-vault`** even if the user says "the vault sounds useful". Tell them how to invoke it; let them decide.
- **Will not run more than ~2 minutes.** Steps 5 and 6 are on-request expansion points.

## Failure modes

- **User asks for the personal track halfway through** — break out, hand off to `/personal-coach:tour`. Don't insist on finishing this tour.
- **User wants to dive into a specific skill** — answer in 2-3 sentences, point at the skill's name, stop the tour. Real work beats orientation every time.
