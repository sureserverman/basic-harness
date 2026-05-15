# personal-coach

Personal-use plugin for `basic-harness`. Turns Claude Code into a structured companion for the parts of life that aren't shipping software:

- **Personal assistant that actually knows you** — a persistent profile that compounds across sessions instead of starting from zero every chat.
- **Reflective listener** — guided journaling and CBT-style reframing for personal development. Not therapy; a structured thinking partner with explicit limits.
- **Business mentor** — strategic-decision frameworks (OKR, Jobs-to-be-Done, Cynefin, decision journals) to think through hard calls before you commit.

This plugin is opinionated about scope. It will refuse to play "diagnose me" — that's a licensed therapist's job. It is good at being the thing in between: a structured, citation-grounded thinking partner that helps you arrive prepared.

**Companion plugins.** Two features that used to live in personal-coach now ship as their own plugins in the same basic-harness release:

- [`news-digest`](../news-digest/README.md) — personalized daily news digest. Topics, sources, exclusions, watchlist; refuses generic headlines. Install separately if you want it.
- [`fintech-legal-advisor`](../fintech-legal-advisor/README.md) — fintech regulatory issue-spotter (KYC/AML/MiCA/GDPR/customer T&Cs). Built around two surfaces: an interactive skill, and a Cowork Routine that watches a Drive folder of inbound contracts. Install separately if you want it.

