---
description: Two-minute calm read-only walkthrough of the personal-coach plugin — what each skill is for, what each subagent does, what the routines and setup commands are, where files land. For users who've already done /personal-coach:onboard (or want to skip onboarding) and want the wider map. Read-only — no profile writes, no scheduling, no skill invocations.
---

# Tour (personal-coach)

A two-minute walkthrough of the personal-coach ecosystem. This is the depth tour for a user who's seen the marketplace map (`/welcome:tour`) or finished onboarding and wants to know what else is around. **Read-only.**

## Step 0 — Detect language

Same as `/personal-coach:onboard` and `/welcome:tour`. Detect from invocation; ask in three languages if ambiguous.

## Step 1 — The shape (one paragraph)

Equivalent of:

> `personal-coach` is shaped around four roles Claude can play for you, each grounded in primary-source methodology:
>
> - **Personal assistant** that remembers who you are across sessions.
> - **Reflective listener** for journaling and CBT-style reframing — not therapy, with explicit limits.
> - **Business mentor** for hard decisions, with frameworks like JTBD / Cynefin / pre-mortem and a decision journal that gets graded 90 days later.
> - **Fintech legal-issue spotter** for KYC/AML, MiCA, GDPR, payment licensing — issue lists for your real lawyer, never advice.
>
> Plus a daily rhythm: morning briefing and a personalized news digest tailored to topics you actually care about.

## Step 2 — Skills (one disclosure level — short labels)

> Eight skills. You don't memorize these — they pick themselves up from what you say. Just so you know what's around:
>
> - **`onboarding`** — the value-first first-run flow you may already have done.
> - **`personal-profile`** — your persistent profile (name, role, values, goals, sensitivities). Edited only with your confirmation.
> - **`reflection-session`** — structured CBT-style reflection. Safety-gated.
> - **`business-mentoring`** — strategic-decision sparring with a decision journal.
> - **`fintech-legal-triage`** — issue-spotter for fintech regulation. Always asks jurisdiction first.
> - **`morning-briefing`** — five-field daily standup with yourself.
> - **`news-preferences`** — your news interests, sources, exclusions, cadence.
> - **`news-digest`** — daily personalized digest against the preferences. Refuses to fabricate.

## Step 3 — Subagents (one paragraph)

> Three model-pinned worker agents that get dispatched automatically when the corresponding skill needs them:
>
> - **`psychologist-listener`** (Sonnet) — reflective listening only, one question per turn, refuses diagnosis.
> - **`business-mentor`** (Sonnet) — strategic-decision worker; refuses to make the call for you.
> - **`fintech-legal-analyst`** (Opus) — careful regulatory issue-spotter; jurisdiction-first; take-to-counsel block on every output.

## Step 4 — Setup commands (one paragraph)

> Four slash commands. The first is the entry point you may already know; the other three wire the recurring rhythms into Cowork's Scheduled Tasks:
>
> - `/personal-coach:onboard` — start (or resume / re-run) the onboarding.
> - `/personal-coach:setup-morning-briefing` — daily briefing rhythm.
> - `/personal-coach:setup-news-digest` — daily news digest rhythm.
> - `/personal-coach:setup-decision-grading` — weekly grading scan for past-deadline decisions.

## Step 5 — Cowork Routines (one paragraph)

> Four optional Routine templates live at `docs/personal-coach-routines/` in the basic-harness repo (not inside the plugin). Routines run in Anthropic's cloud with the laptop closed — useful for the news digest and decision-grading scan. **Reflection and profile edits are deliberately not routinable** because reflection content is the most sensitive material in the plugin and shouldn't transit cloud. Read `docs/personal-coach-routines/README.md` before installing any of them — the privacy posture matters.

## Step 6 — Where files live

> Everything stays on your machine:
>
> - With a personal vault: `<vault>/Profile`, `<vault>/Journal`, `<vault>/Decisions`, `<vault>/Legal`, `<vault>/News`, `<vault>/Goals`, `<vault>/Business`, `<vault>/People`, `<vault>/Reading`.
> - Without a vault: `~/.claude/personal-profile.md`, `~/.claude/journal/`, `~/.claude/decisions/`, `~/.claude/legal-triage/`, `~/.claude/news/`.
>
> If you set up the vault later (`/vault-librarian:bootstrap-vault` with the personal schema), the files don't auto-migrate — you'd move them yourself. The plugin works the same either way.

## Step 7 — Hard limits (always — never skipped)

> A few things this plugin will not do, by design:
>
> - **No medical or psychological diagnosis.** The reflective listener mirrors and asks; never assesses.
> - **No legal advice.** The fintech triage produces issue lists for a real lawyer; every output ends with the take-to-counsel block.
> - **No silent profile edits.** Every write is shown and confirmed.
> - **No surrogate decisions.** The business-mentor sharpens your call; it doesn't make it.
>
> If you ever want to push past these, the plugin will tell you so explicitly and stop. That's a feature, not a flaw — it's how the plugin is safe to use as a long-running companion.

## Step 8 — Closing

> That's the tour. A few starting points, depending on what's most useful right now:
>
> - Just talk and the right skill will pick up — you don't need to memorize names.
> - To set up the daily rhythm: `/personal-coach:setup-morning-briefing`.
> - To set up the news digest: run the `news-preferences` skill first, then `/personal-coach:setup-news-digest`.
> - For the wider basic-harness map (other plugins beyond personal-coach), see `/welcome:tour`.
>
> Done. Take care.

Stop. Do not chain.

## What this command will NOT do

- **Will not write to disk.** Pure read-only orientation.
- **Will not invoke any of the named skills or commands.** The user runs them when they're ready.
- **Will not retry onboarding.** If the user wants to onboard, they type `/personal-coach:onboard` — we don't push.
- **Will not run more than ~2 minutes.** Use Steps 5 and 6 as expansion points if the user asks; otherwise keep it tight.

## Failure modes

- **User asks "tell me more about X"** for a specific skill — answer in 2-3 sentences and offer the SKILL.md path on disk for the curious. Do not paste the whole skill body.
- **User wants to start using something mid-tour** — break out of the tour, hand off to that thing. The tour is not so important that we'd block real work for it.
