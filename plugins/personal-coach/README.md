# personal-coach

Personal-use plugin for `basic-harness`. Turns Claude Code into a structured companion for the parts of life that aren't shipping software:

- **Personal assistant that actually knows you** — a persistent profile that compounds across sessions instead of starting from zero every chat.
- **Reflective listener** — guided journaling and CBT-style reframing for personal development. Not therapy; a structured thinking partner with explicit limits.
- **Business mentor** — strategic-decision frameworks (OKR, Jobs-to-be-Done, Cynefin, decision journals) to think through hard calls before you commit.
- **Fintech legal-issue spotter** — surfaces the regulatory questions you should be asking your real lawyer about KYC/AML, payment licensing, data protection, and customer contracts. Issue-spotter, not advice.

This plugin is opinionated about scope. It will refuse to play "diagnose me" or "tell me what to do legally" — those are jobs for a licensed therapist and a licensed lawyer. It is good at being the thing in between: a structured, citation-grounded thinking partner that helps you arrive prepared.

## Install

See the [top-level basic-harness README](../../README.md#install) for the canonical install flow (Cowork → Customize → upload zip from this repo's GitHub releases).

**No separate vault setup required.** The first time onboarding (or any skill) needs to save something durable, personal-coach asks once where to put it: a recommended folder at `~/Notes/personal-coach`, a custom path, or fall back to `~/.claude/personal-coach/` for users who don't want a vault. The required CLAUDE.md schema, `log.md`, `Home.md`, and the `obsidian-wiki` config file are all written by personal-coach itself — `vault-librarian` is **no longer a prerequisite**.

`vault-librarian` is still useful for users who want to set up a richer vault (writer / researcher schemas, or a vault to share with other plugins) before personal-coach asks. Install it from the same release page if that fits.

## Start here (new users)

After installing, run:

```text
/personal-coach:onboard
```

One short opening — three sentences plus one open question ("what's on your mind right now?") — in whatever language you write to it in (English / Russian / Spanish / German / French / Mandarin / Hindi / Arabic / etc.; the first message picks the language). Whatever you say next gets engaged directly, on a real situation of yours: a decision you're stuck on, something on your mind to reflect on, a fintech regulatory question to triage, or daily news preferences. One tangible artifact lands on disk in turn 2 or 3. No persona quiz, no five-step walkthrough, no feature list. If you'd rather just look around first, ask — `/welcome:tour` and `/personal-coach:tour` exist for that.

## Skills

All eight skills follow the same shape as `process-skills`: a checklist, phased prompts, a hard handoff at the end, and primary-source citations for the methodology.

| Skill | Purpose |
|---|---|
| `onboarding` | Value-first first-run flow. One short opening, then engages the real situation the user names — decision frame / reflection / fintech triage / news digest — producing one tangible artifact in turn 2-3. No phases, no feature list, no persona quiz. Recommended starting point. |
| `personal-profile` | Build and maintain a persistent profile of the user — values, goals, voice, history, recurring stakeholders. The substrate the other skills read from. |
| `reflection-session` | Structured journaling and CBT-style reflection. Emotion labelling → thought record → cognitive reframe → committed action. |
| `business-mentoring` | Frame a strategic decision: pick a framework, work it, write a decision-journal entry that you can grade later. |
| `fintech-legal-triage` | Issue-spotter for fintech work: KYC/AML, PSD2, MiCA, GDPR, data localisation, customer-contract gotchas. Produces a question list for a real lawyer. |
| `morning-briefing` | Daily standup with yourself: yesterday's commits, today's intentions, blockers, one reflection prompt. |
| `news-preferences` | Capture the user's news interests — topics, sources, exclusions, language, length, style, cadence, open-questions watchlist. The substrate `news-digest` reads from. |
| `news-digest` | Personalized daily news digest against the saved preferences. Filters strictly, never fabricates, cites every item. Refuses to run without preferences — generic headlines aren't the point. |

Every existing skill also has a **`## In Cowork (connector-aware enrichment)`** section documenting what it gains when Calendar / Gmail / Drive / DocuSign connectors are granted. Connectors are always optional — the skills work the same with or without them.

The plugin is **language-agnostic**. Every skill that produces user-facing prompts honors the language preference set during onboarding (or detected from your first message). The reference text in each `SKILL.md` is in English — that's what Claude reads — but your conversation runs in whatever language you write in.

## Subagents

Model-pinned worker agents you can dispatch directly when the corresponding skill is overkill:

| Agent | Model | Role |
|---|---|---|
| `psychologist-listener` | Sonnet | Reflective listening only — mirrors, names emotions, asks one question at a time. Refuses diagnosis. |
| `business-mentor` | Sonnet | Strategic-decision sparring partner. Pushes back on premises. Names the framework being applied. |
| `fintech-legal-analyst` | Opus | Careful regulatory analysis. Asks for jurisdiction first. Outputs an issue list, never a verdict. |

## Slash commands

Five commands cover the plugin's onboarding, depth tour, and recurring rhythms:

| Command | What it does |
|---|---|
| `/personal-coach:onboard` | Start (or resume / re-run) the value-first onboarding. Detects whether you're a first-time, partial, or returning user and routes accordingly. |
| `/personal-coach:tour` | Two-minute read-only walkthrough of the personal-coach ecosystem — skills, subagents, setup commands, routines, hard limits. For when you want the wider map without bootstrapping anything. |
| `/personal-coach:setup-morning-briefing` | Schedule a daily morning briefing. Optionally bundles `news-digest` ahead of the briefing in the same scheduled task. |
| `/personal-coach:setup-decision-grading` | Schedule a weekly scan that surfaces decisions whose 90-day grading deadline has landed. Surfaces only — never auto-grades. |
| `/personal-coach:setup-news-digest` | Schedule the daily news digest. Gates on `news-preferences` existing first (refuses to schedule a digest with no preferences). |

The setup commands walk you through Cowork's `/schedule` UI (or sidebar → Scheduled → + New task) with a pre-filled prompt.

## Cowork Routines (cloud-tier automation)

Optional. Four copy-paste templates for Anthropic Cowork **Routines** — cloud-hosted automations that run with the laptop closed — live at `docs/personal-coach-routines/` in the basic-harness repo (not inside the plugin). **Read `docs/personal-coach-routines/README.md` first** — Routines execute in Anthropic's cloud, which changes the privacy posture vs the desktop Scheduled Task path.

| Template | Trigger |
|---|---|
| `news-digest-routine.md` | Daily, at user's preferred time. Privacy: acceptable. |
| `morning-briefing-routine.md` | Weekday mornings. Scaffold-only — five fields stay as prompts. Privacy: mixed. |
| `decision-grading-routine.md` | Weekly. Scan-only, never auto-grades. Privacy: moderate. |
| `legal-triage-on-drive-routine.md` | Event-driven; new file in watched Drive folder. ⚠ Read the privacy header — sensitive contracts should not be triaged via Routine. |

`reflection-session` and `personal-profile` are deliberately not routinable. Reflection is the most sensitive material in the plugin; profile edits should be confirmed in real time.

## Hard limits

- **No medical, psychiatric, or psychological diagnosis.** Reflective listening, never assessment. Crisis content triggers a stop-and-redirect to local emergency lines.
- **No legal advice.** Issue-spotting and question-formulation only. Every fintech-legal output ends with "take this to a licensed lawyer in <jurisdiction>".
- **No silent edits to the personal profile.** The user sees and confirms every profile write.
- **No data exfiltration.** Profile and journal entries stay on disk under the user's vault path. Nothing is sent anywhere.

## Sources and rationale

The skills cite their methodology so the shape isn't arbitrary. See each `SKILL.md`'s "Sources" section. Anchor references:

- CBT thought records — Aaron T. Beck, *Cognitive Therapy of Depression* (1979); Judith Beck, *Cognitive Behavior Therapy: Basics and Beyond* (3rd ed., 2020).
- Decision journals — Annie Duke, *Thinking in Bets* (2018); Daniel Kahneman, *Thinking, Fast and Slow* (2011).
- Jobs-to-be-Done — Clayton Christensen, *Competing Against Luck* (2016).
- Cynefin framework — David Snowden & Mary Boone, "A Leader's Framework for Decision Making" (Harvard Business Review, 2007).
- Stoic journaling — Marcus Aurelius, *Meditations*; Ryan Holiday, *The Daily Stoic* (2016).
- AML/KYC fundamentals — FATF *International Standards on Combating Money Laundering* (40 Recommendations, latest revision).
- EU payments and crypto — PSD2 (Directive (EU) 2015/2366); MiCA (Regulation (EU) 2023/1114).
- Data protection — GDPR (Regulation (EU) 2016/679).