The two companions integrate softly with personal-coach when both are installed (they share the vault path, and read the personal-profile's business-context section when present), but each works fine on its own.

## Install

See the [top-level basic-harness README](../../README.md#install) for the canonical install flow (Cowork → Customize → upload zip from this repo's GitHub releases).

**No separate vault setup required.** The first time onboarding (or any skill) needs to save something durable, personal-coach asks once where to put it: a recommended folder at `~/Notes/personal-coach`, a custom path, or fall back to `~/.claude/personal-coach/` for users who don't want a vault. The required CLAUDE.md schema, `log.md`, `Home.md`, and the `obsidian-wiki` config file are all written by personal-coach itself — `vault-librarian` is **no longer a prerequisite**.

`vault-librarian` is still useful for users who want to set up a richer vault (writer / researcher schemas, or a vault to share with other plugins) before personal-coach asks. Install it from the same release page if that fits.

## Start here (new users)

After installing, run:

```text
/personal-coach:onboard
```

One short opening — two sentences plus one open question ("what's on your mind right now?") — in whatever language you write to it in (English / Russian / Spanish / German / French / Mandarin / Hindi / Arabic / etc.; the first message picks the language). Whatever you say next gets engaged directly, on a real situation of yours: a decision you're stuck on, or something on your mind to reflect on. If you mention news or fintech regulation, onboarding hands you off to the companion plugin that handles it. One tangible artifact lands on disk in turn 2 or 3. No persona quiz, no five-step walkthrough, no feature list. If you'd rather just look around first, ask — `/welcome:tour` and `/personal-coach:tour` exist for that.

## Vault integration (v0.4.0)

As of basic-harness v0.7.0, `reflection-session` and `business-mentoring` are **vault-citizens** — they use the shared `vault-companion` surface from `vault-librarian`:

- **Phase 0.5 — Vault Recall** scans the vault for prior entries on the same theme before each session. Reflection-session asks before surfacing on tender topics ("Want me to bring forward what you wrote then, or start fresh?"). Business-mentoring surfaces the **grade** of related prior decisions when available — the only feedback loop the decision journal has.
- **Save** delegates to `vault-companion-append` with `category=Journal` (reflection) or `category=Decisions` (business-mentoring). Stakeholder / company names are wrapped in `[[wikilink]]` form so `[[Marcus]]` / `[[Project Athena]]` accumulate backlinks across the vault.
- **Bootstrap** happens once, on the first save of the first session. Onboarding Step 3 delegates to `vault-companion-ensure`, which asks one question and writes the personal-schema vault skeleton. The v0.4.0 "auto-bootstrap on first save" promise is preserved; the implementation now lives in vault-librarian so all four substantive vault-citizen skills (this plugin's two, plus `news-digest`, plus `fintech-legal-triage`) share one surface.
- **Fallback.** If you've declined a vault, reflection saves to `~/.claude/journal/`, decisions to `~/.claude/decisions/`. Same skill quality, no cross-linking.

## Skills

All five skills follow the same shape as `process-skills`: a checklist, phased prompts, a hard handoff at the end, and primary-source citations for the methodology.

| Skill | Purpose |
|---|---|
| `onboarding` | Value-first first-run flow. One short opening, then engages the real situation the user names — decision frame or reflection — producing one tangible artifact in turn 2-3. Hands off to companion plugins for news or fintech triage. No phases, no feature list, no persona quiz. Recommended starting point. |
| `personal-profile` | Build and maintain a persistent profile of the user — values, goals, voice, history, recurring stakeholders. The substrate the other skills read from. |
| `reflection-session` | Structured journaling and CBT-style reflection. Emotion labelling → thought record → cognitive reframe → committed action. |
| `business-mentoring` | Frame a strategic decision: pick a framework, work it, write a decision-journal entry that you can grade later. |
| `morning-briefing` | Daily standup with yourself: yesterday's commits, today's intentions, blockers, one reflection prompt. Optionally folds in the news-digest output if the companion plugin is installed. |

Every existing skill also has a **`## In Cowork (connector-aware enrichment)`** section documenting what it gains when Calendar / Gmail / Drive / DocuSign connectors are granted. Connectors are always optional — the skills work the same with or without them.

The plugin is **language-agnostic**. Every skill that produces user-facing prompts honors the language preference set during onboarding (or detected from your first message). The reference text in each `SKILL.md` is in English — that's what Claude reads — but your conversation runs in whatever language you write in.

## Subagents

Model-pinned worker agents you can dispatch directly when the corresponding skill is overkill:

| Agent | Model | Role |
|---|---|---|
| `psychologist-listener` | Sonnet | Reflective listening only — mirrors, names emotions, asks one question at a time. Refuses diagnosis. |
| `business-mentor` | Sonnet | Strategic-decision sparring partner. Pushes back on premises. Names the framework being applied. |

The Opus-pinned `fintech-legal-analyst` worker now lives in the `fintech-legal-advisor` companion plugin. Install that plugin if you want it.

## Slash commands

Four commands cover the plugin's onboarding, depth tour, and recurring rhythms:

| Command | What it does |
|---|---|
| `/personal-coach:onboard` | Start (or resume / re-run) the value-first onboarding. Detects whether you're a first-time, partial, or returning user and routes accordingly. |
| `/personal-coach:tour` | Two-minute read-only walkthrough of the personal-coach ecosystem — skills, subagents, setup commands, routines, hard limits. For when you want the wider map without bootstrapping anything. |
| `/personal-coach:setup-morning-briefing` | Schedule a daily morning briefing. Optionally bundles the news-digest (if the companion plugin is installed) ahead of the briefing in the same scheduled task. |
| `/personal-coach:setup-decision-grading` | Schedule a weekly scan that surfaces decisions whose 90-day grading deadline has landed. Surfaces only — never auto-grades. |

The setup commands walk you through Cowork's `/schedule` UI (or sidebar → Scheduled → + New task) with a pre-filled prompt.

The news-digest setup command and the fintech-legal-advisor setup command live in their respective companion plugins (`/news-digest:setup-news-digest`, `/fintech-legal-advisor:setup-legal-triage-routine`).

## Cowork Routines (cloud-tier automation)

Optional. Two copy-paste templates for Anthropic Cowork **Routines** — cloud-hosted automations that run with the laptop closed — live at `docs/personal-coach-routines/` in the basic-harness repo (not inside the plugin). **Read `docs/personal-coach-routines/README.md` first** — Routines execute in Anthropic's cloud, which changes the privacy posture vs the desktop Scheduled Task path.

| Template | Trigger |
|---|---|
| `morning-briefing-routine.md` | Weekday mornings. Scaffold-only — five fields stay as prompts. Privacy: mixed. |
| `decision-grading-routine.md` | Weekly. Scan-only, never auto-grades. Privacy: moderate. |

`reflection-session` and `personal-profile` are deliberately not routinable. Reflection is the most sensitive material in the plugin; profile edits should be confirmed in real time.

The news-digest and legal-triage-on-Drive Routine templates now ship with their respective companion plugins (`plugins/news-digest/docs/routines/` and `plugins/fintech-legal-advisor/docs/routines/`).

## Hard limits

- **No medical, psychiatric, or psychological diagnosis.** Reflective listening, never assessment. Crisis content triggers a stop-and-redirect to local emergency lines.
- **No silent edits to the personal profile.** The user sees and confirms every profile write.
- **No data exfiltration.** Profile and journal entries stay on disk under the user's vault path. Nothing is sent anywhere from this plugin's skills.

(The `fintech-legal-advisor` companion plugin adds its own hard limit: no legal advice, only issue lists, with a take-to-counsel block on every output.)

## Sources and rationale

The skills cite their methodology so the shape isn't arbitrary. See each `SKILL.md`'s "Sources" section. Anchor references:

- CBT thought records — Aaron T. Beck, *Cognitive Therapy of Depression* (1979); Judith Beck, *Cognitive Behavior Therapy: Basics and Beyond* (3rd ed., 2020).
- Decision journals — Annie Duke, *Thinking in Bets* (2018); Daniel Kahneman, *Thinking, Fast and Slow* (2011).
- Jobs-to-be-Done — Clayton Christensen, *Competing Against Luck* (2016).
- Cynefin framework — David Snowden & Mary Boone, "A Leader's Framework for Decision Making" (Harvard Business Review, 2007).
- Stoic journaling — Marcus Aurelius, *Meditations*; Ryan Holiday, *The Daily Stoic* (2016).
