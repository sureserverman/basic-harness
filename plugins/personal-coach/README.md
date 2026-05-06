# personal-coach

Personal-use plugin for `basic-harness`. Turns Claude Code into a structured companion for the parts of life that aren't shipping software:

- **Personal assistant that actually knows you** — a persistent profile that compounds across sessions instead of starting from zero every chat.
- **Reflective listener** — guided journaling and CBT-style reframing for personal development. Not therapy; a structured thinking partner with explicit limits.
- **Business mentor** — strategic-decision frameworks (OKR, Jobs-to-be-Done, Cynefin, decision journals) to think through hard calls before you commit.
- **Fintech legal-issue spotter** — surfaces the regulatory questions you should be asking your real lawyer about KYC/AML, payment licensing, data protection, and customer contracts. Issue-spotter, not advice.

This plugin is opinionated about scope. It will refuse to play "diagnose me" or "tell me what to do legally" — those are jobs for a licensed therapist and a licensed lawyer. It is good at being the thing in between: a structured, citation-grounded thinking partner that helps you arrive prepared.

## Install

```text
/plugin install personal-coach@basic-harness
```

Pairs especially well with the optional vault stack — see the top-level basic-harness README — because the **personal** vault schema gives Profile/Journal/Goals/Business/Legal/People/Decisions a real home on disk.

## Start here (new users)

After installing, run:

```text
/personal-coach:onboard
```

Five short phases, about five minutes total, in whatever language you write to it in (English / Russian / Spanish / German / French / Mandarin / Hindi / Arabic / etc. — the first message tells the skill which to use). You can skip anything, and your place is saved between phases so you can stop and resume later. By the end you'll have a saved profile, one tangible artifact (a reflection, a decision frame, news preferences, or a fintech regulatory cell), and a clear picture of what's available without being shown all of it at once.

## Skills

All eight skills follow the same shape as `process-skills`: a checklist, phased prompts, a hard handoff at the end, and primary-source citations for the methodology.

| Skill | Purpose |
|---|---|
| `onboarding` | Calm five-minute first-run flow: language detection → profile bootstrap → one tangible artifact → optional automation → wrap-up. Skip-friendly, save-and-resume between phases. Recommended starting point. |
| `personal-profile` | Build and maintain a persistent profile of the user — values, goals, voice, history, recurring stakeholders. The substrate the other skills read from. |
| `reflection-session` | Structured journaling and CBT-style reflection. Emotion labelling → thought record → cognitive reframe → committed action. |
| `business-mentoring` | Frame a strategic decision: pick a framework, work it, write a decision-journal entry that you can grade later. |
| `fintech-legal-triage` | Issue-spotter for fintech work: KYC/AML, PSD2, MiCA, GDPR, data localisation, customer-contract gotchas. Produces a question list for a real lawyer. |
| `morning-briefing` | Daily standup with yourself: yesterday's commits, today's intentions, blockers, one reflection prompt. |
| `news-preferences` | Capture the user's news interests — topics, sources, exclusions, language, length, style, cadence, open-questions watchlist. The substrate `news-digest` reads from. |
| `news-digest` | Personalized daily news digest against the saved preferences. Filters strictly, never fabricates, cites every item. Refuses to run without preferences — generic headlines aren't the point. |

Every existing skill also has a **`## In Cowork (connector-aware enrichment)`** section documenting what it gains when Calendar / Gmail / Drive / DocuSign connectors are granted. Connectors are always optional — the skills work the same in Code or in Cowork-without-connectors.

The plugin is **language-agnostic**. Every skill that produces user-facing prompts honors the language preference set during onboarding (or detected from your first message). The reference text in each `SKILL.md` is in English — that's what Claude reads — but your conversation runs in whatever language you write in.

## Subagents

Model-pinned worker agents you can dispatch directly when the corresponding skill is overkill:

| Agent | Model | Role |
|---|---|---|
| `psychologist-listener` | Sonnet | Reflective listening only — mirrors, names emotions, asks one question at a time. Refuses diagnosis. |
| `business-mentor` | Sonnet | Strategic-decision sparring partner. Pushes back on premises. Names the framework being applied. |
| `fintech-legal-analyst` | Opus | Careful regulatory analysis. Asks for jurisdiction first. Outputs an issue list, never a verdict. |

## Slash commands

Four commands wire the plugin's recurring rhythms and onboarding flow:

| Command | What it does |
|---|---|
| `/personal-coach:onboard` | Start (or resume / re-run) the five-minute onboarding flow. Detects whether you're a first-time, partial, or returning user and routes accordingly. |
| `/personal-coach:setup-morning-briefing` | Schedule a daily morning briefing. Optionally bundles `news-digest` ahead of the briefing in the same scheduled task. |
| `/personal-coach:setup-decision-grading` | Schedule a weekly scan that surfaces decisions whose 90-day grading deadline has landed. Surfaces only — never auto-grades. |
| `/personal-coach:setup-news-digest` | Schedule the daily news digest. Gates on `news-preferences` existing first (refuses to schedule a digest with no preferences). |

The setup commands detect whether you're in Cowork (uses `/schedule`) or Code (prints a `crontab -e` snippet) and adapt.

## Cowork Routines (cloud-tier automation)

Optional. The `routines/` directory ships four copy-paste templates for Anthropic Cowork **Routines** — cloud-hosted automations that run with the laptop closed. **Read `routines/README.md` first** — Routines execute in Anthropic's cloud, which changes the privacy posture vs the desktop Scheduled Task path.

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
