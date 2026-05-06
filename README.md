# basic-harness

[![Latest release](https://img.shields.io/github/v/release/sureserverman/basic-harness?label=latest&color=blue)](https://github.com/sureserverman/basic-harness/releases/latest)

Starter scaffolding for **Claude Cowork**. Process discipline, knowledge management, delegation patterns, and a personal-companion track — for any kind of structured work, not only coding.

**Cowork-first.** The plugins target Cowork on macOS / Windows desktop. Distributed as zips attached to GitHub releases, installed via Cowork's Customize → Browse plugins → upload custom plugin file UI. No external infrastructure required.

## Who it's for

Four personas drive the design:

- **Researchers and analysts** — read, take notes, write briefs, run multi-week investigations.
- **Writers and journalists** — long-form work with sources, drafting, editing.
- **Project leads and consultants** — engagements with deliverables, planning, stakeholder updates.
- **Personal users / fintech operators** — Claude as personal assistant, business mentor, reflective listener, and fintech legal-issue spotter, with a profile that compounds across sessions instead of starting from zero every chat.

If you write software all day, the parent project [`coder-plugins`](https://github.com/sureserverman/coder-plugins) is the better starting point.

## Install

1. Open the [latest release](https://github.com/sureserverman/basic-harness/releases/latest) on GitHub. Each release attaches one zip per plugin.
2. Download the zips you want — at minimum `welcome-vX.Y.Z.zip`. For the personal-companion side, also `personal-coach-vX.Y.Z.zip`. For the research / writing / project track, add `process-skills-vX.Y.Z.zip`, `delegation-agents-vX.Y.Z.zip`, and (optionally) `vault-librarian-vX.Y.Z.zip`.
3. In Cowork: click **Customize** in the sidebar → **Browse plugins** → **upload custom plugin file** → select a zip → repeat for each.
4. Restart Cowork (`Cmd+Q` and reopen) so the skills register.
5. In any Cowork chat, ask "show me what you do" — or type `/welcome:tour` — and the orientation flow starts.

### Optional: notes vault

If you want a local Markdown notes vault to ingest sources into and query from inside Cowork, also install `vault-librarian` (zip from this repo's release) **and** the upstream `obsidian-wiki` plugin (zip from [sureserverman/obsidian-wiki-plugin releases](https://github.com/sureserverman/obsidian-wiki-plugin/releases)) — same upload-zip flow.

Then run `/vault-librarian:bootstrap-vault`. The vault is **purely local** — a directory of Markdown files at a path you pick (default `~/dev/knowledge`). No network, no cloud, no account. If you want sync across machines later, layer on Obsidian Sync, Syncthing, iCloud Drive, or whatever you already use — none of it is built into basic-harness.

If you don't want a notes vault at all, skip this section. Everything else still works.

## Tours

Three calm read-only tour commands. None write to disk; none invoke other skills.

| Command | Length | What it covers |
|---|---|---|
| `/welcome:tour` | ~2 min | The marketplace map. What basic-harness is, the four personas, the install flow. Points at the right specialized tour. |
| `/personal-coach:tour` | ~2 min | The personal-companion track in depth. Skills, subagents, setup commands, Cowork routines, where files live, hard limits. |
| `/process-skills:tour` | ~2 min | The research / writing / project track in depth. The brainstorm-plan-execute pipeline, delegation agents, optional vault stack. |

All three run in whatever language you write to them in. None of them push a follow-up tour without your explicit "yes". You can stop after the marketplace map and dive in, or read both specialized tours, or skip the tours entirely if you already know what you want.

## What's inside

### `welcome` plugin

The marketplace orientation. One read-only slash command, `/welcome:tour`, which gives a calm two-minute map of basic-harness and points at the right specialized tour for your work. Doesn't bootstrap anything, doesn't invoke skills. Recommended first install for any new user.

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

### `personal-coach` plugin

Optional personal-use companion. Five skills + three model-pinned subagents that turn Claude Code into a structured personal assistant / business mentor / reflective listener / fintech legal-issue spotter, with a persistent profile of you that compounds across sessions.

Skills:

- **`personal-profile`** — builds and maintains the persistent profile (values, goals, voice, stakeholders, sensitivities, do-not-do list). The substrate every other skill in this plugin reads from. Never edits silently — every write is shown and confirmed.
- **`reflection-session`** — structured CBT-style reflection: emotion labelling → thought record → distortion check → reframe → committed action. Safety-gated: redirects to a real human on any crisis content.
- **`business-mentoring`** — strategic-decision sparring partner. Picks an appropriate framework (JTBD, Cynefin, OKR, ICE/RICE, pre-mortem, 2x2), works it, and writes a decision-journal entry that can be graded 90 days later.
- **`fintech-legal-triage`** — issue-spotter for fintech work in EU / UK / US / RU / SG / UAE: KYC/AML, payment licensing, MiCA, GDPR, customer T&Cs, marketing claims. Outputs an issue list with named regulations and questions for a real lawyer. Every output ends with "take this to a licensed lawyer in <jurisdiction>".
- **`morning-briefing`** — five-field daily standup with yourself; surfaces decisions whose 90-day grading deadline has arrived.

Subagents (read by the parent session through normal Claude Code dispatch):

- **`psychologist-listener`** (Sonnet) — reflective listening only. One question per turn, no advice, no diagnosis.
- **`business-mentor`** (Sonnet) — strategic-decision sparring partner; pushes back on premises; produces decision-journal entries.
- **`fintech-legal-analyst`** (Opus) — careful regulatory issue-spotter; jurisdiction-first, issue-list-only, take-to-counsel block on every output.

The plugin pairs naturally with the `personal` vault schema in `vault-librarian` (option D in `bootstrap-vault`) — that's where Profile, Journal, Goals, Decisions, Legal, People all live on disk.

**Hard limits.** The plugin will refuse to: diagnose anything medical or psychological, give legal advice, surrogate a business decision the user is trying to make, or send any of the user's content anywhere off-machine. It is a structured thinking partner, not a therapist or a lawyer.

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

### Personal user / fintech operator

You're using Claude as a personal companion: you want it to remember who you are across sessions, help you reflect, spar on hard business calls, flag fintech regulatory questions before they bite, and have a personalized news digest land every morning.

```text
0. /welcome:tour                               → (optional, ~2 min) marketplace map. Helps you confirm personal-coach is the right track.
1. /vault-librarian:bootstrap-vault            → (optional but recommended) pick "personal" schema (D); vault at ~/dev/personal
2. (install personal-coach zip)                → download from releases, upload via Cowork's Customize → Browse plugins
3. /personal-coach:onboard                     → ★ start here. Five short phases, ~5 minutes, in your preferred language. Bootstraps profile, produces one tangible artifact, leaves you with a calm map of what's next.
4. news-preferences                            → (if onboarding didn't set them) topics, sources, exclusions, cadence
5. /personal-coach:setup-morning-briefing      → wire daily briefing into Cowork's Scheduled Tasks; optionally bundles news-digest
6. /personal-coach:setup-news-digest           → wire the daily news digest separately, if not bundled with briefing
7. /personal-coach:setup-decision-grading      → wire the weekly grading scan
8. morning-briefing                            → daily five-field standup with yourself; surfaces decisions due for grading
9. reflection-session                          → structured CBT-style journaling on whatever's stuck; safety-gated
10. business-mentoring                         → frame a hard decision; pick a framework; record it for 90-day grading
11. fintech-legal-triage                       → issue-list for any fintech feature/contract/partner change before counsel call
12. news-digest                                → run the digest manually whenever, or let scheduled tasks fire it
13. (subagent) psychologist-listener           → reflective-listening-only worker; called automatically when reflection-session needs the listening seat
14. (subagent) business-mentor                 → strategic-decision worker; called automatically when business-mentoring needs deep analysis
15. (subagent) fintech-legal-analyst           → Opus-tier regulatory issue-spotter; called automatically when fintech-legal-triage hits the cell-matching phase
```

The onboarding flow is **language-aware**: write to it in Russian / English / Spanish / German / French / Mandarin / Hindi / Arabic / etc. and the entire flow runs in that language. The plugin's reference text is in English (read by Claude); the user-facing conversation is yours.

Without the vault, profile / journal / decisions / legal logs / news digests all live under `~/.claude/` instead — same skills, same flow, just less queryable later. **None of this leaves your machine** — except where you explicitly enable Cowork Routines, which run in Anthropic's cloud (see `plugins/personal-coach/routines/README.md` for the privacy tradeoffs). Profile and reflection content are never routinable by design.

In Cowork specifically, the personal-coach plugin gains:

- **Calendar / Gmail / Drive / DocuSign connectors** that enrich the existing skills — never required, never silent. Each skill's `## In Cowork (connector-aware enrichment)` section documents the specifics.
- **Scheduled Tasks** for the morning-briefing / news-digest / decision-grading rhythms, wired up by the three setup commands above.
- **Routines** (cloud, optional) for the same rhythms when you want them to fire with the laptop closed — privacy-tradeoff documented per template.

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
